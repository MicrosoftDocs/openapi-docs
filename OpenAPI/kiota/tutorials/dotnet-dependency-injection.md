---
title: Use Kiota with dependency injection in .NET
description: Learn how you can combine Kiota with dependency injection in .NET to build API clients.
author: svrooij
ms.author: vibiret
ms.topic: tutorial
date: 05/10/2024
---

# Register a Kiota API client in .NET with dependency injection

In this tutorial, you will learn how to register a [Kiota API client](/openapi/kiota/tutorials/dotnet-azure?wt.mc_id=SEC-MVP-5004985) with [dependency injection](/dotnet/core/extensions/dependency-injection-usage?wt.mc_id=SEC-MVP-5004985).

## Required tools

- [.NET SDK 8.0](https://get.dot.net/8)
- [Kiota CLI](/openapi/kiota/install?tabs=bash&wt.mc_id=SEC-MVP-5004985#install-as-net-tool)

## Create a project

Run the following command in the directory you want to create a new project.

```dotnetcli
dotnet new webapi -o kiota-with-di
cd kiota-with-di
dotnet new gitignore
```

## Add dependencies

Before you can compile and run the generated API client, ensure the generated source files are part of a project with the required dependencies. Your project must reference the [abstraction package](https://github.com/microsoft/kiota-abstractions-dotnet) and default implementations.

- HTTP ([Kiota default HttpClient-based implementation](https://github.com/microsoft/kiota-http-dotnet))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-json-dotnet))
- Form serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-form-dotnet))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-text-dotnet))
- Multipart serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-multipart-dotnet))
- HttpClient support for Dependency Injection [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http)

Run the following commands to get the required dependencies.

```dotnetcli
dotnet add package Microsoft.Extensions.Http
dotnet add package Microsoft.Kiota.Abstractions
dotnet add package Microsoft.Kiota.Http.HttpClientLibrary
dotnet add package Microsoft.Kiota.Serialization.Json
dotnet add package Microsoft.Kiota.Serialization.Form
dotnet add package Microsoft.Kiota.Serialization.Text
dotnet add package Microsoft.Kiota.Serialization.Multipart
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named _github-releases.yml_ and add the following.

```yaml
openapi: 3.0.3
info:
  title: GitHub Release API
  version: 1.0.0
servers:
  - url: https://api.github.com
    description: GitHub API
    variables:
      version:
        default: '2022-11-28'
        description: GitHub API version
paths:
  /repos/{owner}/{repo}/releases/latest:
    get:
      summary: Get the latest release
      parameters:
        - name: owner
          in: path
          required: true
          description: The owner of the repository
          schema:
            type: string
        - name: repo
          in: path
          required: true
          description: The repository name
          schema:
            type: string
      responses:
        200:
          description: Success!
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/github.release"
        404:
          description: Not found
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/github.error"

components:
  schemas:
    github.release:
      type: object
      properties:
        id:
          type: integer
          description: The release ID
        tag_name:
          type: string
          description: The release tag name
        name:
          type: string
          description: The release name
        body:
          type: string
          description: The release body
        created_at:
          type: string
          format: date-time
          description: The release creation date
        published_at:
          type: string
          format: date-time
          description: The release publication date
    github.error:
      type: object
      properties:
        message:
          type: string
          description: The error message
        documentation_url:
          type: string
          description: The error documentation URL
```

You can then use the [Kiota](/openapi/kiota/install?tabs=bash&wt.mc_id=SEC-MVP-5004985#install-as-net-tool) command line tool to generate the API client classes.

```bash
kiota generate -l csharp -d github-releases.yml -c GitHubClient -n GitHub.ApiClient -o ./GitHub
```

## Create extension methods

Create a new file named _KiotaServiceCollectionExtensions.cs_ in the root of your project and add the following code.

```csharp
using Microsoft.Kiota.Http.HttpClientLibrary.Middleware;
using Microsoft.Kiota.Http.HttpClientLibrary.Middleware.Options;

/// <summary>
/// Service collection extensions for Kiota handlers.
/// </summary>
public static class KiotaServiceCollectionExtensions
{
    /// <summary>
    /// Adds the Kiota handlers to the service collection.
    /// </summary>
    /// <param name="services"><see cref="IServiceCollection"/> to add the services to</param>
    /// <returns><see cref="IServiceCollection"/> as per convention</returns>
    /// <remarks>The handlers are added to the http client by the <see cref="AttachKiotaHandlers(IHttpClientBuilder)"/> call, which requires them to be pre-registered in DI</remarks>
    public static IServiceCollection AddKiotaHandlers(this IServiceCollection services)
    {
        services.AddTransient<UriReplacementHandler<UriReplacementHandlerOption>>();
        services.AddTransient<RetryHandler>();
        services.AddTransient<RedirectHandler>();
        services.AddTransient<ParametersNameDecodingHandler>();
        services.AddTransient<UserAgentHandler>();
        services.AddTransient<HeadersInspectionHandler>();
        return services;
    }

