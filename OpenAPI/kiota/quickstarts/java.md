---
title: Build API clients for Java
description: Learn how use Kiota to build API clients in Java.
author: jasonjoh
ms.author: jasonjoh
ms.topic: quickstart
date: 03/24/2023
---

# Build API clients for Java

In this tutorial, you will build a sample app in Java that calls a REST API that does not require authentication.

## Required tools

- [OpenJDK 17](https://learn.microsoft.com/java/openjdk/download)
- [Gradle 7.4](https://gradle.org/install/)

## Create a project

Use Gradle to initialize a Java application project.

```bash
gradle init --dsl groovy --test-framework junit --type java-application --project-name kiotaposts --package kiotaposts
```

## Add dependencies

Before you can compile and run the generated API client, you will need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-java). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of of the following packages.

- HTTP ([Kiota default OkHttp-based implementation](https://github.com/microsoft/kiota-java))
- Form serialization ([Kiota default](https://github.com/microsoft/kiota-java))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-java))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-java))
- Multipart serialization ([Kiota default](https://github.com/microsoft/kiota-java))

For this tutorial, you will use the default implementations.

Edit **./app/build.gradle** to add the following dependencies.

> [!NOTE]
> Find current version numbers for Kiota packages at [Nexus Repository Manager](https://oss.sonatype.org/).

:::code language="gradle" source="~/code-snippets/get-started/quickstart/java/app/build.gradle" id="DependenciesSnippet":::

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **posts-api.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/quickstart/posts-api.yml":::

This is a minimal OpenAPI description that describes how to call the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/).

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l java -c PostsClient -n kiotaposts.client -d ./posts-api.yml -o ./app/src/main/java/kiotaposts/client
```

## Create the client application

The final step is to update the **./app/src/main/java/kiotaposts/App.java** file that was generated as part of the console application, replacing its contents with the code below.

:::code language="java" source="~/code-snippets/get-started/quickstart/java/app/src/main/java/kiotaposts/App.java" id="ProgramSnippet":::

> [!NOTE]
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.

## Run the application

Run the following command in your project directory to start the application.

```bash
./gradlew --console plain run
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstart/go) contains the code from this guide.
- [Microsoft Graph sample using Microsoft identity platform authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/go)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
