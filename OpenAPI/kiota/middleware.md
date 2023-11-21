---
title: Middleware
description: Learn about implementing middleware into Kiota.
author: baywet
ms.author: vibiret
ms.topic: overview
date: 21/11/2023
---

# Implementing Middleware

By registering custom middleware handlers you can perform operations before a request is made. For example, auditing and filtering the request before the client sends it.

## Middleware

Create your middleware class and add your business requirements. For example you may wish to add custom Headers to the request or filter headers and content.

```cs
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

## Register Middleware

Create a middleware handlers array and use the existing middleware already implemented within **Microsoft.Kiota.HttpClientLibrary** that includes existing handlers like retry, redirect and more.

```cs
var handlers = KiotaClientFactory.CreateDefaultHandlers();
handlers.Add(new SaveRequestHandler());
```

Next you will need to create a Delegate chain so the middleware handlers are registered in the right order.

```cs
var httpMessageHandler =
    KiotaClientFactory.ChainHandlersCollectionAndGetFirstLink(
        KiotaClientFactory.GetDefaultHttpMessageHandler(),
        handlers.ToArray());
```

Finally, create a request adapter using a HttpClient with the message handler that was just created. This adapter can then be used when submitting requests from the generated client. This design means different adapters/middleware can be used when calling APIs and therefore gives flexibility to how and when a middleware handler is used.

```cs
var httpClient = new HttpClient(httpMessageHandler!);
var adapter = new HttpClientRequestAdapter(authProvider, httpClient:httpClient);
var client = new PostsClient(adapter); // the name of the client will vary based on your generation parameters
```
