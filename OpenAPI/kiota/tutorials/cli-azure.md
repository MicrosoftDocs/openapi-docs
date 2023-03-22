---
title: Build API clients for CLIs with Microsoft identity authentication
description: Learn how use Kiota to build API clients for CLIs to access APIs that require Microsoft identity authentication.
author: jasonjoh
ms.author: jasonjoh
ms.topic: tutorial
date: 03/20/2023
---

# Build API clients for CLIs with Microsoft identity authentication

In this tutorial, you will generate an API client that uses [Microsoft identity authentication](/azure/active-directory/fundamentals/auth-oauth2) to access [Microsoft Graph](/graph/overview).

## Required tools

A command line tool is required. We recommend:

- [Windows Terminal](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=en-us&gl=us) (or equivalent on MacOS/linux)
- [.NET SDK 7.0](https://get.dot.net/7)

## Create a project

Run the following command in the directory you want to create a new project.

```bash
dotnet new console -o GetUserClient
dotnet new gitignore
```

## Add dependencies

Before you can compile and run the generated API client, you will need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-abstractions-dotnet) and the [cli-commons package](https://github.com/microsoft/kiota-cli-commons). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of of the following packages.

- Authentication ([Kiota default Azure authentication](https://github.com/microsoft/kiota-authentication-azure-dotnet))
- HTTP ([Kiota default HttpClient-based implementation](https://github.com/microsoft/kiota-http-dotnet))
- Form serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-form-dotnet))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-json-dotnet))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-text-dotnet))

For this tutorial, you will use the default implementations.

Run the following commands to get the required dependencies.

```bash
dotnet add package Microsoft.Kiota.Abstractions
dotnet add package Microsoft.Kiota.Cli.Commons --prerelease
dotnet add package Microsoft.Kiota.Http.HttpClientLibrary
dotnet add package Microsoft.Kiota.Serialization.Form
dotnet add package Microsoft.Kiota.Serialization.Json
dotnet add package Microsoft.Kiota.Serialization.Text
dotnet add package Microsoft.Kiota.Authentication.Azure
dotnet add package Azure.Identity
dotnet add package Microsoft.Extensions.DependencyInjection
dotnet add package Microsoft.Extensions.Hosting
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **get-me.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/getme.yml":::

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate --openapi get-me.yml --language shell -c GetUserApiClient -n GetUserClient.ApiClient -o ./Client
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

The final step is to update the **Program.cs** file that was generated as part of the console application to include the code below. Replace `YOUR_CLIENT_ID` with the client ID from your app registration.

```csharp
using System.CommandLine.Builder;
using System.CommandLine.Parsing;
using Azure.Identity;
using GetUserClient.ApiClient;
using Microsoft.Kiota.Authentication.Azure;
using Microsoft.Kiota.Cli.Commons.Extensions;
using Microsoft.Kiota.Http.HttpClientLibrary;

var rootCommand = new GetUserApiClient().BuildRootCommand();
rootCommand.Description = "CLI description";

// Set up services
var builder = new CommandLineBuilder(rootCommand)
    .UseDefaults()
    .UseRequestAdapter(ic =>
    {
        // The auth provider will only authorize requests to
        // the allowed hosts, in this case Microsoft Graph
        var allowedHosts = new [] { "graph.microsoft.com" };
        var graphScopes = new [] { "User.Read" };
        var options = new DeviceCodeCredentialOptions
        {
            ClientId = "15efe9a3-d292-48e9-afa0-6747c61b925c",
            DeviceCodeCallback = (code, cancellation) =>
            {
                Console.WriteLine(code.Message);
                return Task.FromResult(0);
            },
        };
        var credential = new DeviceCodeCredential(options);

        var authProvider = new AzureIdentityAuthenticationProvider(credential, allowedHosts, scopes: graphScopes);
        var adapter = new HttpClientRequestAdapter(authProvider);
        adapter.BaseUrl = "https://graph.microsoft.com/v1.0";
        return adapter;
    }).RegisterCommonServices();

return await builder.Build().InvokeAsync(args);
```

> [!NOTE]
> This example uses the **DeviceCodeCredential** class. You can use any of the credential classes from the `Azure.Identity` library.

## Run the application

Run the following command in your project directory to start the application.

```bash
dotnet run -- me get
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/cli) contains the code from this guide.
- [Additional CLI samples](https://github.com/microsoftgraph/msgraph-cli/tree/main/samples)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
