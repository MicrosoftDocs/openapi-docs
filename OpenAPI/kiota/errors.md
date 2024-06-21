---
title: Error handling with Kiota API clients
description: Learn about error handling with Kiota API clients.
author: baywet
ms.author: vibiret
ms.topic: conceptual
date: 03/21/2023
---

# Error handling with Kiota API clients

Kiota API clients include error types for errors defined in OpenAPI descriptions.

## Error types mapping

The abstractions library for each language defines an API exception type (or error) which inherits or implements the default error/exception type for the language. Kiota generates types for schemas mapped to HTTP specific error status codes (400-600) or to ranges (4XX, 5XX).

> [!NOTE]
> If the error schema is an `allOf`, Kiota will flatten all the `allOf` entries recursively into the generated type as most languages do not support multiple parents inheritance.

This mapping of codes to types is passed to the request adapter as a dictionary/map so it can be used at runtime and is specific to each endpoint. An error mapping is only considered valid if the response has a schema of type `object` and at least one field. All generated error/exception types inherit from the base API exception/error defined in the abstractions library.

## Runtime behavior

At runtime, if the API returns an error status code, the request adapter follows this sequence:

1. If a mapping is present for the specific status code and if the response body can be deserialized to that type, deserialize then throw.
1. If a mapping is present for the corresponding range (4XX, 5XX) and if the response body can be deserialized to that type, deserialize then throw.
1. If a mapping is present for "default" responses and if the response body can be deserialized to that type, deserialize then throw.
1. If none of the previous criteria are met, throw a new instance of the API exception defined in the abstractions library.

The generated clients throw exceptions/errors for failed response codes to allow the consumer to tell the difference between a successful response that didn't return a body (204, etc.) and a failed response. The consumer can implement try/catch behavior that either catches the generic API exception type, or is more specific to an exception type mapped to a range or even a status code, depending on the scenario.

## Transient errors

Most request adapters handle transient errors (for example `429` because of throttling, network interruption, etc.) through two main mechanisms. In this scenario, the request adapter and the generated client aren't aware that an error happened and don't throw an exception.

- The native clients of most platforms provide support for some transient errors handling (for example, reconnecting when the network was interrupted).
- Request adapters provide a middleware infrastructure to implement cross-cutting concerns like transient error handling (for example, exponential back-off when throttled) which is enabled by default.

## Error message

Kiota generates code to extract the value of a property from the generated error type as the main exception/error message/reason if a string property is identified with the `x-ms-primary-error-message` extension.

In this example, the value for the **errorMessage** property returned from the API is used as the error message for the exception if any 404 response is returned.

```yaml
openapi: 3.0.3
info:
  title: OData Service for namespace microsoft.graph
  description: This OData service is located at https://graph.microsoft.com/v1.0
  version: 1.0.1
paths:
  /foo:
    get:
      responses:
        '404':
          content:
            application/json:
              schema:
                type: object
                properties:
                  bar:
                    type: string
                  errorMessage:
                    type: string
                    x-ms-primary-error-message: true
servers:
  - url: https://graph.microsoft.com/v1.0
```

## API Exception properties

The base API exception/error type from which all generated error types derive defines the following properties. The request adapter for the language automatically populates these properties.

| Name | Type | Description |
| ---- | ---- | ----------- |
| ResponseStatusCode | int32 | The status code from the HTTP response. |
| ResponseHeaders | map(string, string[]) | The response headers. |

Additionally these types also derive from the native exception/error type of the platform, and can be expected to provide the same properties.

## Solutions to missing error mappings

When a description doesn't document a mapping for an error status code encountered at runtime, the client application receives an exception/error with the following error message: "The server returned an unexpected status code and no error factory is registered for this code." There are many ways to deal with this situation:

- You can reach out to the API producer to ask them to improve their description and add the missing code to the API description. The API client can then be refreshed to include the error mapping.
- You can maintain a local copy of the description and add the missing code to the API description. The API client can then be refreshed to include the error mapping.
- The client application can rely on the status code and header fields to drive its recovery.
