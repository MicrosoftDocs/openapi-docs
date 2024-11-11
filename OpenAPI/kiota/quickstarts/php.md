---
title: Build API clients for PHP
description: Learn how use Kiota to build API clients in PHP.
author: jasonjoh
ms.author: jasonjoh
ms.topic: quickstart
ms.date: 03/24/2023
---

# Build API clients for PHP

In this tutorial, you build a sample app in PHP that calls a REST API that doesn't require authentication.

## Required tools

- [PHP ^7.4 or ^8.0](https://www.php.net/downloads)
- [Composer](https://getcomposer.org/)

## Create a project

Run the following commands in the directory where you want to create a new project.

```bash
composer init
```

## Project configuration

In case you're adding a Kiota client to an existing project, the following configuration is required:

- **composer.json** > **require** > **php** set to "php": "^8.0 || ^7.4" or later.

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-abstractions-php). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of the following packages.

- HTTP ([Kiota default Guzzle-based implementation](https://github.com/microsoft/kiota-http-guzzle-php))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-json-php))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-text-php))

For this tutorial, use the default implementations.

Run the following commands to get the required dependencies.

```bash
composer require microsoft/kiota-abstractions
composer require microsoft/kiota-http-guzzle
composer require microsoft/kiota-serialization-json
composer require microsoft/kiota-serialization-text
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **posts-api.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/quickstart/posts-api.yml":::

This file is a minimal OpenAPI description that describes how to call the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/).

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l PHP -d ../posts-api.yml -c PostsApiClient -n KiotaPosts\Client -o ./client
```

> [!TIP]
> Add [`--exclude-backward-compatible`](../using.md#--exclude-backward-compatible---ebc)
> if you want to reduce the size of the generated client and are not concerned about
> potentially source breaking changes with future versions of Kiota when updating the client.

Add the following to your `composer.json` to set your namespaces correctly:

```json
"autoload": {
    "psr-4": {
        "KiotaPosts\\Client\\": "client/"
    }
}
```

To ensure the newly generated classes can be imported, update the autoload paths using:

```bash
composer dumpautoload
```

## Create the client application

Create a file in the root of the project named **main.php** and add the following code.

:::code language="php" source="~/code-snippets/get-started/quickstart/php/main.php" id="ProgramSnippet":::

> [!NOTE]
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.

## Run the application

To start the application, run the following command in your project directory.

```bash
php main.php
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstart/php) contains the code from this guide.
- [Microsoft Graph sample using Microsoft identity platform authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/php)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
