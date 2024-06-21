---
title: Build API clients for Go with Microsoft identity authentication
description: Learn how use Kiota to build API clients in Go to access APIs that require Microsoft identity authentication.
author: jasonjoh
ms.author: jasonjoh
ms.topic: tutorial
date: 03/20/2023
---

# Build API clients for Go with Azure authentication

In this tutorial, you generate an API client that uses [Microsoft identity authentication](/azure/active-directory/fundamentals/auth-oauth2) to access [Microsoft Graph](/graph/overview).

## Required tools

- [Go 1.20](https://golang.org/dl/)

## Create a project

Run the following commands in the directory where you want to create a new project.

```bash
go mod init getuser
```

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-abstractions-go). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of the following packages.

- Authentication ([Kiota default Azure authentication](https://github.com/microsoft/kiota-authentication-azure-go))
- HTTP ([Kiota default net/http-based implementation](https://github.com/microsoft/kiota-http-go))
- Form serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-form-go))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-json-go))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-text-go))
- Multipart serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-multipart-go))

For this tutorial, use the default implementations.

Run the following commands to get the required dependencies.

```bash
go get github.com/microsoft/kiota-abstractions-go
go get github.com/microsoft/kiota-http-go
go get github.com/microsoft/kiota-serialization-form-go
go get github.com/microsoft/kiota-serialization-json-go
go get github.com/microsoft/kiota-serialization-text-go
go get github.com/microsoft/kiota-serialization-multipart-go
go get github.com/microsoft/kiota-authentication-azure-go
go get github.com/Azure/azure-sdk-for-go/sdk/azidentity
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **get-me.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/azure-auth/get-me.yml":::

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l go -d ../get-me.yml -c GraphApiClient -n getuser/client -o ./client
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

Create a file in the root of the project named **getuser.go** and add the following code. Replace `YOUR_CLIENT_ID` with the client ID from your app registration.

:::code language="go" source="~/code-snippets/get-started/azure-auth/go/getuser.go" id="ProgramSnippet":::

> [!NOTE]
> This example uses the **DeviceCodeCredential** class. You can use any of the credential classes from the `azidentity` library.

## Run the application

To start the application, run the following command in your project directory.

```bash
go run .
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/go) contains the code from this guide.
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
