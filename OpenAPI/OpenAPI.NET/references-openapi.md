---
title: References in OpenAPI documents
description: Learn the types of references that are supported in the OpenAPI.NET library.
author: darrelmiller
ms.author: darrmi
ms.topic: article
ms.date: 02/17/2025
---

# References in OpenAPI documents

OpenAPI uses the keyword `$ref` to enable referencing and reusing existing objects defined in an OpenAPI description. However, there are many different ways to use this keyword and not all are supported by the OpenAPI.NET library.  Also, OpenAPI 3.1 formalized the distinction between the use of the `$ref` keyword in a Schema object vs elsewhere in an OpenAPI description. To best understand how to use `$ref` it is helpful to be aware of its capabilities and the current limitations of OpenAPI.Net.

## Reference Objects

Prior to OpenAPI v3.1, Reference Objects was the term for all usages of `$ref` within OpenAPI descriptions. However, when OpenAPI v3.1 introduced support for JSON Schema beyond the draft-4 version, it became necessary to allow the use of `$ref` within Schema Objects to follow the rules of JSON Schema and all other uses in OpenAPI descriptions would be considered Reference Objects. Therefore, this section only describes the rules for `$ref` keywords that are not in a Schema Object.

### Where Reference Objects can be used

Reference objects can appear in one of two places.  Either in place of an inline object, or a map value in the components section.

This example below shows a common usage pattern of referencing a shared parameter object using a reference object.

```yaml
openapi: 3.1.0
info:
  title: Example of reference object pointing to a parameter
  version: 1.0.0
paths:
  /item:
    get:
      parameters:
        $ref: '#/components/parameters/size'
components:
  parameters:
    size:
      name: size
      in: query
      schema:
        type: number
```

The following example shows a way to externalize security schemes using a reference object in a component.  This is necessary because security requirement objects do not allow referencing using an external URI.

```yaml
openapi: 3.1.0
info:
  title: Example of reference object in a component object
  version: 1.0.0
paths:
  /item:
    get:
      security:
        - customapikey: []
components:
  securitySchemes:
    customapikey:
      $ref: ./commonSecuritySchemes/customapikey.json#/components/securitySchemes/customapikey
```

### Targeting Mechanisms used in Reference objects

Reference objects can use a relative reference with just a fragment identifier to point to an OAS Component.

```yaml
openapi: 3.1.0
info:
  title: Example of reference object pointing to a pathitem
  version: 1.0.0
paths:
  /jobs/{id}:
    post:
      requestBody:
        $ref: '#/components/requestBodies/job'
components:
  requestBodies:
    job:
      required: true
      content:
        application/json: {}
```

Reference objects can reference OAS components in another OpenAPI document.

```yaml
openapi: 3.1.0
info:
  title: Example of reference object pointing to an example object in an OpenAPI document
  version: 1.0.0
paths:
  /items:
    get:
      responses:
        200:
          description: Ok
          content:
            application/json:
              examples:
                item-list:
                  $ref: './examples.yaml#/components/examples/item-list'
```

```yaml
# file for examples (examples.yaml)
openapi: 3.1.0
info:
  title: OpenAPI document containing examples for reuse
  version: 1.0.0
components:
  examples:
    item-list:
      value:
        - name: thing
          description: a thing
```

For completeness, Reference Objects can also point to targets in a JSON/YAML document that contain properly formed OpenAPI objects, but is not a complete OpenApi document. We refer to these as OpenAPI "fragments". Currently OpenAPI.NET does not support referencing OpenAPI fragments.

```yaml
openapi: 3.1.0
info:
  title: Example of reference object pointing to a parameter
  version: 1.0.0
paths:
  /items:
    get:
      responses:
        200:
          description: Ok
          content:
            application/json:
              examples:
                item-list:
                  $ref: './examples.yaml#/item-list'

```

```yaml
# file for example fragments (examples.yaml)
item-list:
  value:
    - name: thing
      description: a thing
```

