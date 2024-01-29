# ConvertOpenApiDocument
OpenAPI.NET provides the ability to convert OpenAPI documents between different formats, such as from YAML to JSON. This can be useful when you need to share or collaborate on an OpenAPI document with others who may prefer a different format. 

Here is an example of how you can convert an OpenAPI document from YAML to JSON using OpenAPI.NET: 

In this example we will convert the OpenAPI Document we created and modified from the **CreateOpenApiDocument** and **ModifyOpenApiDocument** from YAML to JSON. This is the YAML file we will convert to JSON:

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

Here is how we convert the file into JSON format:

```csharp
using System; 

using Microsoft.OpenApi.Models; 

using Microsoft.OpenApi.Readers; 

using System.IO; 

using Microsoft.OpenApi.Writers; 

class ConvertOpenApiDocument 

{ 

    static void Main() 

    { 

        // Load the existing OpenAPI document from a YAML file 

        var filePath = "updated_petstore.yaml"; 

        if (File.Exists(filePath)) 

        { 

            var document = ReadOpenApiDocumentFromYamlFile(filePath); 

            // Convert the document to JSON 

            var jsonOpenApi = ConvertToJSON(document); 

            // Save the JSON OpenAPI document to a new file 

            SaveJsonDocumentToFile(jsonOpenApi, "updated_petstore.json"); 

            Console.WriteLine("OpenAPI document converted from YAML to JSON."); 

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

    static string ConvertToJSON(OpenApiDocument document) 

    { 

        using (var stringWriter = new StringWriter()) 

        { 

            var writer = new OpenApiJsonWriter(stringWriter); 

            document.SerializeAsV3(writer); 

            return stringWriter.ToString(); 

        } 

    } 

    static void SaveJsonDocumentToFile(string jsonDocument, string filePath) 

    { 

        File.WriteAllText(filePath, jsonDocument); 

    } 

} 
```
And here is the output from the conversion process:

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

                  "type": "array" 

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

                "type": "object", 

                "properties": { 

                  "name": { 

                    "type": "string" 

                  }, 

                  "category": null 

                } 

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

  } 

} 
```
