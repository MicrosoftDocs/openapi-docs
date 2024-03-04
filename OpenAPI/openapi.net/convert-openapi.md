---
title: Convert an OpenAPI document
description: Learn how to use the OpenAPI.NET library to convert OpenAPI documents between YAML and JSON
author: njaci1
ms.author: kelvinnjaci
ms.topic: conceptual
---

# Convert an OpenAPI document

OpenAPI.NET has the ability to convert OpenAPI documents between different formats, such as YAML or JSON. This can be useful when you need to share or collaborate on an OpenAPI document with others who may prefer a different format.

Here is an example of how you can convert an OpenAPI document from YAML to JSON using OpenAPI.NET.

This example converts the OpenAPI document [created](create-openapi.md) and [modified](modify-openapi.md) in the previous examples from YAML to JSON.

```csharp
using Microsoft.OpenApi.Readers;
using Microsoft.OpenApi.Writers;

// Load the existing OpenAPI document from a YAML file
using var streamReader = new StreamReader("updated-pet-store.yaml");
var reader = new OpenApiStreamReader();
var document = reader.Read(streamReader.BaseStream, out var diagnostic);

// Serialize and save the OpenAPI document to a JSON file
using var streamWriter = new StreamWriter("updated-pet-store.json");
var writer = new OpenApiJsonWriter(streamWriter);
document.SerializeAsV3(writer);

Console.WriteLine("OpenAPI document converted from YAML to JSON.");
```

Compare the original YAML with the new JSON.

## YAML

```yaml
openapi: 3.0.1
info:
  title: PetStore API
  version: 1.0.0
servers:
  - url: https://api.petstore.com
paths:
  /pets:
    get:
      description: Get all pets
      responses:
        '200':
          description: A list of pets
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Pet'
  /pets/post:
    post:
      description: Add a new pet
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Pet'
        required: true
      responses:
        '201':
          description: Pet created successfully
components:
  schemas:
    Pet:
      type: object
      properties:
        name:
          type: string
        category:
          type: object
          properties:
            id:
              type: integer
            name:
              type: string
```

## JSON

```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "PetStore API",
    "version": "1.0.0"
  },
  "servers": [
    {
      "url": "https://api.petstore.com"
    }
  ],
  "paths": {
    "/pets": {
      "get": {
        "description": "Get all pets",
        "responses": {
          "200": {
            "description": "A list of pets",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "$ref": "#/components/schemas/Pet"
                  }
                }
              }
            }
          }
        }
      }
    },
    "/pets/post": {
      "post": {
        "description": "Add a new pet",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "$ref": "#/components/schemas/Pet"
              }
            }
          },
          "required": true
        },
        "responses": {
          "201": {
            "description": "Pet created successfully"
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "Pet": {
        "type": "object",
        "properties": {
          "name": {
            "type": "string"
          },
          "category": {
            "type": "object",
            "properties": {
              "id": {
                "type": "integer"
              },
              "name": {
                "type": "string"
              }
            }
          }
        }
      }
    }
  }
}
```
