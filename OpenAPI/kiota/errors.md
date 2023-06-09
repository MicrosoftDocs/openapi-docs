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

The abstractions library for each language defines an API exception type (or error) which inherits or implements the default error/exception type for the language. Kiota will also generate types for schemas mapped to HTTP specific error status codes (400-600) or to ranges (4XX, 5XX), derived from the API exception defined in the abstractions library.

> [!NOTE]
> If the error schema is an `allOf`, Kiota will flatten all the `allOf` entries recursively into the generated type as most languages do not support multiple parents inheritance.

This mapping of codes to types will be passed to the request adapter as a dictionary/map so it can be used at runtime and is specific to each endpoint.

## Runtime behavior

At runtime, if an error status code is returned by the API, the request adapter will follow this sequence:

1. If a mapping is present for the specific status code and if the response body can be deserialized to that type, deserialize and throw.
1. If a mapping is present for the corresponding range (4XX, 5XX) and if the response body can be deserialized to that type, deserialize and throw.
1. Otherwise throw a new instance of the API exception defined in the abstractions library.

The generated clients throw exceptions/errors when running into failed response codes to allow the consumer to tell the difference between a successful response that didn't return a body (204, etc...) and a failed response. The consumer can choose to implement try/catch behavior that either catches the generic API exception type or is more specific to an exception type mapped to a range or even a status code depending on the scenario.

## Transient errors

Most request adapters handle transient errors (e.g. `429` because of throttling, network interruption, etc.) through two main mechanisms. When transient errors are handled and requests are retried, the request adapter itself and the generated client are not aware that an error happened and consequently won't throw any exception.

- The native clients of most platforms provide support for some transient errors handling (e.g. reconnecting when the network was interrupted).
- Request adapters provide a middleware infrastructure to implement cross-cutting concerns like transient error handling (e.g. exponential back-off when throttled) which is enabled by default.
