---
title: Middleware
description: Learn about implementing middleware into Kiota.
author: gpltaylor
ms.author: gpltaylor
ms.topic: overview
date: 21/11/2023
---

# Implementing Middleware
Here we review implementing Middleware into Kiota. Using this example we are going to access the request before we proccess the request. This allows us to audio the request or to maker changes before submitting.

## Middleware class
Create your middleware class and add your business requirements. For example you may wish to add custom Headers to the request.

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

Finally create the HttpClient and then the Adapter 

```
var httpClient = new HttpClient(handler!);
var adapter = new HttpClientRequestAdapter(authProvider, httpClient:httpClient);
var client = new PostsClient(adapter);
```
