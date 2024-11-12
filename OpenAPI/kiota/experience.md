---
title: Kiota API client experience
description: Learn about the developer experience provided by Kiota-generated API clients.
author: baywet
ms.author: vibiret
ms.topic: conceptual
ms.date: 03/10/2023
---

# Kiota API client experience

## Create a resource

Here's an example of the basic read and write syntax for a resource.

```csharp
// An authentication provider from the supported language table
// https://github.com/microsoft/kiota#supported-languages, or your own implementation
var authProvider = ;
var requestAdapter = new HttpClientRequestAdapter(authProvider);
var client = new ApiClient(requestAdapter);
var user = await client.Users["bob@contoso.com"].GetAsync();

var newUser = new User
{
    FirstName = "Bill",
    LastName = "Brown"
};

await client.Users.PostAsync(newUser);
```

## Access related resources

Resources are accessed via relation properties starting from the client object. Collections of resources are accessible by an indexer and a parameter. Once the desired resource is referenced, the supported HTTP methods are exposed by corresponding methods. Deeply nested resource hierarchies are accessible by continuing to traverse relationships.

```csharp
// An authentication provider from the supported language table
// https://github.com/microsoft/kiota#supported-languages, or your own implementation
var authProvider = ;
var requestAdapter = new HttpClientRequestAdapter(authProvider);
var client = new ApiClient(requestAdapter);
var message = await client.Users["bob@contoso.com"]
    .MailFolders["Inbox"]
    .Messages[23242]
    .GetAsync();
```

The client object is a [request builder](request-builders.md) object, and forms the root of a hierarchy of request builder objects. These request builders can access any number of APIs that are merged into a common URI space.

Requests can be further refined by providing query parameters. Each HTTP operation method that supports query parameters accepts a lambda that can configure an object with the desired query parameters.

```csharp
// An authentication provider from the supported language table
// https://github.com/microsoft/kiota#supported-languages, or your own implementation
var authProvider = ;
var requestAdapter = new HttpClientRequestAdapter(authProvider);
var client = new ApiClient(requestAdapter);
var message = await client.Users["bob@contoso.com"]
    .Events
    .GetAsync(q =>
    {  q.StartDateTime = DateTime.Now;
        q.EndDateTime = DateTime.Now.AddDays(7);
    });
```

Using a configured query parameter object prevents tight coupling on the order of query parameters and makes optional parameters easy to implement across languages.

## Response with no schema

```yaml
openapi: 3.0.3
info:
  title: The simplest thing that works
  version: 1.0.0
servers:
  - url: https://example.org/
paths:
  /speakers:
    get:
      responses:
        200:
          description: Ok
```

If the OpenAPI description doesn't describe the response payload, then it should be assumed to be of content type `application/octet-stream`.

Client library implementations should return the response payload in the language-native way of providing an untyped set of bytes. This response format could be a byte array or some kind of stream response.

```csharp
Stream speaker = await apiClient.Speakers.GetAsync();
```

> [!WARNING]
> Support for stream responses identified by application/octet-stream are not supported yet

## Response with simple schema

```yaml
openapi: 3.0.3
info:
  title: Response with simple schema
  version: 1.0.0
servers:
  - url: https://example.org/
paths:
  /speakers/{speakerId}:
    get:
      parameters:
        - name: speakerId
          in: path
          required: true
          schema:
            type: string
      responses:
        200:
          description: Ok
          content:
            application/json:
              schema:
                type: object
                properties:
                  displayName:
                    type: string
```

```csharp
Speaker speaker = await apiClient.Speakers["23"].GetAsync();
string displayName = speaker.DisplayName;
```

## Response with primitive payload

Responses with a content type of `text/plain` should be serialized into a primitive data type. Unless a Schema object indicates a more precise data type, the payload should be serialized to a string.

```yaml
openapi: 3.0.3
info:
  title: Response with primitive payload
  version: 1.0.0
servers:
  - url: https://example.org/
paths:
  /speakers/count:
    get:
      responses:
        200:
          description: Ok
          content:
            text/plain:
              schema:
                type: number
```

```csharp
int speakerCount = await apiClient.Speakers.Count.GetAsync();
```

> [!WARNING]
> Support for `text/plain` responses are not supported yet

