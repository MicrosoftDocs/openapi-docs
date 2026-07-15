---
title: Earlier-version serialization considerations
description: Learn what to consider when serializing OpenAPI documents to earlier OpenAPI versions with OpenAPI.NET.
author: baywet
ms.author: vibiret
ms.topic: article
ms.date: 07/14/2026
---

# Earlier-version serialization considerations

OpenAPI.NET can deserialize newer OpenAPI documents into an object model and serialize that object model as an earlier OpenAPI version. This conversion is useful for interoperability, but OpenAPI 2.0 and OpenAPI 3.0 don't support every feature in later OpenAPI versions or in the OpenAPI.NET object model.

The output format, JSON or YAML, doesn't change which information is preserved. The target `OpenApiSpecVersion` determines the conversion behavior.

Information can be handled in four ways when you serialize to an older version:

| Result | Meaning |
| --- | --- |
| Preserved natively | The target OpenAPI version has an equivalent field. |
| Preserved as an extension | OpenAPI.NET writes the information with an `x-` extension because the target version has no native field. Other tools might ignore it. |
| Approximated | OpenAPI.NET writes the closest older-version representation, but the exact original semantics aren't recoverable. |
| Dropped or blocked | OpenAPI.NET omits the information, writes an empty object, or throws when the target version can't represent the value safely. |

## Default value considerations before serialization

Some object model properties use non-nullable `bool` values. After a document is loaded, OpenAPI.NET can't always distinguish between a field that was explicitly set to its default value and a field that was omitted. When the writer omits default values, the explicit presence of those fields can be lost even if you serialize back to the same OpenAPI version.

This affects fields such as:

| Object | Boolean fields where explicit default presence can be lost |
| --- | --- |
| `OpenApiParameter` | `allowEmptyValue`, `allowReserved`, `deprecated`, `explode`, `required` |
| `OpenApiHeader` | `allowEmptyValue`, `allowReserved`, `deprecated`, `explode`, `required` |
| `OpenApiOperation` | `deprecated` |
| `OpenApiRequestBody` | `required` |
| `OpenApiSchema` | `additionalProperties` when represented by `AdditionalPropertiesAllowed`, `deprecated`, `readOnly`, `unevaluatedProperties`, `writeOnly` |
| `OpenApiSecurityScheme` | `deprecated` |

For example, an explicitly written `deprecated: false` can become indistinguishable from an omitted `deprecated` field.

## Serializing to OpenAPI 3.0

OpenAPI 3.0 is close to OpenAPI 3.1 and 3.2, but it doesn't support JSON Schema 2020-12, webhooks, reusable path items, media type components, the `query` HTTP method, or the `querystring` parameter location.

### Document-level information

| Source information | OpenAPI.NET behavior when writing 3.0 | Result |
| --- | --- | --- |
| `jsonSchemaDialect` | Omitted. | Dropped |
| `webhooks` | Omitted. | Dropped |
| `$self` | Written as `x-oai-$self`. | Preserved as an extension |
| `components.pathItems` | Omitted. | Dropped |
| `components.mediaTypes` | Omitted. | Dropped |

### Paths, operations, and parameters

| Source information | OpenAPI.NET behavior when writing 3.0 | Result |
| --- | --- | --- |
| Path item `query` operation or any other method that isn't native to OpenAPI 3.0 | Written under `x-oai-additionalOperations`. | Preserved as an extension |
| Parameter location `querystring` | Throws because `querystring` is only supported in OpenAPI 3.2 and later. | Blocked |
| Parameter style `cookie` | Throws because this style is only supported in OpenAPI 3.2 and later. | Blocked |

### Components, responses, tags, examples, media types, and encodings

| Source information | OpenAPI.NET behavior when writing 3.0 | Result |
| --- | --- | --- |
| Response `summary` | Written as `x-oai-summary`. | Preserved as an extension |
| Tag `summary`, `parent`, and `kind` | Written as `x-oas-summary`, `x-oas-parent`, and `x-oas-kind`. | Preserved as extensions |
| Example `dataValue` and `serializedValue` | Written as `x-oai-dataValue` and `x-oai-serializedValue`. | Preserved as extensions |
| Media type `itemSchema`, `itemEncoding`, and `prefixEncoding` | Written as `x-oai-itemSchema`, `x-oai-itemEncoding`, and `x-oai-prefixEncoding`. | Preserved as extensions |
| Encoding `encoding`, `itemEncoding`, and `prefixEncoding` | Written as `x-oai-encoding`, `x-oai-itemEncoding`, and `x-oai-prefixEncoding`. | Preserved as extensions |

### Security schemes

