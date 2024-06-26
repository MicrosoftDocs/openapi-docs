---
title: Use Kiota with dependency injection in .NET
description: Learn how you can combine Kiota with dependency injection in .NET to build API clients.
author: svrooij
ms.author: vibiret
ms.topic: tutorial
date: 05/10/2024
---

# Register a Kiota API client in .NET with dependency injection

In this tutorial, you will learn how to register a [Kiota API client](/openapi/kiota/tutorials/dotnet-azure) with [dependency injection](/dotnet/core/extensions/dependency-injection-usage?wt.mc_id=SEC-MVP-5004985).

## Required tools

- [.NET SDK 8.0](https://get.dot.net/8)
- [Kiota CLI](/openapi/kiota/install?tabs=bash#install-as-net-tool)

## Create a project

Run the following command in the directory you want to create a new project.

```dotnetcli
dotnet new webapi -o kiota-with-di
cd kiota-with-di
dotnet new gitignore
```

## Add dependencies

Before you can compile and run the generated API client, ensure the generated source files are part of a project with the required dependencies. Your project must reference the [abstraction package](https://github.com/microsoft/kiota-abstractions-dotnet) and default implementations.

- HTTP ([Kiota default HttpClient-based implementation](https://github.com/microsoft/kiota-http-dotnet)) version `1.4.2` or higher
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

:::code language="yaml" source="~/code-snippets/get-started/dotnet-dependency-injection/github-releases.yml":::

You can then use the [Kiota](/openapi/kiota/install?tabs=bash&wt.mc_id=SEC-MVP-5004985#install-as-net-tool) command line tool to generate the API client classes.

```bash
kiota generate -l csharp -d github-releases.yml -c GitHubClient -n GitHub.ApiClient -o ./GitHub
```

## Create extension methods

Create a new file named _KiotaServiceCollectionExtensions.cs_ in the root of your project and add the following code.

:::code language="csharp" source="~/code-snippets/get-started/dotnet-dependency-injection/KiotaServiceCollectionExtensions.cs":::

## Create a client factory

Create a new file named _GitHubClientFactory.cs_ in the _GitHub_ folder and add the following code.

This is what you call a factory pattern, where you create a factory class that is responsible for creating an instance of the client. This way you have a single place that is responsible for creating the client, and you can easily change the way the client is created. In this sample we will be registering the factory in the dependency injection container as well, because it will get the `HttpClient` injected in the constructor.

:::code language="csharp" source="~/code-snippets/get-started/dotnet-dependency-injection/GitHub/GitHubClientFactory.cs":::

## Register the API client

Open the _Program.cs_ file and add the following code above the `var app = builder.Build();` line. Sample [Program.cs](https://github.com/microsoft/kiota-samples/blob/main/get-started/dotnet-dependency-injection/Program.cs).

```csharp
// Add Kiota handlers to the dependency injection container
builder.Services.AddKiotaHandlers();

// Register the factory for the GitHub client
builder.Services.AddHttpClient<GitHubClientFactory>((sp, client) => {
    // Set the base address and accept header
    // or other settings on the http client
    client.BaseAddress = new Uri("https://api.github.com");
    client.DefaultRequestHeaders.Add("Accept", "application/vnd.github.v3+json");
}).AttachKiotaHandlers(); // Attach the Kiota handlers to the http client, this is to enable all the Kiota features.

// Register the GitHub client
builder.Services.AddTransient(sp => sp.GetRequiredService<GitHubClientFactory>().GetClient());
```

## Create an endpoint

We will now create an endpoint that uses the GitHub client from dependency injection to get the latest release of `dotnet/runtime`.

Open the _Program.cs_ file and add the following code above the `app.Run();` line. Sample [Program.cs](https://github.com/microsoft/kiota-samples/blob/main/get-started/dotnet-dependency-injection/Program.cs).

```csharp
app.MapGet("/dotnet/releases", async (GitHub.ApiClient.GitHubClient client, CancellationToken cancellationToken) =>
{
    var releases = await client.Repos["dotnet"]["runtime"].Releases["latest"].GetAsync(cancellationToken: cancellationToken);
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

The `GitHubClientFactory` class is responsible for creating the client, and it is also responsible for creating the `IAuthenticationProvider`. In this sample, we are using the `AnonymousAuthenticationProvider`, which is a simple provider that does not add any authentication headers. More details on authentication with Kiota can be found in the [Kiota authentication documentation](/openapi/kiota/authentication).

The `GitHubClientFactory` is used through dependency injection, which means you can have other types injected by the constructor that are needed for the authentication provider.

You might want to create a [BaseBearerTokenAuthenticationProvider](/openapi/kiota/authentication?tabs=csharp&wt.mc_id=SEC-MVP-5004985#base-bearer-token-authentication-provider) that is available in Kiota and expects an `IAccessTokenProvider` which is just an interface with one method "give me an access token". Access tokens usually have a lifetime of 1 hour, which is why you need to get them at runtime and not set them at registration time.

## See also

- [Sample project used in this guide](https://github.com/microsoft/kiota-samples/blob/main/get-started/dotnet-dependency-injection?wt.mc_id=SEC-MVP-5004985)
- [Build API clients for .NET with Microsoft Identity authentication](/openapi/kiota/tutorials/dotnet-azure) shows how to use Kiota with Microsoft Identity authentication.
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
