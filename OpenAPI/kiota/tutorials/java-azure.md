---
title: Build API clients for Java with Microsoft identity authentication
description: Learn how use Kiota to build API clients in Java to access APIs that require Microsoft identity authentication.
author: jasonjoh
ms.author: jasonjoh
ms.topic: tutorial
date: 03/20/2023
---

# Build API clients for Java with Microsoft identity authentication

## Required tools

- [Java Development Kit (JDK) 16](https://adoptopenjdk.net/)
- [Gradle 7.4](https://gradle.org/install/)

## Create a project

Use Gradle to initialize a Java application project.

```bash
gradle init --dsl groovy --test-framework junit --type java-application --project-name getuserclient --package getuserclient
```

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [bundle package](https://github.com/microsoft/kiota-java).  For more information about kiota dependencies, refer to [the dependencies documentation](../dependencies.md).

For this tutorial, use the default implementations.

Edit **./app/build.gradle** to add the following dependencies.

> [!NOTE]
> Find current version numbers for Kiota packages at [Nexus Repository Manager](https://oss.sonatype.org/).

:::code language="gradle" source="~/code-snippets/get-started/azure-auth/java/app/build.gradle" id="DependenciesSnippet":::

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **get-me.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/azure-auth/get-me.yml":::

You can then use the Kiota command line tool to generate the SDK classes.

```bash
kiota generate -l java -d get-me.yml -c GetUserApiClient -n getuserclient.apiclient -o ./app/src/main/java/getuserclient/apiclient --ds none -s none
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

The final step is to update the **./app/src/main/java/getuserclient/App.java** file that was generated as part of the console application to include the following code. Replace `YOUR_CLIENT_ID` with the client ID from your app registration.

:::code language="java" source="~/code-snippets/get-started/azure-auth/java/app/src/main/java/getuserclient/App.java" id="ProgramSnippet":::

> [!NOTE]
> This example uses the **DeviceCodeCredential** class. You can use any of the credential classes from the `com.azure.identity` library.

## Run the application

To start the application, run the following command in your project directory.

```bash
./gradlew --console plain run
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/java) contains the code from this guide.
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
