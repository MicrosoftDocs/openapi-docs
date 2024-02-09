---
title: Create an OpenAPI document
description: Learn how to use the OpenAPI.NET library to create an OpenAPI document
author: njaci1
ms.author: kelvinnjaci
ms.topic: conceptual
---

# Create an OpenAPI document

The `OpenApiDocument` class contains methods and properties that developers use to create an OpenAPI document in code.

This example defines a simple PetStore API definition that allows users to view pets in an online store. It adds a GET operation for a single path (`/pets`), and defines the `Pet` type returned by the API.

```csharp
using Microsoft.OpenApi.Models;
using Microsoft.OpenApi.Writers;

// Create an OpenAPI document for the PetStore API
var document = new OpenApiDocument
{
    Info = new OpenApiInfo
    {
        Title = "PetStore API",
        Version = "1.0.0",
    },
    Servers =
    [
        new OpenApiServer { Url = "https://api.petstore.com" },
    ],
    Paths = new OpenApiPaths
    {
        ["/pets"] = new OpenApiPathItem
        {
            Operations = new Dictionary<OperationType, OpenApiOperation>
            {
                [OperationType.Get] = new OpenApiOperation
                {
                    Description = "Get all pets",
                    Responses = new OpenApiResponses
                    {
                        ["200"] = new OpenApiResponse
                        {
                            Description = "A list of pets",
                            Content = new Dictionary<string, OpenApiMediaType>
                            {
                                ["application/json"] = new OpenApiMediaType
                                {
                                    Schema = new OpenApiSchema
                                    {
                                        Type = "array",
                                        Items = new OpenApiSchema
                                        {
                                            Reference = new OpenApiReference
                                            {
                                                Type = ReferenceType.Schema,
                                                Id = "Pet",
                                            },
                                        },
                                    },
                                },
                            },
                        },
                    },
                },
            },
        },
    },
    Components = new OpenApiComponents
    {
        Schemas = new Dictionary<string, OpenApiSchema>
        {
            ["Pet"] = new OpenApiSchema
            {
                Type = "object",
                Properties = new Dictionary<string, OpenApiSchema>
                {
                    ["name"] = new OpenApiSchema
                    {
                        Type = "string",
                    },
                },
            },
        },
    },
};

// Serialize the OpenAPI document to a YAML file
using (var streamWriter = new StreamWriter("pet-store.yaml"))
{
    var writer = new OpenApiYamlWriter(streamWriter);
    document.SerializeAsV3(writer);
}
Console.WriteLine("PetStore OpenAPI document created and saved.");
```

Here is the resulting OpenAPI description for our PetStore service:

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
components:
  schemas:
    Pet:
      type: object
      properties:
        name:
          type: string
```

## Next steps

- See [Modifying an OpenAPI document](modify-openapi.md) to learn how to add a new path to this OpenAPI document.
- See [Converting an OpenAPI document](convert-openapi.md) to learn how to use the OpenAPI.NET library to convert between YAML and JSON formats.
