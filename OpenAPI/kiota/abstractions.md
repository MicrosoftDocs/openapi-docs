---
title: Kiota abstractions
description: Learn how Kiota decouples the API clients and models from the underlying implementations of HTTP, authentication, and serialization.
author: baywet
ms.author: vibiret
ms.topic: conceptual
date: 03/21/2023
---

# Kiota abstractions

On most platforms, there are a range of different HTTP client library implementations. Developers often have preferences on which is the best implementation to meet their needs. Kiota's objective is to make it easier to create an HTTP request object but attempt to be agnostic of the library that makes the HTTP call. In order to decouple Kiota from specific HTTP libraries, we defined a set of abstractions.

Kiota attempts to minimizing the amount of generated code to decrease processing time and reduce the binary footprint of the client libraries. In order to achieve this goal, we attempt to put as much code as a possible into the core libraries. Kiota ships with a default set of core libraries that include default implementations for HTTP transport, authentication, and serialization. Replacing these core libraries with ones optimized for your scenarios is a supported scenario.

The core libraries take care of all generic processing of HTTP requests. The service library that Kiota generates is designed to create a strongly typed layer over the core libraries to simplify the process of creating requests and consuming responses.

## Requests

This section provides information about the types offered in the abstractions library the generated result depends on which are related to executing requests.

### Request adapter

The request adapter interface is the primary point where Kiota service libraries trigger the creation of an HTTP request. The following code snippet is the [C# implementation](https://github.com/microsoft/kiota/blob/main/abstractions/dotnet/src/IRequestAdapter.cs).

```csharp
public interface IRequestAdapter
{
    void EnableBackingStore(IBackingStoreFactory backingStoreFactory);

    ISerializationWriterFactory SerializationWriterFactory { get; }

    Task<ModelType> SendAsync<ModelType>(
        RequestInformation requestInfo,
        ParsableFactory<ModelType> factory,
        IResponseHandler responseHandler = default,
        Dictionary<string, ParsableFactory<IParsable>> errorMappings = default) where ModelType : IParsable;

    Task<IEnumerable<ModelType>> SendCollectionAsync<ModelType>(
        RequestInformation requestInfo,
        ParsableFactory<ModelType> factory,
        IResponseHandler responseHandler = default,
        Dictionary<string, ParsableFactory<IParsable>> errorMappings = default) where ModelType : IParsable;

    Task<ModelType> SendPrimitiveAsync<ModelType>(
        RequestInformation requestInfo,
        IResponseHandler responseHandler = default,
        Dictionary<string, ParsableFactory<IParsable>> errorMappings = default);

    Task SendNoContentAsync(
        RequestInformation requestInfo,
        IResponseHandler responseHandler = default,
        Dictionary<string, ParsableFactory<IParsable>> errorMappings = default);
}
```

Kiota service libraries return the model type that is associated with an HTTP resource. This behavior can be overridden by changing the `responseHandler` to do something different than default behavior. One use of this approach is to change the response type to be either a native HTTP response class, or return a generic API response class that provides access to more underlying metadata.

> [!NOTE]
> This interface is meant to support the generated code and not to be used by application developers. Should you need to make arbitrary requests because they are not part of the generated client, please use a native HTTP client instead as it will be better suited for that purpose. Additionally you should obtain that native client from the provided factory, or use the middleware factory to augment an existing client. For more information about middleware, see [Implementing middleware](./middleware.md).

### Request information

In order to enable Kiota service libraries to make requests, they need to be able to accumulate information about the request and pass it to the core library. The request information object is designed to do that. It only contains properties that are provided by the request builders. As request builders get more sophisticated, so can the request information class.

```csharp
public class RequestInformation
{
    public string UrlTemplate { get; set; }
    public Uri URI { get; set; }
    public HttpMethod HttpMethod { get; set; }
    public IDictionary<string, object> QueryParameters { get; set; } =
        new Dictionary<string, object>(StringComparer.OrdinalIgnoreCase);
    public IDictionary<string, string> Headers { get; set; } =
        new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);
    public IDictionary<string, object> PathParameters { get; set; } =
        new Dictionary<string, object>(StringComparer.OrdinalIgnoreCase);
    public Stream Content { get; set; }
    public IEnumerable<IRequestOption> RequestOptions { get }
}
```

### Response handler

When the request information is passed to the execution method from the fluent style API, the core library does all of the default hard work. A custom response handler can change the behavior of and access the native response object.

```csharp
public interface IResponseHandler
{
    Task<ModelType> HandleResponseAsync<NativeResponseType, ModelType>(
        NativeResponseType response,
        Dictionary<string, ParsableFactory<IParsable>> errorMappings);
}
```

### Failed responses handling

Kiota implements default [error handling](errors.md). If a response handler is passed, the error detection logic is bypassed to allow the caller to implement whichever custom handling they desire.

## Backing store

This interface defines the members a backing store needs to implement for a model to be able to store its field values in a custom data store instead of using fields.

```csharp
public interface IBackingStore
{
    T Get<T>(string key);
    void Set<T>(string key, T value);
    IEnumerable<KeyValuePair<string, object>> Enumerate();
    IEnumerable<string> EnumerateKeysForValuesChangedToNull();
    string Subscribe(Action<string, object, object> callback);
    void Subscribe(Action<string, object, object> callback, string subscriptionId);
    void Unsubscribe(string subscriptionId);
    void Clear();
    bool InitializationCompleted { get; set; }
    bool ReturnOnlyChangedValues { get; set; }
}
```

### In-memory backing store

Default implementation that stores information in a map/dictionary in memory that enables for dirty tracking of changes on a model and reuse of models.

## Authentication

See the [authentication](./authentication.md) documentation page for information on interfaces that allow for implementing authentication libraries.

## Serialization

See the [serialization](serialization.md) documentation page for information on interfaces that allow for implementing serialization libraries.
