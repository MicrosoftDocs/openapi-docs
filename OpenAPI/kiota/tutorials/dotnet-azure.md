---
title: Build API clients for .NET with Microsoft identity authentication
description: Learn how use Kiota to build API clients in .NET to access APIs that require Microsoft identity authentication.
author: jasonjoh
ms.author: jasonjoh
ms.topic: tutorial
date: 11/29/2023
---

# Build API clients for .NET with Microsoft identity authentication

In this tutorial, you generate an API client that uses [Microsoft identity authentication](/azure/active-directory/fundamentals/auth-oauth2) to access [Microsoft Graph](/graph/overview).

## Required tools

- [.NET SDK 8.0](https://get.dot.net/8)

## Create a project

Run the following command in the directory you want to create a new project.

```dotnetcli
dotnet new console -o GetUserClient
dotnet new gitignore
```

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-abstractions-dotnet). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of the following packages.

- Authentication ([Kiota default Azure authentication](https://github.com/microsoft/kiota-authentication-azure-dotnet))
- HTTP ([Kiota default HttpClient-based implementation](https://github.com/microsoft/kiota-http-dotnet))
- Form serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-form-dotnet))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-json-dotnet))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-text-dotnet))
- Multipart serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-multipart-dotnet))

For this tutorial, use the default implementations.

Run the following commands to get the required dependencies.

```dotnetcli
dotnet add package Microsoft.Kiota.Abstractions
dotnet add package Microsoft.Kiota.Http.HttpClientLibrary
dotnet add package Microsoft.Kiota.Serialization.Form
dotnet add package Microsoft.Kiota.Serialization.Json
dotnet add package Microsoft.Kiota.Serialization.Text
dotnet add package Microsoft.Kiota.Serialization.Multipart
dotnet add package Microsoft.Kiota.Authentication.Azure
dotnet add package Azure.Identity
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **get-me.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/azure-auth/get-me.yml":::

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l csharp -d get-me.yml -c GetUserApiClient -n GetUserClient.ApiClient -o ./Client
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

The final step is to update the _Program.cs_ file that was generated as part of the console application to include the following code. Replace `YOUR_CLIENT_ID` with the client ID from your app registration.

:::code language="csharp" source="~/code-snippets/get-started/azure-auth/dotnet/src/Program.cs" id="ProgramSnippet":::

> [!NOTE]
> This example uses the `DeviceCodeCredential` class. You can use any of the credential classes from the `Azure.Identity` library.

## Run the application

To start the application, run the following command in your project directory.

```dotnetcli
dotnet run
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/dotnet) contains the code from this guide.
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
