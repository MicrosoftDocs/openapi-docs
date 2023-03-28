---
title: Build API clients for Python
description: Learn how use Kiota to build API clients in Python.
author: jasonjoh
ms.author: jasonjoh
ms.topic: quickstart
date: 03/24/2023
---

# Build API clients for Python

In this tutorial, you will build a sample app in Python that calls a REST API that does not require authentication.

## Required tools

- [Python 3.6+](https://www.python.org/)
- [pip 20.0+](https://pip.pypa.io/en/stable/)
- [asyncio/any other supported async environment e.g AnyIO, Trio.](https://docs.python.org/3/library/asyncio.html)

## Create a project

Create a directory that will contain the new project.

## Add dependencies

Before you can compile and run the generated API client, you will need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-abstractions-python). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of of the following packages.

- HTTP ([Kiota default HTTPX-based implementation](https://github.com/microsoft/kiota-http-python))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-json-python))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-text-python))

For this tutorial, you will use the default implementations.

Run the following commands to get the required dependencies.

```bash
pip install microsoft-kiota-abstractions
pip install microsoft-kiota-http
pip install microsoft-kiota-serialization-json
pip install microsoft-kiota-serialization-text
```

> [!TIP]
> It is recommended to use a package manager/virtual environment to avoid installing packages system wide. Read more [here](https://packaging.python.org/en/latest/).

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **posts-api.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/quickstart/posts-api.yml":::

This is a minimal OpenAPI description that describes how to call the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/).

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l python -c PostsClient -n client -d ./posts-api.yml -o ./client
```

## Create the client application

Create a file in the root of the project named **main.py** and add the following code.

:::code language="python" source="~/code-snippets/get-started/quickstart/python/main.py" id="ProgramSnippet":::

> [!NOTE]
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.

## Run the application

Run the following command in your project directory to start the application.

```bash
php main.php
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstart/php) contains the code from this guide.
- [Microsoft Graph sample using Microsoft identity platform authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/php)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
