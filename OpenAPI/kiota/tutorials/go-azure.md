---
title: Build API clients for Go with Microsoft identity authentication
description: Learn how use Kiota to build API clients in Go to access APIs that require Microsoft identity authentication.
author: jasonjoh
ms.author: jasonjoh
ms.topic: tutorial
date: 03/20/2023
---

# Build API clients for Go with Azure authentication

In this tutorial, you will generate an API client that uses [Microsoft identity authentication](/azure/active-directory/fundamentals/auth-oauth2) to access [Microsoft Graph](/graph/overview).

## Required tools

- [Go 1.18](https://golang.org/dl/)

## Create a project

Run the following commands in the directory where you want to create a new project.

```bash
go mod init getuser
```

## Add dependencies

Before you can compile and run the generated API client, you will need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-abstractions-go). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of of the following packages.

- Authentication ([Kiota default Azure authentication](https://github.com/microsoft/kiota-authentication-azure-go))
- HTTP ([Kiota default net/http-based implementation](https://github.com/microsoft/kiota-http-go))
- Form serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-form-go))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-json-go))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-text-go))

For this tutorial, you will use the default implementations.

Run the following commands to get the required dependencies.

```bash
go get github.com/microsoft/kiota-abstractions-go
go get github.com/microsoft/kiota-http-go
go get github.com/microsoft/kiota-serialization-form-go
go get github.com/microsoft/kiota-serialization-json-go
go get github.com/microsoft/kiota-serialization-text-go
go get github.com/microsoft/kiota-authentication-azure-go
go get github.com/Azure/azure-sdk-for-go/sdk/azidentity
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **get-me.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/getme.yml":::

You can then use the Kiota command line tool to generate the API client classes.

```shell
kiota generate -l go -d ../get-me.yml -c GraphApiClient -n getuser/client -o ./client
```

## Register an application

[!INCLUDE [register-azure-app-device-code-intro](../includes/register-azure-app-device-code-intro.md)]

### Azure portal [#tab/portal]

[!INCLUDE [register-azure-app-device-code-portal](../includes/register-azure-app-device-code-portal.md)]

### PowerShell [#tab/powershell]

[!INCLUDE [register-azure-app-device-code-powershell](../includes/register-azure-app-device-code-powershell.md)]

## Create the client application

Create a file in the root of the project named **getuser.go** and add the following code. Replace `YOUR_CLIENT_ID` with the client ID from your app registration.

```go
package main

import (
  "context"
  "fmt"

  "getuser/client"

  "github.com/Azure/azure-sdk-for-go/sdk/azidentity"
  azure "github.com/microsoft/kiota-authentication-azure-go"
  http "github.com/microsoft/kiota-http-go"
)

func main() {
  clientId := "YOUR_CLIENT_ID"

  // The auth provider will only authorize requests to
  // the allowed hosts, in this case Microsoft Graph
  allowedHosts := []string{"graph.microsoft.com"}
  graphScopes := []string{"User.Read"}

  credential, err := azidentity.NewDeviceCodeCredential(&azidentity.DeviceCodeCredentialOptions{
    ClientID: clientId,
    UserPrompt: func(ctx context.Context, dcm azidentity.DeviceCodeMessage) error {
      fmt.Println(dcm.Message)
      return nil
    },
  })

  if err != nil {
    fmt.Printf("Error creating credential: %v\n", err)
  }

  authProvider, err := azure.NewAzureIdentityAuthenticationProviderWithScopesAndValidHosts(
    credential, graphScopes, allowedHosts)

  if err != nil {
    fmt.Printf("Error creating auth provider: %v\n", err)
  }

  adapter, err := http.NewNetHttpRequestAdapter(authProvider)

  if err != nil {
    fmt.Printf("Error creating request adapter: %v\n", err)
  }

  client := client.NewGraphApiClient(adapter)

  me, err := client.Me().Get(context.Background(), nil)

  if err != nil {
    fmt.Printf("Error getting user: %v\n", err)
  }

  fmt.Printf("Hello %s, your ID is %s\n", *me.GetDisplayName(), *me.GetId())
}
```

> [!NOTE]
> This example uses the **DeviceCodeCredential** class. You can use any of the credential classes from the `azidentity` library.

## Run the application

Run the following command in your project directory to start the application.

```shell
go run .
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/go) contains the code from this guide.
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