### What types Reference objects can target

Reference objects can be used to target the follow OAS types: parameter, response, requestBody, example, header, securityScheme, link, callback and pathItem.  In OpenAPI 3.0, they also can reference schema objects.

## JSON Schema $refs in OpenAPI descriptions

From OpenAPI 3.1 onwards, a `$ref` inside a Schema object is not considered an OpenAPI Reference Object. It behaves like a JSON Schema reference, specifically a JSON Schema 2020-12 reference, unless the dialect is changed for the OpenAPI document.

### Where can JSON Schema references be used

JSON Schema references can be used in the following locations:

- at the root of an inline schema
- in a subschema of an inline schema
- at the root of a component schema
- in a subschema of a component schema

#### Root of an inline schema

```yaml
openapi: 3.1.0
info:
  title: Reference in at the root of an inline schema
  version: 1.0.0
paths:
  /item:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/item'
components:
  schemas:
    item:
      type: object
```

#### In a subschema of an inline schema

```yaml
openapi: 3.1.0
info:
  title: Reference in at the root of an inline schema
  version: 1.0.0
paths:
  /items:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/item'
components:
  schemas:
    item:
      type: object
```

#### At the root of a component schema

```yaml
openapi: 3.1.0
info:
  title: Reference at the root of a component schema
  version: 1.0.0
paths:
  /items:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/specialitem'
components:
  schemas:
    specialitem:  # Use the item type but provide a different title for the type
      title: Special Item
      $ref: "#/components/schemas/item"
    item:
      title: Item
      type: object
```

#### In a subschema of a component schema

```yaml
openapi: 3.1.0
info:
  title: Reference in a subschema of an component schema
  version: 1.0.0
paths:
  /items:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/items'
components:
  schemas:
    items:
      type: array
      items:
        $ref: '#/components/schemas/item'
    item:
      type: object
```

#### References local to a JSON Schema Resource defined by an OpenAPI Schema

JSON Schema defines the concept of a JSON Schema Resource which is identified by a URI.  $ref values can be specified relative to the JSON Schema Resource.  Each OpenAPI schema object can be considered a JSON Schema Resource.

In this example schema "a" is a JSON Schema resource and the reference in the "b" property of the "c" object is relative to the JSON Schema resource "a".  Unfortunately, OpenAPI 3.1 has no well defined URIs for OpenAPI Schemas. This is resolved in OpenAPI 3.2 with the introduction of a new top level "$self" property.

```yaml
components:
  schemas:
  # this component schema is ALSO a JSON schema resource
  # therefore, the ref is relative to this schema.
  # this would also work if the whole tree is a schema inside a media type schema, etc...
    a:
      type:
        - object
        - 'null'
      properties:
        b:
          type:
            - object
            - 'null'
          properties:
            c:
              type:
                - object
                - 'null'
              properties:
                b:
                  $ref: '#/properties/b'
```

#### References local to a JSON Schema Resource defined by a $id [Not currently supported]

```yaml
components:
  schemas:
  a:
    type:
      - object
      - 'null'
    additionalProperties: false
    properties:
      b:
        type:
          - object
          - 'null'
        additionalProperties: false
        properties:
          c:
          # this is similar to the prior example, but the $id here turns this specific schema in a resource
          # it effectively defines a "new root" for relative lookups.
            $id: 'http://example.org/c'
            type:
              - object
              - 'null'
            additionalProperties: false
            properties:
              b:
                $ref: '#/properties/d'
              d:
                type: string
```

### What kinds of JSON Schema references exist

JSON Schema references can use either a locator or an identifier. Locators indicate where to find the target schema based on the name and structure of documents.  However, identifiers are opaque URIs that match to the identifier of a JSON schema that has a $id either explicitly or implicitly by inheriting an identity from its parent.

### What can a JSON Schema locator reference target

