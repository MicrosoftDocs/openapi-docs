---
title: Build API clients for TypeScript with Microsoft identity authentication
description: Learn how use Kiota to build API clients in TypeScript to access APIs that require Microsoft identity authentication.
author: jasonjoh
ms.author: jasonjoh
ms.topic: tutorial
date: 03/20/2023
---

# Build API clients for TypeScript with Microsoft identity authentication

## Required tools

- [NodeJS 18](https://nodejs.org/en/)
- [TypeScript](https://www.typescriptlang.org/)

## Create a project

Run the following commands in the directory where you want to create a new project.

```bash
npm init
npm install -D typescript ts-node
npx tsc --init
```

## Add dependencies

Before you can compile and run the generated API client, you will need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://www.npmjs.com/package/@microsoft/kiota-abstractions). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of of the following packages.

- Authentication ([Kiota default Azure authentication](https://www.npmjs.com/package/@microsoft/kiota-authentication-azure))
- HTTP ([Kiota default fetch-based implementation](https://www.npmjs.com/package/@microsoft/kiota-http-fetchlibrary))
- Form serialization ([Kiota default](https://www.npmjs.com/package/@microsoft/kiota-serialization-form))
- JSON serialization ([Kiota default](https://www.npmjs.com/package/@microsoft/kiota-serialization-json))
- Text serialization ([Kiota default](https://www.npmjs.com/package/@microsoft/kiota-serialization-text))
- Multipart serialization ([Kiota default](https://www.npmjs.com/package/@microsoft/kiota-serialization-multipart))

For this tutorial, you will use the default implementations.

Run the following commands to get the required dependencies.

```bash
npm install @microsoft/kiota-abstractions
npm install @microsoft/kiota-authentication-azure
npm install @microsoft/kiota-http-fetchlibrary
npm install @microsoft/kiota-serialization-form
npm install @microsoft/kiota-serialization-json
npm install @microsoft/kiota-serialization-text
npm install @microsoft/kiota-serialization-multipart
npm install @azure/identity
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **get-me.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/azure-auth/get-me.yml":::

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l typescript -d get-me.yml -c GetUserApiClient -o ./client
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

Create a file in the root of the project named **index.ts** and add the following code. Replace `YOUR_CLIENT_ID` with the client ID from your app registration.

:::code language="typescript" source="~/code-snippets/get-started/azure-auth/typescript/index.ts" id="ProgramSnippet":::

> [!NOTE]
> This example uses the **DeviceCodeCredential** class. You can use any of the credential classes from the `@azure/identity` library.

## Run the application

Run the following command in your project directory to start the application.

```bash
npx ts-node index.ts
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/typescript) contains the code from this guide.
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
