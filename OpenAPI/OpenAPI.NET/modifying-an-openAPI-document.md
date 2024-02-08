---
title: Modifying an OpenApi Document
description: Learn how to use OpenAPI.NET library to modify an OpenAPI documents
author: njaci1
ms.author: kelvinnjaci
ms.topic: conceptual
date: 01/02/2024
---
# Modify an OpenApi Document
The PetStore service we described in the [Create an OpenApi Document](creating-an-openAPI.md) has only one path for getting all pets. Let’s assume now that we have added capability to allow users to add a pet to the store. To modify the OpenAPI document for the PetStore service, we can use the object model to make changes to its components. In this example, we are going to add a new path to the OpenAPI document to describe a new endpoint for adding a pet to the PetStore.

```csharp
using System; 

using Microsoft.OpenApi.Models; 

using Microsoft.OpenApi.Readers; 

using System.IO; 

using Microsoft.OpenApi.Writers;

class ModifyOpenApiDocument 

{ 

    static void Main() 

    { 

        // Load the existing OpenAPI document from a YAML file 

        var filePath = "petstore.yaml"; 

        if (File.Exists(filePath)) 

        { 
            var document = ReadOpenApiDocumentFromYamlFile(filePath); 

            // Now you can modify the document as needed 

            // For example, let's add a new path 

            var newPetPath = new OpenApiPathItem 

            { 

                Operations = new Dictionary<OperationType, OpenApiOperation> 

                { 

                    [OperationType.Post] = new OpenApiOperation 

                    { 

                        Description = "Add a new pet", 

                        RequestBody = new OpenApiRequestBody 

                        { 

                            Content = new Dictionary<string, OpenApiMediaType> 

                            { 

                                ["application/json"] = new OpenApiMediaType 

                                { 

                                    Schema = new OpenApiSchema 

                                    {

                                        Type = "object", 

                                        Properties = new Dictionary<string, OpenApiSchema> 

                                        { 

                                            ["name"] = new OpenApiSchema { Type = "string" }, 

                                            ["category"] = new OpenApiSchema 

                                            { 

                                                Reference = new OpenApiReference { Type = ReferenceType.Schema, Id = "Category" } 

                                            } 

                                            // Add other properties as needed 

                                        }, 

                                    } 

                                } 

                            }, 

                             Required = true, // Indicates that the "name" property is required 
                        }, 

                        Responses = new OpenApiResponses 

                        { 

                            ["201"] = new OpenApiResponse 

                            { 

                                Description = "Pet created successfully" 

                            } 

                        } 

                    } 

                } 

            }; 

            // Add the new path to the document 

            document.Paths.Add("/pets/post", newPetPath); 

            // Serialize and save the modified OpenAPI document to a new file 

            SerializeOpenAPIDocument(document, "updated_petstore.yaml"); 

            Console.WriteLine("PetStore OpenAPI document updated."); 

        } 

        else 

        { 

            Console.WriteLine($"File '{filePath}' not found."); 

        } 

    } 

    static OpenApiDocument ReadOpenApiDocumentFromYamlFile(string filePath) 

    { 

        using (var streamReader = new StreamReader(filePath)) 

        { 

            var reader = new OpenApiStreamReader(); 

            return reader.Read(streamReader.BaseStream, out var diagnostic); 

        } 

    } 

    static void SerializeOpenAPIDocument(OpenApiDocument document, string filePath) 

    { 

        using (var streamWriter = new StreamWriter(filePath)) 

        {

            var writer = new OpenApiYamlWriter(streamWriter); 

            document.SerializeAsV3(writer); 

        } 

    } 

}
```
This is how the modified OpenAPI description for PetStore service looks now:

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

  /pets/post: 

    post: 

      description: Add a new pet 

      requestBody: 

        content: 

          application/json: 

            schema: 

              type: object 

              properties: 

                name: 

                  type: string 

                category: 

                  $ref: '#/components/schemas/Category' 

        required: true 

      responses: 

        '201': 

          description: Pet created successfully
```
In this example we modified an existing YAML OpenAPI document and output it in YAML format. OpenAPI.NET also supports converting OpenAPI files between YAML and JSON formats. Here’s an example of how to [convert the YAML OpenAPI file to JSON format](converting-an-openAPI document.md).
