---
title: Build API clients for a command line interface (CLI) with Microsoft identity authentication
description: Learn how use Kiota to build API clients for a CLI to access APIs that require Microsoft identity authentication.
author: jasonjoh
ms.author: jasonjoh
ms.topic: tutorial
date: 03/20/2023
---

# Build API clients for a command line interface (CLI) with Microsoft identity authentication

In this tutorial, you generate an API client that uses [Microsoft identity authentication](/azure/active-directory/fundamentals/auth-oauth2) to access [Microsoft Graph](/graph/overview).

## Required tools

A command line tool is required. We recommend:

- [Windows Terminal](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=en-us&gl=us) (or equivalent on MacOS/linux)
- [.NET SDK 8.0](https://get.dot.net/8)

## Create a project

Run the following command in the directory you want to create a new project.

```bash
dotnet new console -o GetUserClient
dotnet new gitignore
```

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [bundle package](https://github.com/microsoft/kiota-dotnet) and the [cli-commons package](https://github.com/microsoft/kiota-cli-commons). For more information about Kiota dependencies, see [the dependencies documentation](../dependencies.md).

Run the following commands to get the required dependencies.

```bash
dotnet add package Microsoft.Kiota.Bundle
dotnet add package Microsoft.Kiota.Cli.Commons --prerelease
dotnet add package Azure.Identity
dotnet add package Microsoft.Extensions.DependencyInjection
dotnet add package Microsoft.Extensions.Hosting
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **get-me.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/azure-auth/get-me.yml":::

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate --openapi get-me.yml --language CLI -c GetUserApiClient -n GetUserClient.ApiClient -o ./Client
```

## Register an application

[!INCLUDE [register-azure-app-device-code-intro](../includes/register-azure-app-device-code-intro.md)]

<!-- markdownlint-disable MD051 -->
### [Azure portal](#tab/portal)

[!INCLUDE [register-azure-app-device-code-portal](../includes/register-azure-app-device-code-portal.md)]

### [PowerShell](#tab/powershell)

[!INCLUDE [register-azure-app-device-code-powershell](../includes/register-azure-app-device-code-powershell.md)]
<!-- markdownlint-enable MD051 -->

---

## Create the client application

The final step is to update the **Program.cs** file that was generated as part of the console application to include the following code. Replace `YOUR_CLIENT_ID` with the client ID from your app registration.

:::code language="csharp" source="~/code-snippets/get-started/azure-auth/cli/src/Program.cs" id="ProgramSnippet":::

> [!NOTE]
> This example uses the **DeviceCodeCredential** class. You can use any of the credential classes from the `Azure.Identity` library.

## Run the application

To start the application, run the following command in your project directory.

```bash
dotnet run -- me get
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/cli) contains the code from this guide.
- [More CLI samples](https://github.com/microsoftgraph/msgraph-cli/tree/main/samples)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