    /// <summary>
    /// Adds the Kiota handlers to the http client builder.
    /// </summary>
    /// <param name="builder"></param>
    /// <returns></returns>
    /// <remarks>
    /// Requires the handlers to be registered in DI by <see cref="AddKiotaHandlers(IServiceCollection)"/>.
    /// The order in which the handlers are added is important, as it defines the order in which they will be executed.
    /// <see href="https://github.com/microsoft/kiota-http-dotnet/blob/c1c295d3b0ebb2182b66d9a6858241117b59b540/src/KiotaClientFactory.cs#L50-L62">KiotaClientFactory.cs</see> for the default order.
    /// </remarks>
    public static IHttpClientBuilder AttachKiotaHandlers(this IHttpClientBuilder builder)
    {
        builder.AddHttpMessageHandler<UriReplacementHandler<UriReplacementHandlerOption>>();
        builder.AddHttpMessageHandler<RetryHandler>();
        builder.AddHttpMessageHandler<RedirectHandler>();
        builder.AddHttpMessageHandler<ParametersNameDecodingHandler>();
        builder.AddHttpMessageHandler<UserAgentHandler>();
        builder.AddHttpMessageHandler<HeadersInspectionHandler>();
        return builder;
    }
}
```

## Create a client factory

Create a new file named _GitHubClientFactory.cs_ in the _GitHub_ folder and add the following code.

This is what you call a factory pattern, where you create a factory class that is responsible for creating an instance of the client. This way you have a single place that is responsible for creating the client, and you can easily change the way the client is created. In this sample we will be registering the factory in the dependency injection container as well, because it will get the `HttpClient` injected in the constructor.

```csharp
using GitHub.ApiClient;
using Microsoft.Kiota.Abstractions.Authentication;
using Microsoft.Kiota.Http.HttpClientLibrary;

namespace GitHub;

public class GitHubClientFactory
{
    private readonly IAuthenticationProvider _authenticationProvider;
    private readonly HttpClient _httpClient;

    public GitHubClientFactory(HttpClient httpClient)
    {
        _authenticationProvider = new AnonymousAuthenticationProvider();
        _httpClient = httpClient;
    }

    public GitHubClient GetClient() {
      return new GitHubClient(new HttpClientRequestAdapter(_authenticationProvider, httpClient: _httpClient));
    }
}
```

## Register the API client

Open the _Program.cs_ file and add the following code above the `var app = builder.Build();` line.

```csharp
using GitHub;
...

// Add Kiota handlers to the dependency injection container
builder.Services.AddKiotaHandlers();

// Register the factory for the GitHub client
builder.Services.AddHttpClient<GitHubClientFactory>((sp, client) => {
    client.BaseAddress = new Uri("https://api.github.com");
    client.DefaultRequestHeaders.Add("Accept", "application/vnd.github.v3+json");
}).AttachKiotaHandlers();

// Register the GitHub client
builder.Services.AddTransient(sp=> sp.GetRequiredService<GitHubClientFactory>().GetClient());
```

## Create an endpoint

We will now create an endpoint that uses the GitHub client from dependency injection to get the latest release of `dotnet/runtime`.

Open the _Program.cs_ file and add the following code above the `app.Run();` line.

```csharp
app.MapGet("/dotnet/releases", async (GitHub.ApiClient.GitHubClient client, CancellationToken cancellationToken) =>
{
    var releases = await client.Repos["dotnet"]["runtime"].Releases.Latest.GetAsync(cancellationToken: cancellationToken);
    return releases;
})
.WithName("GetDotnetReleases")
.WithOpenApi();
```

## Run the application

Run the following command in your project directory to start the application.

```dotnetcli
dotnet run
```

You can now open a browser and navigate to `http://localhost:{port}/dotnet/releases` to see the latest release of `dotnet/runtime`. Alternatively, you can also go to `http://localhost:{port}/swagger` to see the Swagger UI with this new endpoint.

## Authentication

The `GitHubClientFactory` class is responsible for creating the client, and it is also responsible for creating the `IAuthenticationProvider`. In this sample, we are using the `AnonymousAuthenticationProvider`, which is a simple provider that does not add any authentication headers. More details on authentication with Kiota can be found in the [Kiota authentication documentation](/openapi/kiota/authentication?wt.mc_id=SEC-MVP-5004985).

What to remember is that the `GitHubClientFactory` is used through dependency injection, which means you can have other types injected by the constructor that are needed for the authentication provider.

You might want to create a [BaseBearerTokenAuthenticationProvider](/openapi/kiota/authentication?tabs=csharp&wt.mc_id=SEC-MVP-5004985#base-bearer-token-authentication-provider) that is available in Kiota and expects an `IAccessTokenProvider` which is just an interface with one method "give me an access token". Access tokens usually have a lifetime of 1 hour, which is why you need to get them at runtime and not set them at registration time.

## See also

- [Build API clients for .NET with Microsoft Identity authentication](/openapi/kiota/tutorials/dotnet-azure?wt.mc_id=SEC-MVP-5004985) shows how to use Kiota with Microsoft Identity authentication.
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
