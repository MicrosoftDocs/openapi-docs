---
title: Middleware
description: Learn about implementing middleware into Kiota.
author: baywet
ms.author: vibiret
ms.topic: overview
ms.date: 11/21/2024
---

# Implementing middleware

By registering custom middleware handlers, you can perform operations before a request is made. For example, auditing and filtering the request before the client sends it.

## Middleware

Create your middleware class and add your business requirements. For example, you might wish to add custom headers to the request or filter headers and content.

<!-- markdownlint-disable MD051 -->
<!-- markdownlint-disable MD024 -->
### [.NET](#tab/csharp)

```csharp
public class SaveRequestHandler : DelegatingHandler
{
    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
    {
        var jsonContent = await request.Content.ReadAsStringAsync(cancellationToken);
        Console.WriteLine($"Request: {jsonContent}");

        return await base.SendAsync(request, cancellationToken);
    }
}
```

### [TypeScript](#tab/typescript)

```typescript
export class SaveRequestHandler implements Middleware {
    next: Middleware | undefined;
    public execute(url: string, requestInit: RequestInit, requestOptions?: Record<string, RequestOption>): Promise<Response> {
        console.log(`Request: ${requestInit.body}`);
        return await this.next?.execute(url, requestInit as RequestInit, requestOptions);
    }
}
```

---

## Register middleware

Create a middleware handlers array and use the existing middleware already implemented within the HTTP library that includes existing handlers like retry, redirect, and more.

### [.NET](#tab/csharp)

```csharp
var handlers = KiotaClientFactory.CreateDefaultHandlers();
handlers.Add(new SaveRequestHandler());
```

### [TypeScript](#tab/typescript)

```typescript
const handlers = MiddlewareFactory.getDefaultMiddlewares();
handlers.unshift(new SaveRequestHandler());
```

---

Next you need to create a delegate chain so the middleware handlers are registered in the right order.

### [.NET](#tab/csharp)

```csharp
var httpMessageHandler =
    KiotaClientFactory.ChainHandlersCollectionAndGetFirstLink(
        KiotaClientFactory.GetDefaultHttpMessageHandler(),
        handlers.ToArray());
```

### [TypeScript](#tab/typescript)

```typescript
// this step is not required for TypeScript
```

---

Finally, create a request adapter using an HTTP client. This adapter can then be used when submitting requests from the generated client. This design means different adapters/middleware can be used when calling APIs and therefore gives flexibility to how and when a middleware handler is used.

### [.NET](#tab/csharp)

```csharp
var httpClient = new HttpClient(httpMessageHandler!);
var adapter = new HttpClientRequestAdapter(authProvider, httpClient:httpClient);
var client = new PostsClient(adapter); // the name of the client will vary based on your generation parameters
```

### [TypeScript](#tab/typescript)

```typescript
const httpClient = KiotaClientFactory.create(undefined, handlers);
const adapter = new FetchRequestAdapter(authProvider, undefined, undefined, httpClient);
const client = createApiClient(adapter);
```

---

## Middleware handlers

The following table describes which middleware handlers are implemented by the HTTP package, and which ones are included by default with any new request adapter.

| Name | Description | C# Support | Go Support | Java Support | TypeScript Support | PHP Support | Python Support |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Authorization | Adds an access token to the request when using a native HTTP client. | Yes | No | Yes | No | No | No |
| Body Inspection | Allows the client application to inspect the request and response body through the associated options. | Yes | No | No | No | No | No |
| Chaos | For testing purposes only. Randomly fails requests to allow testing clients' resilience. | Yes | Yes | Yes | Yes | Yes | No |
| Headers Inspection | Allows the client application to inspect the request and response headers through the associated options. | Default | Default | Default | Default | Default | Default |
| Parameters Name Decoding | Decodes query parameter names that were encoded when building the URL because they contained characters not allowed by RFC 6570. | Default | Default | Default | Default | Default | Default |
| Redirect | Automatically follows Location response headers when a **301** or **302** response status code is received. | Default | Default | Default | Default | Default | Default |
| Request Compression | Compresses request bodies and retries requests without compression when a **415** response status code is received. | No | Default | No | Default | No | No |
| Response Decompression | Adds an **Accept-Encoding** request header and decompresses any response body with a **Content-Encoding** response header. | N/A | N/A | N/A | N/A | N/A | N/A |
| Retry | Automatically retries requests when a **429** or a **503** response status code is received. | Default | Default | Default | Default | Default | Default |
| Sunset | Logs a warning message with Open Telemetry upon receiving a **Sunset** response header. | No | No | No | No | No | No |
| Telemetry | Adds an extension point to augment the request with telemetry headers. | Yes | No | No | Yes | Yes | No |
| Uri Replacement | Enables replacement of segments of the request URI before it's sent out. | Default | Yes | Default | Yes | Yes | Default |
| User Agent | Adds the kiota http library version to the user agent request header. | Default | Default | Default | Default | Default | Default |

> [!NOTE]
> Languages with **N/A** in the response decompression column leverage the native response decompression offered by the HTTP client instead.

> [!NOTE]
> Languages with **Default** for the handler support value include the handler by default when creating the request adapter with no additional configuration.

> [!NOTE]
> Request body decompression is not enabled by default in ASP.NET Core APIs and needs to be enabled. Find out how to enable it with [Request Decompression in ASP.NET core](/aspnet/core/fundamentals/middleware/request-decompression).