| Source information | OpenAPI.NET behavior when writing 3.0 | Result |
| --- | --- | --- |
| `mutualTLS` security scheme | Throws because `mutualTLS` is only supported in OpenAPI 3.1 and later. | Blocked |
| OAuth 2 `oauth2MetadataUrl` | Written as `x-oauth2-metadata-url`. | Preserved as an extension |
| Security scheme `deprecated` | Written as `x-oai-deprecated`. | Preserved as an extension |

### Schema information

OpenAPI 3.0 uses an extended subset of JSON Schema rather than JSON Schema 2020-12. Schema conversion is the most common source of version-specific behavior.

| Source information | OpenAPI.NET behavior when writing 3.0 | Result |
| --- | --- | --- |
| `$schema`, `$id`, `$comment`, `$vocabulary`, `$defs`, `$dynamicRef`, `$dynamicAnchor` | Omitted. | Dropped |
| `const` | Converted to a one-value `enum` only when no `enum` is already present. If `enum` is present, `const` is omitted. | Approximated or dropped |
| Multiple schema types, such as `type: ["string", "integer"]` | The `type` field is omitted unless the only extra type is `null`. | Dropped |
| Nullable types, such as `type: ["string", "null"]` | Written as `type: string` plus `nullable: true`. | Approximated |
| A schema that only allows `null` | Written as `enum: [null]`. | Approximated |
| Numeric `exclusiveMaximum` or `exclusiveMinimum` | Converted to `maximum` or `minimum` plus a boolean exclusive flag. If both inclusive and exclusive bounds are modeled, the older form can't preserve both independently. | Approximated |
| `examples` on a schema | Omitted. The singular `example` field is preserved. | Dropped |
| `dependentRequired` | Omitted. | Dropped |
| `unevaluatedProperties`, `patternProperties`, `$anchor`, `contentEncoding`, `contentMediaType`, `contentSchema`, `contains`, `maxContains`, `minContains`, `propertyNames`, `dependentSchemas`, `if`, `then`, and `else` | Written with `x-jsonschema-*` extension names. | Preserved as extensions |

## Serializing to OpenAPI 2.0

OpenAPI 2.0 has a substantially different document shape. It has one global `host`, one `basePath`, a global `consumes` and `produces` model, `definitions` instead of `components.schemas`, body and form parameters instead of `requestBody`, and no native support for callbacks, links, content maps, OpenID Connect, mutual TLS, or most JSON Schema 2020-12 keywords.

### Document-level information

| Source information | OpenAPI.NET behavior when writing 2.0 | Result |
| --- | --- | --- |
| `servers` | The first server is used to write `host` and `basePath`. Schemes are collected only from servers with the same host, port, and path as the first server. Server variables are substituted before writing. Other server URLs and server metadata are not preserved. | Approximated |
| `jsonSchemaDialect`, `$self`, and `webhooks` | Omitted. | Dropped |
| `components.schemas` | Written as `definitions`. | Preserved natively |
| `components.parameters` | Written as top-level `parameters`. | Preserved natively |
| `components.requestBodies` | Converted to top-level body parameters when a parameter with the same key doesn't already exist. | Approximated |
| `components.responses` | Written as top-level `responses`. | Preserved natively |
| `components.securitySchemes` | Written as `securityDefinitions`, with the security scheme limitations described later. | Approximated |
| `components.examples`, `components.headers`, `components.links`, `components.callbacks`, `components.pathItems`, and `components.mediaTypes` | Omitted. | Dropped |

### Paths and operations

| Source information | OpenAPI.NET behavior when writing 2.0 | Result |
| --- | --- | --- |
| Path item `summary` and `description` | Written as `x-summary` and `x-description`. | Preserved as extensions |
| Path item `servers` | Omitted. | Dropped |
| `trace`, `query`, and other non-OpenAPI 2.0 operations | Written under `x-oai-additionalOperations`. | Preserved as extensions |
| Operation `requestBody` | Converted to a body parameter or to multiple `formData` parameters for `application/x-www-form-urlencoded` and `multipart/form-data`. | Approximated |
| Operation request body content media types | Written as operation-level `consumes`. The parameter schema comes from the first applicable media type. | Approximated |
| Operation response content media types | Written as operation-level `produces`. Response schemas and examples are derived from response content as described later. | Approximated |
| Operation callbacks | Omitted. | Dropped |
| Operation servers | Only the URL schemes are written at the operation level. Host, path, variables, and other server information are omitted. | Approximated |

### Request bodies, responses, parameters, headers, media types, and links

