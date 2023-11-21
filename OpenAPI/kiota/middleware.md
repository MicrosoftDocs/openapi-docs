---
title: Middleware1
description: Learn about implementing middleware into Kiota.
author: gpltaylor
ms.author: gpltaylor
ms.topic: overview
date: 21/11/2023
---

# Implementing Middleware
By registering custom middleware delegates we can perform operations before a request is made. For example, auditing and filtering the request before we send.

## Middleware
Create your middleware class and add your business requirements. For example you may wish to add custom Headers to the request or filter headers and content.

```
public class SaveRequestHandler : DelegatingHandler
{
    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
    {
        string jsonContent = await request.Content.ReadAsStringAsync();
        Console.WriteLine($"Request: {jsonContent}");

        return await base.SendAsync(request, cancellationToken);
    }
}
```
## Register Middleware
Create a Middleware delegate array and use the existing Middleware already implemented within Kiota.HttpClient that includes existing Delegates like Retry, redirect handling and more.

```
var delegates = KiotaClientFactory.CreateDefaultHandlers();
delegates.Add(new SaveRequestHandler());
```
Next we create a Delegate chain so middleware is registered in the right order

```
var handler =
    KiotaClientFactory.ChainHandlersCollectionAndGetFirstLink(
        KiotaClientFactory.GetDefaultHttpMessageHandler(),
        delegates.ToArray());
```

Finally we create Adapter using our HttpClient that's registered our Middleware. This adapter can then be using when submitting requests. The power of this design means that different adapters/middleware can be used when calling Endpoints and therefore gives flexibility to how and when Middleware is used.

```
var httpClient = new HttpClient(handler!);
var adapter = new HttpClientRequestAdapter(authProvider, httpClient:httpClient);
var client = new PostsClient(adapter);
```
