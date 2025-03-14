---
title: Build API clients for Go
description: Learn how use Kiota to build API clients in Go.
author: jasonjoh
ms.author: jasonjoh
ms.topic: quickstart
ms.date: 03/24/2023
---

# Build API clients for Go

In this tutorial, you build a sample app in Go that calls a REST API that doesn't require authentication.

## Required tools

- [Go 1.22](https://golang.org/dl/)

## Create a project

Run the following command in the directory you want to create a new project.

```bash
go mod init kiota_posts
```

## Project configuration

In case you're adding a Kiota client to an existing project, the following configuration is required:

- **go.mod** > go set to version 18 or above.

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [bundle package](https://github.com/microsoft/kiota-bundle-go). For more information about Kiota dependencies, see [the dependencies documentation](../dependencies.md).

Run the following commands to get the required dependencies.

```bash
go get github.com/microsoft/kiota-bundle-go
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **posts-api.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/quickstart/posts-api.yml":::

This file is a minimal OpenAPI description that describes how to call the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/).

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l go -c PostsClient -n kiota_posts/client -d ./posts-api.yml -o ./client
```

> [!TIP]
> Add [`--exclude-backward-compatible`](../using.md#--exclude-backward-compatible---ebc)
> if you want to reduce the size of the generated client and are not concerned about
> potentially source breaking changes with future versions of Kiota when updating the client.

## Create the client application

Create a file in the root of the project named **main.go** and add the following code.

:::code language="go" source="~/code-snippets/get-started/quickstart/go/main.go" id="ProgramSnippet":::

> [!NOTE]
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.

## Run the application

To start the application, run the following command in your project directory.

```bash
go run .
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstart/go) contains the code from this guide.
- [Microsoft Graph sample using Microsoft identity platform authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/go)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
