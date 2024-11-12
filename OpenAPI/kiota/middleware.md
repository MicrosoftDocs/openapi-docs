---
title: Middleware
description: Learn about implementing middleware into Kiota.
author: baywet
ms.author: vibiret
ms.topic: overview
ms.date: 11/21/2023
---

# Implementing middleware

By registering custom middleware handlers, you can perform operations before a request is made. For example, auditing and filtering the request before the client sends it.

## Middleware

Create your middleware class and add your business requirements. For example, you might wish to add custom headers to the request or filter headers and content.

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

## Register middleware

Create a middleware handlers array and use the existing middleware already implemented within **Microsoft.Kiota.HttpClientLibrary** that includes existing handlers like retry, redirect, and more.

```csharp
var handlers = KiotaClientFactory.CreateDefaultHandlers();
handlers.Add(new SaveRequestHandler());
```

Next you need to create a delegate chain so the middleware handlers are registered in the right order.

```csharp
var httpMessageHandler =
    KiotaClientFactory.ChainHandlersCollectionAndGetFirstLink(
        KiotaClientFactory.GetDefaultHttpMessageHandler(),
        handlers.ToArray());
```

Finally, create a request adapter using an `HttpClient` with the message handler. This adapter can then be used when submitting requests from the generated client. This design means different adapters/middleware can be used when calling APIs and therefore gives flexibility to how and when a middleware handler is used.

```csharp
var httpClient = new HttpClient(httpMessageHandler!);
var adapter = new HttpClientRequestAdapter(authProvider, httpClient:httpClient);
var client = new PostsClient(adapter); // the name of the client will vary based on your generation parameters
```
