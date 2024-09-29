---
title: Use Kiota with dependency injection in .NET
description: Learn how you can combine Kiota with dependency injection in .NET to build API clients.
author: svrooij
ms.author: vibiret
ms.topic: tutorial
date: 05/10/2024
---

# Register a Kiota API client in .NET with dependency injection

In this tutorial, you learn how to register a [Kiota API client](/openapi/kiota/tutorials/dotnet-azure) with [dependency injection](/dotnet/core/extensions/dependency-injection-usage?wt.mc_id=SEC-MVP-5004985).

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

Before you can compile and run the generated API client, ensure the generated source files are part of a project with the required dependencies. Your project must reference the [bundle package](https://github.com/microsoft/kiota-dotnet). For more information about kiota dependencies, refer to [the dependencies documentation](../dependencies.md).

Run the following commands to get the required dependencies.

```dotnetcli
dotnet add package Microsoft.Extensions.Http
dotnet add package Microsoft.Kiota.Bundle
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Let's download the GitHub API specs and generate the API client.

You can then use the [Kiota](/openapi/kiota/install?tabs=bash&wt.mc_id=SEC-MVP-5004985#install-as-net-tool) command line tool to generate the API client classes.

```bash
# Download the specs to ./github-api.json
kiota download apisguru::github.com -o ./github-api.json
# Generate the client, path filter for just releases
kiota generate -l csharp -d github-api.json -c GitHubClient -n GitHub.ApiClient -o ./GitHub --include-path "/repos/{owner}/{repo}/releases/*" --clean-output
```

## Create extension methods

Create a new file named _KiotaServiceCollectionExtensions.cs_ in the root of your project and add the following code.

:::code language="csharp" source="~/code-snippets/get-started/dotnet-dependency-injection/KiotaServiceCollectionExtensions.cs":::

## Create a client factory

Create a new file named _GitHubClientFactory.cs_ in the _GitHub_ folder and add the following code.

The code in these files implement a factory pattern, where you create a factory class that is responsible for creating an instance of the client. This way you have a single place that is responsible for creating the client, and you can easily change the way the client is created. In this sample we are registering the factory in the dependency injection container as well, because it gets the `HttpClient` injected in the constructor.

:::code language="csharp" source="~/code-snippets/get-started/dotnet-dependency-injection/GitHub/GitHubClientFactory.cs":::

## Register the API client

Open the _Program.cs_ file and add the following code above the `var app = builder.Build();` line. Sample [Program.cs](https://github.com/microsoft/kiota-samples/blob/main/get-started/dotnet-dependency-injection/Program.cs).

:::code language="csharp" source="~/code-snippets/get-started/dotnet-dependency-injection/Program.cs" highlight="9-23":::

## Create an endpoint

We will now create an endpoint that uses the GitHub client from dependency injection to get the latest release of `dotnet/runtime`.

Open the _Program.cs_ file and add the following code above the `app.Run();` line. Sample [Program.cs](https://github.com/microsoft/kiota-samples/blob/main/get-started/dotnet-dependency-injection/Program.cs).

:::code language="csharp" source="~/code-snippets/get-started/dotnet-dependency-injection/Program.cs" highlight="56-66":::

## Run the application

To start the application run the following command in your project directory.

```dotnetcli
dotnet run
```

You can now open a browser and navigate to `http://localhost:{port}/dotnet/releases` to see the latest release of `dotnet/runtime`. Alternatively, you can also go to `http://localhost:{port}/swagger` to see the Swagger UI with this new endpoint.

## Authentication

The `GitHubClientFactory` class is responsible for creating the client, and it is also responsible for creating the `IAuthenticationProvider`. In this sample, we are using the `AnonymousAuthenticationProvider`, which is a simple provider that does not add any authentication headers. More details on authentication with Kiota can be found in the [Kiota authentication documentation](/openapi/kiota/authentication).

The `GitHubClientFactory` is used through dependency injection, which means you can have other types injected by the constructor that are needed for the authentication provider.

You might want to create a [BaseBearerTokenAuthenticationProvider](/openapi/kiota/authentication?tabs=csharp&wt.mc_id=SEC-MVP-5004985#base-bearer-token-authentication-provider) that is available in Kiota and expects an `IAccessTokenProvider`, which is just an interface with one method to provide an access token. Access tokens usually have a lifetime of 1 hour, which is why you need to get them at runtime and not set them at registration time.

## See also

- [Sample project used in this guide](https://github.com/microsoft/kiota-samples/blob/main/get-started/dotnet-dependency-injection?wt.mc_id=SEC-MVP-5004985)
- [Build API clients for .NET with Microsoft Identity authentication](/openapi/kiota/tutorials/dotnet-azure) shows how to use Kiota with Microsoft Identity authentication.
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
