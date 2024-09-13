---
title: Build API clients for a command line interface (CLI)
description: Learn how use Kiota to build API clients for a CLI.
author: jasonjoh
ms.author: jasonjoh
ms.topic: quickstart
date: 03/24/2023
---

# Build API clients for a command line interface (CLI)

In this tutorial, you build a sample command line interface (CLI) app that calls a REST API that doesn't require authentication.

## Required tools

A command line tool is required. We recommend:

- [Windows Terminal](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=en-us&gl=us) (or equivalent on MacOS/linux)
- [.NET SDK 7.0](https://get.dot.net/7)

## Create a project

Run the following command in the directory you want to create a new project.

```bash
dotnet new console -o KiotaPostsCLI
cd KiotaPostsCLI
dotnet new gitignore
```

## Project configuration

In case you're adding a Kiota client to an existing project, the following configuration is required:

- ***.csproj** > **TargetFramework** set to "net6" or later. [More information](/dotnet/standard/frameworks)
- ***.csproj** > **LangVersion** set to "latest". [More information](/dotnet/csharp/language-reference/configure-language-version#c-language-version-reference)
- ***.csproj** > **Nullable** set to "enable".

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [bundle package](https://github.com/microsoft/kiota-dotnet) and the [cli-commons package](https://github.com/microsoft/kiota-cli-commons). For more information about kiota dependencies, refer to [the dependencies documentation](../dependencies.md).

For this tutorial, use the default implementations.

Run the following commands to get the required dependencies.

```bash
dotnet add package Microsoft.Kiota.Bundle
dotnet add package Microsoft.Kiota.Cli.Commons --prerelease
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **posts-api.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/quickstart/posts-api.yml":::

This file is a minimal OpenAPI description that describes how to call the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/).

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l CLI -c PostsClient -n KiotaPostsCLI.Client -d ./posts-api.yml -o ./src/Client
```

> [!TIP]
> Add [`--exclude-backward-compatible`](../using.md#--exclude-backward-compatible---ebc)
> if you want to reduce the size of the generated client and are not concerned about
> potentially source breaking changes with future versions of Kiota when updating the client.

## Create the client application

The final step is to update the **Program.cs** file that was generated as part of the console application, replacing its contents with the following code.

:::code language="csharp" source="~/code-snippets/get-started/quickstart/cli/src/Program.cs" id="ProgramSnippet":::

> [!NOTE]
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.

## Run the application

To invoke the CLI app, run the following commands in your project directory.

### GET /posts

```bash
dotnet run -- posts list
```

### GET /posts/{id}

```bash
dotnet run -- posts get --post-id 5
```

### POST /posts

```bash
dotnet run -- posts create --body '{ "userId": 42, "title": "Testing Kiota-generated API client", "body": "Hello world!" }'
```

### PATCH /posts/{id}

```bash
dotnet run -- posts patch --post-id 5 --body '{ "title": "Updated title" }'
```

### DELETE /posts/{id}

```bash
dotnet run -- posts delete --post-id 5
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstart/cli) contains the code from this guide.
- [Microsoft Graph sample using Microsoft identity platform authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/cli)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