- An internal component
- An internal component subschema
- An internal inline subschema
- An external OpenApi document component
- An external inline subchema
- An external inline subchema using anchor
- An external fragment [Not supported]

#### An internal component

```yaml
openapi: 3.1.0
info:
  title: Reference to an internal component
  version: 1.0.0
paths:
  /item:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/item'
components:
  schemas:
    item:
      type: object
```

#### An internal component subschema

```yaml
openapi: 3.1.0
info:
  title: Reference to an internal component
  version: 1.0.0
paths:
  /person/{id}:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/person'
  /person/{id}/address:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/person/properties/address'
components:
  schemas:
    person:
      type: object
      properties:
        name:
          type: string
        address:
          type: object
          properties:
            street:
              type: string
            city:
              type: string
```

#### An external OpenApi document component

```yaml
openapi: 3.1.0
info:
  title: Reference to an external OpenApi document component
  version: 1.0.0
paths:
  /person/{id}:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                # the URI is assumed to be relative according to RFC 3986
                $ref: 'OAS-schemas.yaml#/components/schemas/person'
```

```yaml
# OAS-schemas.yaml file
openapi: 3.1.0
info:
  title: OpenAPI document containing reusable components
  version: 1.0.0
components:
  schemas:
    person:
      type: object
      properties:
        name:
          type: string
        address:
          type: object
          properties:
            street:
              type: string
            city:
              type: string
```

#### An external inline subchema

```yaml
openapi: 3.1.0
info:
  title: Reference to an external OpenApi document component
  version: 1.0.0
paths:
  /person/{id}:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                $ref: 'OAS-schemas.yaml#/components/schemas/person/properties/address'
```

```yaml
# OAS-schemas.yaml file
openapi: 3.1.0
info:
  title: OpenAPI document containing reusable components
  version: 1.0.0
components:
  schemas:
    person:
      type: object
      properties:
        name:
          type: string
        address:
          type: object
          properties:
            street:
              type: string
            city:
              type: string
```

#### An external inline subchema using an anchor

OpenAPI.NET supports resolving plain-name JSON Schema anchors in external OpenAPI documents.

> Note: support was added in versions 3.9.0 and 2.11.0.

We accept pull requests.

```yaml
openapi: 3.1.0
info:
  title: Reference to an external OpenApi document component
  version: 1.0.0
paths:
  /person/{id}:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                $ref: 'OAS-schemas.yaml#address'
```

```yaml
# OAS-schemas.yaml file
openapi: 3.1.0
info:
  title: OpenAPI document containing reusable components
  version: 1.0.0
components:
  schemas:
    person:
      type: object
      properties:
        name:
          type: string
        address:
          $anchor: address
          type: object
          properties:
            street:
              type: string
            city:
              type: string

```

#### An external fragment [Not supported]

```yaml
openapi: 3.1.0
info:
  title: Reference to an external OpenApi document component
  version: 1.0.0
paths:
  /person/{id}:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                $ref: 'OAS-schemas.yaml#/person/properties/address'
```

```yaml
# OAS-schemas.yaml file
person:
  type: object
  properties:
    name:
      type: string
    address:
      type: object
      properties:
        street:
          type: string
        city:
          type: string
```

### What can a JSON Schema identifier reference target

- Reference an internal component using a $id
- Reference an internal subschema using a $id
- Reference an external component schema using a $id

#### Reference an internal component using a $id

```yaml
openapi: 3.1.0
info:
  title: Reference an internal component using id
  version: 1.0.0
paths:
  /person/{id}:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                $ref: 'https://schemas.acme.org/person'
components:
  schemas:
    person:
      $id: 'https://schemas.acme.org/person'
      type: object
      properties:
        name:
          type: string
        address:
          type: object
          properties:
            street:
              type: string
            city:
              type: string
```

#### Reference an internal subschema using a $id

OpenAPI.NET supports resolving `$id` values on nested schemas. Relative `$id` values are resolved against the closest parent schema resource identifier.