| Source information | OpenAPI.NET behavior when writing 2.0 | Result |
| --- | --- | --- |
| Request body object | Converted to a body parameter, or to form data parameters for form and multipart content. The request body object itself doesn't exist in OpenAPI 2.0. | Approximated |
| Request body description and required flag | Moved to the generated body or form parameters. | Approximated |
| Request body content map | The first content entry is used for schema and examples; media type names are represented separately in `consumes`. | Approximated |
| Request body media type `example`, `encoding`, `itemSchema`, `itemEncoding`, `prefixEncoding`, and media type extensions | Omitted during request body conversion. | Dropped |
| Response content map | The first content entry is used for the response `schema`. Media type examples are written under `examples` or `x-examples`. Media type extensions from the selected content entry are copied to the response. | Approximated |
| Response `summary` | Omitted. | Dropped |
| Response links | Omitted. | Dropped |
| Header `content` and `examples` | Omitted. Header schema is flattened to OpenAPI 2.0 header fields. | Dropped or approximated |
| Parameter `content` | Omitted. Parameters use the schema-based OpenAPI 2.0 representation. | Dropped |
| Parameter `examples` | Written as `x-examples`. | Preserved as an extension |
| Query array parameter serialization | Only some `style` and `explode` combinations are mapped to `collectionFormat` (`multi`, `pipes`, or `ssv`). Other combinations are not recoverable. | Approximated |
| Parameter location `querystring` | Throws because it isn't supported in OpenAPI 2.0. | Blocked |
| Parameter style `cookie` | Throws because this style is only supported in OpenAPI 3.2 and later. | Blocked |
| Link objects | Omitted. | Dropped |
| Media type and encoding objects serialized directly | Omitted because OpenAPI 2.0 has no equivalent objects. | Dropped |

### Security schemes

| Source information | OpenAPI.NET behavior when writing 2.0 | Result |
| --- | --- | --- |
| HTTP Basic security scheme | Written as OpenAPI 2.0 `type: basic`. | Preserved natively |
| HTTP token or other non-basic HTTP scheme | Written as an empty security scheme object. | Dropped |
| OpenID Connect security scheme | Written as an empty security scheme object. | Dropped |
| Mutual TLS security scheme | Written as an empty security scheme object. | Dropped |
| OAuth 2 flows | Only one flow is selected, in this order: implicit, password, client credentials, authorization code. Other flows are omitted. | Approximated |
| OAuth 2 flow `refreshUrl` and extensions | Omitted because OpenAPI 2.0 OAuth flow fields only include `flow`, `authorizationUrl`, `tokenUrl`, and `scopes`. | Dropped |
| Security scheme `deprecated`, `bearerFormat`, `openIdConnectUrl`, and `oauth2MetadataUrl` | Omitted unless carried separately as custom extensions. | Dropped |

### Schema information

| Source information | OpenAPI.NET behavior when writing 2.0 | Result |
| --- | --- | --- |
| `anyOf` and `oneOf` | If `allOf` isn't already present, only the first `anyOf` schema is written as `allOf`; if `anyOf` isn't present, only the first `oneOf` schema is written as `allOf`. Remaining alternatives are omitted. | Approximated |
| `not` | Omitted. | Dropped |
| Multiple schema types, such as `type: ["string", "integer"]` | The `type` field is omitted unless the only extra type is `null`. | Dropped |
| Nullable types | Written as `x-nullable: true`. | Preserved as an extension |
| `writeOnly` and schema `deprecated` | Omitted. | Dropped |
| `readOnly` on a property that is also required by the parent schema | Omitted because OpenAPI 2.0 doesn't allow required read-only properties. | Dropped |
| Discriminator mapping | Only the discriminator property name is written. Mappings are omitted. | Approximated |
| `const` | Converted to a one-value `enum` only when no `enum` is already present. If `enum` is present, `const` is omitted. | Approximated or dropped |
| JSON Schema 2020-12 keywords such as `$schema`, `$id`, `$comment`, `$vocabulary`, `$defs`, `$dynamicRef`, `$dynamicAnchor`, `dependentRequired`, `contentEncoding`, `contentMediaType`, `contentSchema`, `contains`, `maxContains`, `minContains`, `propertyNames`, `dependentSchemas`, `if`, `then`, and `else` | Omitted, except for the compatibility extensions listed below. | Dropped |
| `unevaluatedProperties` and `patternProperties` | Written as `x-jsonschema-unevaluatedProperties` and `x-jsonschema-patternProperties`. | Preserved as extensions |
| Schema `examples` | Omitted. The singular `example` field is preserved. | Dropped |
| Array item serialization metadata from parameter `style` and `explode` | Not available from reusable schemas, so `collectionFormat` might not be emitted for schema definitions. | Dropped |

## Reducing conversion gaps

Use the newest OpenAPI version that all of your downstream tools support. If you must serialize to OpenAPI 3.0 or 2.0, review the output before publishing it and avoid depending on extension-preserved data unless your consumers explicitly understand those extensions.

For workflows that need round-tripping, keep the original source document as the canonical artifact. Treat older-version output as a compatibility projection, not as a complete copy of the source document.
