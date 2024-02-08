---
title: Creating an OpenApi Document
description: Learn how to use OpenAPI.NET library to create an OpenAPI documents
author: njaci1
ms.author: kelvinnjaci
ms.topic: conceptual
date: 01/02/2024
---
# Creating an OpenApi Document
Let’s say we have a service called “PetStore” that allows users to view and  post pets to the online store.
To create a new OpenAPI document for the PetStore service, we can use the object model provided by OpenAPI.NET.We can create a new OpenApiDocument object, and then add paths, operations, parameters, and responses to it to describe the different endpoints of the PetStore API. Here we will create the OpenAPI document info and server components and add only one path for retrieving all pets(“/pets”). We will write this document out as a YAML file(petstore.yaml)

Here is the code to create an OpenAPI document for the PetStore API:

```csharp
using System; 

using Microsoft.OpenApi.Models; 

using Microsoft.OpenApi.Writers; 

using System.IO; 


class CreateOpenApiDocument 

{ 

    static void Main() 

    { 

        // Create an OpenAPI document for the PetStore API 

        var document = new OpenApiDocument 

        { 

            Info = new OpenApiInfo 

            { 

                Title = "PetStore API", 

                Version = "1.0.0" 

            }, 

            Servers = new List<OpenApiServer> 

            { 

                new OpenApiServer { Url = "https://api.petstore.com" } 

            }, 

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

                                                        Id = "Pet" 

                                                    } 

                                                } 

                                            } 

                                        } 

                                    } 

                                } 

                            } 

                        } 

                    } 

                } 

            } 

        }; 

        // Serialize the OpenAPI document to a YAML file 

        using (var streamWriter = new StreamWriter("petstore.yaml")) 

        { 

            var writer = new OpenApiYamlWriter(streamWriter); 

            document.SerializeAsV3(writer); 

        } 
        Console.WriteLine("PetStore OpenAPI document created and saved."); 

    } 

} 
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
```
Check out the [Modifying an OpenApi Document](modifying-an-openAPI-document.md) example to learn how to add a new path to the OpenAPI document we created earlier or checkout [Converting an OpenApi Document](converting-an-openAPI-document.md) to learn how to use the OpenAPI.NET library to convert between YAML and JSON formats.