> Note: support was added in versions 3.9.0 and 2.11.0.

```yaml
openapi: 3.1.0
info:
  title: Reference an internal subschema using id
  version: 1.0.0

paths:
  /person/{id}/address:
    get:
      responses:
        200:
          description: ok
          content:
            application/json:
              schema:
                $ref: 'https://schemas.acme.org/address'
components:
  schemas:
    person:
      $id: 'https://schemas.acme.org/person'
      type: object
      properties:
        name:
          type: string
        address:
          # this is equivalent to https://schemas.acme.org/address because
          # https://schemas.acme.org/person does NOT end up with / (RFC 3986)
          $id: 'address'
          type: object
          properties:
            street:
              type: string
            city:
              type: string
```

#### Reference external component schemas using a $id

External component schemas are referenced in the exact same way as internal component schemas. External subschemas that rely on their own `$id` are not currently supported.

## $dynamicAnchor and $dynamicRef

JSON Schema 2020-12, which is the basis for OpenAPI 3.1 schemas, introduces `$dynamicAnchor` and `$dynamicRef` as a mechanism for building extensible, generic schemas. They work together as overridable extension hooks:

- `$dynamicAnchor` declares a named placeholder anchor in a schema. Any parent schema that references it can redefine the anchor to specialize behavior.
- `$dynamicRef` references a dynamic anchor. Unlike `$ref`, it does not resolve statically. Instead, evaluation looks back through the stack of schema resources traversed so far, and jumps to the **first** encountered definition of that anchor. This allows a specializing schema to override the referenced subschema.

This is analogous to generic type parameters in programming languages such as C++ templates or Java generics.

OpenAPI.NET exposes `$dynamicAnchor` and `$dynamicRef` as properties on the schema model, but dynamic reference resolution is not currently implemented.

The following example defines a generic `collection` schema whose item type can be overridden by a more specialized schema.

> Note: support was added in versions 3.9.0 and 2.11.0.

```yaml
openapi: 3.1.0
info:
  title: Generic collection schema using $dynamicAnchor and $dynamicRef
  version: 1.0.0
components:
  schemas:
    # A generic, reusable collection schema.
    # The item type is left open and can be overridden by a specializing
    # schema that redefines the "collection-item" dynamic anchor.
    collection:
      $id: 'https://schemas.acme.org/collection'
      type: array
      items:
        $dynamicRef: '#collection-item'
      $defs:
        default:
          $comment: Default declaration to satisfy the bookending requirement
          $dynamicAnchor: collection-item

    # A specialized collection schema that constrains items to strings.
    # It references the generic schema and overrides the dynamic anchor.
    string-collection:
      $ref: 'https://schemas.acme.org/collection'
      $defs:
        collection-item:
          $dynamicAnchor: collection-item
          type: string
```

In this example:

- `collection` uses `$dynamicRef: '#collection-item'` to declare that each array item should validate against whatever `collection-item` resolves to at runtime.
- The `$defs/default` subschema in `collection` declares the `$dynamicAnchor: collection-item` placeholder. This satisfies the *bookending requirement*: the base schema that uses `$dynamicRef` must itself provide a `$dynamicAnchor` with the same name, even if it imposes no constraints.
- `string-collection` uses `$ref` to extend `collection` and redefines `collection-item` in its own `$defs` with `type: string`.
- When `string-collection` is evaluated, the `$dynamicRef` in `collection` resolves to the `collection-item` anchor defined in `string-collection`, effectively constraining all array items to be strings.

## $ref in PathItem Objects

This scenario is supported in OpenAPI.NET.

```yaml
openapi: 3.1.0
info:
  title: Example of reference object pointing to a pathitem
  version: 1.0.0
paths:
  /jobs/{id}:
    $ref: '#/components/pathItems/job'
components:
  pathItems:
    job:
      get: {}
      patch: {}
      delete: {}
```
