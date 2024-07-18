---
title: Build API clients for Java
description: Learn how use Kiota to build API clients in Java.
author: jasonjoh
ms.author: jasonjoh
ms.topic: quickstart
date: 03/24/2023
---

# Build API clients for Java

In this tutorial, you build a sample app in Java that calls a REST API that doesn't require authentication.

## Required tools

- [OpenJDK 17](/java/openjdk/download)
- [Gradle 7.4](https://gradle.org/install/)

## Create a project

Use Gradle to initialize a Java application project.

```bash
gradle init --dsl groovy --test-framework junit --type java-application --project-name kiotaposts --package kiotaposts
```

## Project configuration

In case you're adding a Kiota client to an existing project, the following configuration is required:

- **build.gradle** > **compileJava** > **sourceCompatibility** set to "1.8" or later. (or equivalent if you're not using gradle)
- **build.gradle** > **compileJava** > **targetCompatibility** set to "1.8" or later. (or equivalent if you're not using gradle)
- Java Development Kit (JDK) version 17 or above.

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-java). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of the following packages.

- HTTP ([Kiota default OkHttp-based implementation](https://github.com/microsoft/kiota-java))
- Form serialization ([Kiota default](https://github.com/microsoft/kiota-java))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-java))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-java))
- Multipart serialization ([Kiota default](https://github.com/microsoft/kiota-java))
- Jakarta annotations [Maven Central](https://central.sonatype.com/artifact/jakarta.annotation/jakarta.annotation-api)

For this tutorial, use the default implementations.

Edit **./app/build.gradle** to add the following dependencies.

> [!NOTE]
> Find current version numbers for Kiota packages at [Nexus Repository Manager](https://oss.sonatype.org/).

:::code language="gradle" source="~/code-snippets/get-started/quickstart/java/app/build.gradle" id="DependenciesSnippet":::

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **posts-api.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/quickstart/posts-api.yml":::

This file is a minimal OpenAPI description that describes how to call the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/).

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l java -c PostsClient -n kiotaposts.client -d ./posts-api.yml -o ./app/src/main/java/kiotaposts/client
```

> [!TIP]
> Add [`--exclude-backward-compatible`](../using.md#--exclude-backward-compatible---ebc)
> if you want to reduce the size of the generated client and are not concerned about
> potentially source breaking changes with future versions of Kiota when updating the client.

## Create the client application

The final step is to update the **./app/src/main/java/kiotaposts/App.java** file that was generated as part of the console application, replacing its contents with the following code.

:::code language="java" source="~/code-snippets/get-started/quickstart/java/app/src/main/java/kiotaposts/App.java" id="ProgramSnippet":::

> [!NOTE]
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.

## Run the application

To start the application, run the following command in your project directory.

```bash
./gradlew --console plain run
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstart/java) contains the code from this guide.
- [Microsoft Graph sample using Microsoft identity platform authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/java)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.

## (UNOFFICIAL) Community tools and libraries

> [!IMPORTANT]
> The Kiota Community tools and libraries are maintained and distributed by the community and are not official Microsoft tools and libraries. Microsoft makes no warranties, express or implied, with respect to the tools and libraries or their use. Use of the tools and libraries at your own risk. Microsoft shall not be liable for any damages arising out of or in connection with the use of the tools and libraries.

There are community tools and libraries to make it easy for you to get started with Kiota and to better harmonize the class path according to the rest of your stack:

- [Kiota Maven Plugin](https://github.com/kiota-community/kiota-java-extra?tab=readme-ov-file#maven-plugin)
- [Kiota Jackson Serialization](https://github.com/kiota-community/kiota-java-extra?tab=readme-ov-file#serialization-jackson)
- [Kiota Http VertX](https://github.com/kiota-community/kiota-java-extra?tab=readme-ov-file#http-vertx)
- [Kiota Http JDK](https://github.com/kiota-community/kiota-java-extra?tab=readme-ov-file#http-jdk)
- [Kiota Quarkus Extension](https://github.com/quarkiverse/quarkus-kiota)
