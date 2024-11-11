---
title: Build API clients for Ruby with Microsoft identity authentication
description: Learn how use Kiota to build API clients in Ruby to access APIs that require Microsoft identity authentication.
author: jasonjoh
ms.author: jasonjoh
ms.topic: tutorial
ms.date: 03/20/2023
---

# Build API clients for Ruby with Microsoft identity authentication

In this tutorial, you generate an API client that uses [Microsoft identity authentication](/azure/active-directory/fundamentals/auth-oauth2) to access [Microsoft Graph](/graph/overview).

## Required tools

- [Ruby 3](https://www.ruby-lang.org/en/downloads/)
- [Bundler](https://bundler.io/)

## Create a project

Create a new file named **Gemfile** in the root directory of your project.

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-abstractions-ruby). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of the following packages.

- Authentication ([Kiota default Azure authentication](https://github.com/microsoft/kiota-authentication-oauth-ruby))
- HTTP ([Kiota default Faraday-based implementation](https://github.com/microsoft/kiota-http-ruby))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-json-ruby))

For this tutorial, use the default implementations.

1. Edit your **Gemfile** and add the following code.

    ```ruby
    source 'https://rubygems.org'

    gem "microsoft_kiota_abstractions"
    gem "microsoft_kiota_serialization_json"
    gem "microsoft_kiota_authentication_oauth"
    gem "microsoft_kiota_faraday"
    ```

2. To install dependencies, run the following command.

    ```bash
    bundle install
    ```

### Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **get-me.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/azure-auth/get-me.yml":::

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l ruby -d get-me.yml -n Graph -o ./client
```

Lastly, create a file called **graph.rb** in the `client` folder that was created by Kiota. Add the following code:

```ruby
# frozen_string_literal: true
module Graph
end
```

## Register an application

[!INCLUDE [register-azure-app-auth-code-intro](../includes/register-azure-app-auth-code-intro.md)]

<!-- markdownlint-disable MD051 -->
### [Azure portal](#tab/portal)

[!INCLUDE [register-azure-app-auth-code-portal](../includes/register-azure-app-auth-code-portal.md)]

### [PowerShell](#tab/powershell)

[!INCLUDE [register-azure-app-auth-code-powershell](../includes/register-azure-app-auth-code-powershell.md)]
<!-- markdownlint-enable MD051 -->

---

## Create the client application

Create a file in the root of the project named **get_user.rb** and add the following code. Replace the `client_id` and `client_secret` with your credentials from the previous step.

[!INCLUDE [get-azure-auth-code-manually](../includes/get-azure-auth-code-manually.md)]

Replace the `auth_code` with your authorization code.

:::code language="ruby" source="~/code-snippets/get-started/azure-auth/ruby/get_user.rb" id="ProgramSnippet":::

## Run the application

To start the application, run the following command in your project directory.

```bash
ruby ./get_user.rb
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/ruby) contains the code from this guide.
