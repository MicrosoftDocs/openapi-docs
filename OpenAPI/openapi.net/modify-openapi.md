---
title: Modify an OpenAPI document
description: Learn how to use the OpenAPI.NET library to modify an OpenAPI document.
author: jasonjoh
ms.author: jasonjoh
ms.topic: article
---

# Modify an OpenAPI document

The PetStore API described in [Create an OpenAPI document](create-openapi.md) has only one path for getting all pets. Imagine the API adds the capability to add a pet to the store.

This example adds a new path to the OpenAPI document to describe a new endpoint for adding a pet to the store.

```csharp
using Microsoft.OpenApi;
using Microsoft.OpenApi.Reader;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using System.Threading.Tasks;

// Configure settings so YAML parsing is also available
var readerSettings = new OpenApiReaderSettings();
readerSettings.AddYamlReader();

// Load the existing OpenAPI document from a YAML file
var (document, diagnostic) = await OpenApiDocument.LoadAsync("pet-store.yaml", readerSettings);

// Add a new property to the Pet schema
var categoryProperty = new OpenApiSchema
{
    Type = JsonSchemaType.Object,
    Properties = new Dictionary<string, IOpenApiSchema>
    {
        ["id"] = new OpenApiSchema
        {
            Type = JsonSchemaType.Integer,
        },
        ["name"] = new OpenApiSchema
        {
            Type = JsonSchemaType.String,
        },
    },
};

((OpenApiSchema)document.Components.Schemas["Pet"]).Properties.Add("category", categoryProperty);

// Add a new path
var newPetPath = new OpenApiPathItem
{
  Operations = new Dictionary<HttpMethod, OpenApiOperation>
  {
    [HttpMethod.Post] = new OpenApiOperation
    {
      Description = "Add a new pet",
      RequestBody = new OpenApiRequestBody
      {
        Content = new Dictionary<string, IOpenApiMediaType>
        {
          ["application/json"] = new OpenApiMediaType
          {
            Schema = new OpenApiSchemaReference("Pet"),
          },
        },
        Required = true, // Indicates that the body is required
      },
      Responses = new OpenApiResponses
      {
        ["201"] = new OpenApiResponse
        {
          Description = "Pet created successfully",
        },
      },
    },
    },
};

// Add the new path to the document
document.Paths.Add("/pets/post", newPetPath);

// Serialize and save the modified OpenAPI document to a new file
using var streamWriter = new StreamWriter("updated-pet-store.yaml");
var writer = new OpenApiYamlWriter(streamWriter);
document.SerializeAsV3(writer);
Console.WriteLine("PetStore OpenAPI document updated.");
```

The following YAML snippet is how the modified OpenAPI description for PetStore service looks now:

```yaml
openapi: 3.0.1
info:
  title: PetStore API
  version: 1.0.0
servers:
  - url: https://api.petstore.com
paths:
  '/pets':
    get:
      description: Get all pets
      responses:
        '200':
          description: A list of pets
          content:
            'application/json':
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Pet'
  '/pets/post':
    post:
      description: Add a new pet
      requestBody:
        content:
          'application/json':
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

## Next steps

- See [Converting an OpenAPI document](convert-openapi.md) to learn how to use the OpenAPI.NET library to convert between YAML and JSON formats.
