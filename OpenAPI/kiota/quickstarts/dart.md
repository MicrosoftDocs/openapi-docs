---
title: Build API clients for Dart
description: Learn how use Kiota to build API clients in Dart.
author: baywet
ms.author: vibiret
ms.topic: quickstart
ms.date: 01/17/2025
---

# Build API clients for Dart

In this tutorial, you build a sample app in Dart that calls a REST API that doesn't require authentication.

## Required tools

- [Dart SDK 3.6 or above](https://dart.dev/get-dart)

## Create a project

Run the following commands in the directory where you want to create a new project.

```bash
dart create -t console cli
```

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [bundle package](https://github.com/microsoft/kiota-dart). For more information about Kiota dependencies, see [the dependencies documentation](../dependencies.md).

Run the following commands to get the required dependencies.

```bash
dart pub add microsoft_kiota_bundle
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **posts-api.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/quickstart/posts-api.yml":::

This file is a minimal OpenAPI description that describes how to call the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/).

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l dart -d posts-api.yml -c PostsClient -o ./client
```

> [!TIP]
> Add [`--exclude-backward-compatible`](../using.md#--exclude-backward-compatible---ebc)
> if you want to reduce the size of the generated client and are not concerned about
> potentially source breaking changes with future versions of Kiota when updating the client.

## Create the client application

Create a file in the root of the project named **index.ts** and add the following code.

:::code language="dart" source="~/code-snippets/get-started/quickstart/dart/bin/cli.dart" id="ProgramSnippet":::

> [!NOTE]
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.

## Run the application

To start the application, run the following command in your project directory.

```bash
dart run
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstart/dart) contains the code from this guide.
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
