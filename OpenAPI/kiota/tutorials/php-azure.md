---
title: Build API clients for PHP with Microsoft identity authentication
description: Learn how use Kiota to build API clients in PHP to access APIs that require Microsoft identity authentication.
author: Ndiritu
ms.author: pgichuhi
ms.topic: tutorial
date: 03/20/2023
---

# Build API clients for PHP with Microsoft identity authentication

In this tutorial, you will generate an API client that uses [Microsoft identity authentication](/azure/active-directory/fundamentals/auth-oauth2) to access [Microsoft Graph](/graph/overview).

## Required tools

- [PHP ^7.4 or ^8.0](https://www.php.net/downloads)
- [Composer](https://getcomposer.org/)

## Create a project

Run the following commands in the directory where you want to create a new project.

```bash
composer init
```

## Add dependencies

Before you can compile and run the generated API client, you will need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-abstractions-php). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of of the following packages.

- Authentication ([Kiota default Azure authentication](https://github.com/microsoft/kiota-authentication-phpleague-php))
- HTTP ([Kiota default Guzzle-based implementation](https://github.com/microsoft/kiota-http-guzzle-php))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-json-php))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-text-php))

For this tutorial, you will use the default implementations.

Run the following commands to get the required dependencies.

```bash
composer require microsoft/kiota-abstractions
composer require microsoft/kiota-http-guzzle
composer require microsoft/kiota-authentication-phpleague
composer require microsoft/kiota-serialization-json
composer require microsoft/kiota-serialization-text
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **get-me.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/azure-auth/get-me.yml":::

You can then use the Kiota command line tool to generate the API client classes.

```shell
kiota generate -l PHP -d get-me.yml -c GraphApiClient -n GetUser\Client -o ./client
```

Add the following to your `composer.json` to set your namespaces correctly:

```json
"autoload": {
    "psr-4": {
        "GetUser\\Client\\": "client/"
    }
}
```

To ensure the newly generated classes can be imported, update the autoload paths using:

```shell
composer dumpautoload
```

## Register an application

[!INCLUDE [register-azure-app-auth-code-intro](../includes/register-azure-app-auth-code-intro.md)]

<!-- markdownlint-disable MD051 -->
### [Azure portal](#tab/portal)

[!INCLUDE [register-azure-app-auth-code-portal](../includes/register-azure-app-auth-code-portal.md)]

### [PowerShell](#tab/powershell)

[!INCLUDE [register-azure-app-auth-code-powershell](../includes/register-azure-app-auth-code-powershell.md)]
<!-- markdownlint-enable MD051 -->

---

## Create the client application

Create a file in the root of the project named **GetUser.php** and add the following code. Replace the `$clientId` and `$clientSecret` with your credentials from the previous step.

[!INCLUDE [get-azure-auth-code-manually](../includes/get-azure-auth-code-manually.md)]

Replace the `$authorizationCode` with your authorization code.

:::code language="php" source="~/code-snippets/get-started/azure-auth/php/GetUser.php" id="ProgramSnippet":::

> [!NOTE]
> This example uses the **AuthorizationCodeContext** class. You can use any of the credential classes from the `kiota-authentication-phpleague` library.

## Run the application

Run the following command in your project directory to start the application.

```shell
php GetUser.php
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/php) contains the code from this guide.
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
