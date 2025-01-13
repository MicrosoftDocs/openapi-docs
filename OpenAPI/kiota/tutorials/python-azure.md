---
title: Build API clients for Python with Microsoft identity authentication
description: Learn how use Kiota to build API clients in Python to access APIs that require Microsoft identity authentication.
author: isvargasmsft
ms.author: isvargas
ms.topic: tutorial
ms.date: 03/20/2023
---

# Build API clients for Python with Microsoft identity authentication

In this tutorial, you generate an API client that uses [Microsoft identity authentication](/azure/active-directory/fundamentals/auth-oauth2) to access [Microsoft Graph](/graph/overview).

## Required tools

- [Python 3.9+](https://www.python.org/)
- [pip 20.0+](https://pip.pypa.io/en/stable/)
- [asyncio](https://docs.python.org/3/library/asyncio.html) or any other supported async environment, for example, AnyIO, Trio.

## Create a project

Create a directory to contain the new project.

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-abstractions-python). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of the following packages.

- Authentication ([Kiota default Azure authentication](https://github.com/microsoft/kiota-authentication-azure-python))
- HTTP ([Kiota default HTTPX-based implementation](https://github.com/microsoft/kiota-http-python))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-json-python))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-text-python))

For this tutorial, use the default implementations.

Run the following commands to get the required dependencies.

```bash
pip install microsoft-kiota-abstractions
pip install microsoft-kiota-authentication-azure
pip install microsoft-kiota-http
pip install microsoft-kiota-serialization-json
pip install microsoft-kiota-serialization-text
pip install azure-identity
```

> [!TIP]
> It is recommended to use a package manager/virtual environment to avoid installing packages system wide. Read more [here](https://packaging.python.org/en/latest/).

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **get-me.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/azure-auth/get-me.yml":::

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l python -d get-me.yml -c GetUserApiClient -n client -o ./client
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

Create a file in the root of the project named **get_user.py** and add the following code. Replace `YOUR_CLIENT_ID` with the client ID from your app registration.

:::code language="python" source="~/code-snippets/get-started/azure-auth/python/get_user.py" id="ProgramSnippet":::

> [!NOTE]
> This example uses the **DeviceCodeCredential** class. You can use any of the credential classes from the `azure.identity` library.

## Run the application

To start the application, run the following command in your project directory.

```bash
python3 get_user.py
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/python) contains the code from this guide.
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