## Filtered collection

```yaml
openapi: 3.0.3
info:
  title: Collection filtered by query parameter
  version: 1.0.0
servers:
  - url: https://example.org/
paths:
  /speakers:
    get:
      parameters:
        - name: location
          in: query
          required: false
          schema:
            type: string
      responses:
        200:
          description: Ok
          content:
            application/json:
              schema:
                type: array
                item:
                  $ref: "#/components/schemas/speaker"
components:
  schemas:
    speaker:
      type: object
      properties:
        displayName:
          type: string
        location:
          type: string
```

```csharp
IEnumerable<Speaker> speakers = await apiClient.Speakers.GetAsync(x => { x.Location="Montreal"; });
```

## Heterogeneous collection

Kiota client libraries automatically downcast heterogeneous collection items (or single properties) to the target type if the following criteria are met. This way client library users can easily access the other properties available on the specialized type.

- A discriminator is present in the description
- The response payload contains a matching mapping entry.

```yaml
openapi: 3.0.3
info:
  title: Heterogeneous collection
  version: 1.0.0
servers:
  - url: https://example.org/
paths:
  /sessions:
    get:
      parameters:
        - name: location
          in: query
          required: false
          schema:
            type: string
      responses:
        200:
          description: Ok
          content:
            application/json:
              schema:
                type: array
                item:
                  $ref: "#/components/schemas/session"
components:
  schemas:
    entity:
      type: object
      properties:
        id:
          type: string
    session:
      allof:
        - $ref: "#/components/schemas/entity"
      type: object
      properties:
        '@OData.Type':
          type: string
          enum:
           - session
        displayName:
          type: string
        location:
          type: string
    workshop:
      allof:
        - $ref: "#/components/schemas/session"
      type: object
      properties:
        '@OData.Type':
          type: string
          enum:
           - workshop
        requiredEquipment:
          type: string
    presentation:
      allof:
        - $ref: "#/components/schemas/session"
      type: object
      properties:
        '@OData.Type':
          type: string
          enum:
           - presentation
        recorded:
          type: boolean

```

```csharp
IEnumerable<Session> sessions = await apiClient.Sessions.GetAsync();
List<Presentation> presentations = sessions.OfType<Presentation>().ToList();
// OfType is a native method that filters a collection based on the item type returning a subset
```

## Explicit Error Response

```yaml
openapi: 3.0.3
info:
  title: The simplest thing that works
  version: 1.0.0
servers:
  - url: https://example.org/
paths:
  /speakers:
    get:
      responses:
        "2XX":
          description: Success
        "4XX":
          $ref: "#/components/responses/errorResponse"
        "5XX":
          $ref: "#/components/responses/errorResponse"
components:
  responses:
    errorResponse:
      description: error
      content:
        application/json:
          schema:
            type: object
            properties:
              code:
                type: string
              message:
                type: string
```

```csharp
try
{
    var speakersStream = await apiClient.Speakers.GetAsync();
}
catch ( ServerException exception )
{
    Console.WriteLine(exception.Error.Message)
}
```

> [!WARNING]
> Support for ServerException is not supported yet

## Readability of the API client

Kiota is designed as a lightweight and fast API client generator that focuses on the discovery, exploration, and calling of any HTTP API with minimal effort. The primary goal of Kiota is to provide developers with a tool that simplifies the process of working with APIs, rather than producing human-readable code. This approach allows Kiota to support a wide range of languages and to handle the complexities of all APIs efficiently. By keeping code readability as a non-goal, Kiota can prioritize performance, scalability, and ease of use, ensuring that developers can focus on the functionality of their applications rather than the underlying auto-generated code.

Developers are encouraged to treat Kiota-generated code as an opaque box that reliably handles API interactions. Peeking into the auto-generated code is discouraged because it is optimized for machines, not human interpretation, which could lead to confusion and misinterpretation. Instead, developers should utilize the well-documented interfaces provided by Kiota to interact with their APIs. This abstraction allows for a cleaner development experience and ensures that any updates or changes to the Kiota generator do not impact the developer's workflow. Trust in the robustness of Kiota's output allows developers to dedicate their time to crafting the best possible applications with the assurance that the API communication is handled efficiently in the background.