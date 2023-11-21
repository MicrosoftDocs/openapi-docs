---
title: Middleware
description: Learn about implementing middleware into Kiota.
author: gpltaylor
ms.author: gpltaylor
ms.topic: overview
date: 21/11/2023
---

# Implementing Middleware
Implement middleware to access the pure request.

## Middleware class
Create your middleware class and add your business logic

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
Create a Middleware delegate array however, make sure to use the existing Middleware already implemented within Kiota.HttpClient. This includes Retry, redirect handling and more.

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

Finally create the HttpClient and then the Adapter 

```
var httpClient = new HttpClient(handler!);
var adapter = new HttpClientRequestAdapter(authProvider, httpClient:httpClient);
var client = new PostsClient(adapter);
```
