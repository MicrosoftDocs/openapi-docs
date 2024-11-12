---
title: Build API clients for .NET
description: Learn how use Kiota to build API clients in .NET.
author: jasonjoh
ms.author: jasonjoh
ms.topic: quickstart
ms.date: 03/20/2023
---

# Build API clients for .NET

In this tutorial, you build a sample app in .NET that calls a REST API that doesn't require authentication.

## Required tools

- [.NET SDK 8.0](https://get.dot.net/8)

## Create a project

Run the following command in the directory you want to create a new project.

```bash
dotnet new console -o KiotaPosts
dotnet new gitignore
```

## Project configuration

In case you're adding a Kiota client to an existing project, the following configuration is required:

- ***.csproj** > **TargetFramework** set to "netstandard2.0", "netstandard2.1", "net462", or "net6" or later. [More information](/dotnet/standard/frameworks)
- ***.csproj** > **LangVersion** set to "preview", "latest", or "7.3" or later. [More information](/dotnet/csharp/language-reference/configure-language-version#c-language-version-reference)

The following configuration is recommended:

- ***.csproj** > **Nullable** set to "enable".

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [bundle package](https://github.com/microsoft/kiota-dotnet). For more information about Kiota dependencies, see [the dependencies documentation](../dependencies.md).

Run the following commands to get the required dependencies.

```bash
dotnet add package Microsoft.Kiota.Bundle
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **posts-api.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/quickstart/posts-api.yml":::

This file is a minimal OpenAPI description that describes how to call the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/).

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l CSharp -c PostsClient -n KiotaPosts.Client -d ./posts-api.yml -o ./Client
```

> [!TIP]
> Add [`--exclude-backward-compatible`](../using.md#--exclude-backward-compatible---ebc)
> if you want to reduce the size of the generated client and are not concerned about
> potentially source breaking changes with future versions of Kiota when updating the client.

## Create the client application

The final step is to update the **Program.cs** file that was generated as part of the console application to include the following code.

:::code language="csharp" source="~/code-snippets/get-started/quickstart/dotnet/src/Program.cs" id="ProgramSnippet":::

> [!NOTE]
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.

## Run the application

To start the application, run the following command in your project directory.

```bash
dotnet run
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstart/dotnet) contains the code from this guide.
- [Microsoft Graph sample using Microsoft identity platform authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/dotnet)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
