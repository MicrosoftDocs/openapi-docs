---
title: References in OpenAPI documents
description: Learn the types of references that are supportined in the OpenAPI.NET library.
author: darrmi
ms.author: darrmi
ms.topic: conceptual
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
paths:
  /item:
    get:
      parameters:
        $ref: '#/components/parameters/size'
components:
  parameters:
    size:
     type: number
```

The following example shows a way to externalize security schemes using a reference object in a component.  This is necessary because security requirement objects do not allow referencing using an external URI.

```yaml
openapi: 3.1.0
info:
  title: Example of reference object in a component object
paths:
  /item:
    get:
        security:
            - customapikey: []
components:
  securitySchemes:
    customapikey:
        $ref: ./commonSecuritySchemes/customapikey.json#/components/securityschemes/customapikey
```


### Targeting Mechanisms used in Reference objects

Reference objects can use a relative reference with just a fragment identifier to point to an OAS Component.

```yaml
openapi: 3.1.0
info:
  title: Example of reference object pointing to a parameter
paths:
  /jobs/{id}:
    $ref: '#/components/pathItems/job'
components:
  pathItems:
    job:
      get:
      patch:
      delete:
```

Reference objects can reference OAS components in another OpenAPI document.

```yaml
openapi: 3.1.0
info:
  title: Example of reference object pointing to an example object in an OpenAPI document
paths:
  /items:
    get:
      responses:
        200:
          description: Ok
            application/json:
              examples:
                item-list:
                  $ref: './examples.yaml#/components/item-list'

```

```yaml
# file for examples (examples.yaml)
openapi: 3.1.0
info:
  title: OpenAPI document containing examples for reuse
components:
  examples:
    item-list:
      value:
        - name: thing
          description: a thing
```

For completeness, Reference Objects can also point to targets in a JSON/YAML document that contain properly formed OpenAPI objects, but not complete OpenApi document. We refer to these as OpenAPI "fragments". Currently OpenAPI.NET does not support referencing OpenAPI fragments.

```yaml
openapi: 3.1.0
info:
  title: Example of reference object pointing to a parameter
paths:
  /items:
    get:
      responses:
        200:
          description: Ok
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

Reference objects can be used to target the follow OAS types: parameter, response, example, header, securityScheme, link, callback and pathItem.  In OpenAPI 3.0, they also can reference schema objects.

## JSON Schema $refs in OpenAPI descriptions