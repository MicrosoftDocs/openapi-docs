---
title: Build API clients for TypeScript
description: Learn how use Kiota to build API clients in TypeScript.
author: jasonjoh
ms.author: jasonjoh
ms.topic: quickstart
ms.date: 03/24/2023
---

# Build API clients for TypeScript

In this tutorial, you build a sample app in TypeScript that calls a REST API that doesn't require authentication.

## Required tools

- [NodeJS 18 or above](https://nodejs.org/en/)
- [TypeScript 5 or above](https://www.typescriptlang.org/)

## Create a project

Run the following commands in the directory where you want to create a new project.

```bash
npm init
npm install -D typescript @types/node
npx tsc --init
```

## Project configuration

In case you're adding a Kiota client to an existing project, the following configuration is required:

- **tsconfig** > **compilerOptions** > **esModuleInterop** set to "true".
- **tsconfig** > **compilerOptions** > **forceConsistentCasingInFileNames** set to "true".
- **tsconfig** > **compilerOptions** > **lib** with an entry of "es2015".
- **tsconfig** > **compilerOptions** > **module** set to "NodeNext".
- **tsconfig** > **compilerOptions** > **moduleResolution** set to "NodeNext".
- **tsconfig** > **compilerOptions** > **target** set to "es2020" or later.
- **package.json** > **type** set to "module".

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [bundle package](https://github.com/microsoft/kiota-typescript). For more information about Kiota dependencies, see [the dependencies documentation](../dependencies.md).

Run the following commands to get the required dependencies.

```bash
npm install @microsoft/kiota-bundle
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **posts-api.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/quickstart/posts-api.yml":::

This file is a minimal OpenAPI description that describes how to call the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/).

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l typescript -d posts-api.yml -c PostsClient -o ./client
```

> [!TIP]
> Add [`--exclude-backward-compatible`](../using.md#--exclude-backward-compatible---ebc)
> if you want to reduce the size of the generated client and are not concerned about
> potentially source breaking changes with future versions of Kiota when updating the client.

## Create the client application

Create a file in the root of the project named **index.ts** and add the following code.

:::code language="typescript" source="~/code-snippets/get-started/quickstart/typescript/index.ts" id="ProgramSnippet":::

> [!NOTE]
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.

## Run the application

To start the application, run the following command in your project directory.

```bash
tsc && node out/index.js
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstart/typescript) contains the code from this guide.
- [Microsoft Graph sample using Microsoft identity platform authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/typescript)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
