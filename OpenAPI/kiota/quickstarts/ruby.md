---
title: Build API clients for Ruby
description: Learn how use Kiota to build API clients in Ruby.
author: baywet
ms.author: vibiret
ms.topic: quickstart
ms.date: 11/02/2026
---

# Build API clients for Ruby

In this tutorial, you build a sample app in Ruby that calls a REST API that doesn't require authentication.

## Required tools

- [Ruby 3.3 or newer](https://www.ruby-lang.org/en/downloads/)
- [Bundler](https://bundler.io/)

## Create a project

Create a directory to contain the new project, and create a new file named **Gemfile** in it.

## Add dependencies

Before you can compile and run the generated API client, you need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-ruby/tree/main/components/abstractions). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of the following packages.

- HTTP ([Kiota default Faraday-based implementation](https://github.com/microsoft/kiota-ruby/tree/main/components/http))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-ruby/tree/main/components/serialization/json))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-ruby/tree/main/components/serialization/text))
- Form serialization ([Kiota default](https://github.com/microsoft/kiota-ruby/tree/main/components/serialization/form))
- Multipart serialization ([Kiota default](https://github.com/microsoft/kiota-ruby/tree/main/components/serialization/multipart))

For this tutorial, use the default implementations.

1. Edit your **Gemfile** and add the following code.

    ```ruby
    source 'https://rubygems.org'

    ruby ">= 3.3.0"

    gem "microsoft_kiota_abstractions"
    gem "microsoft_kiota_faraday"
    gem "microsoft_kiota_serialization_json"
    gem "microsoft_kiota_serialization_text"
    gem "microsoft_kiota_serialization_form"
    gem "microsoft_kiota_serialization_multipart"
    ```

2. To install dependencies, run the following command.

    ```bash
    bundle install
    ```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **posts-api.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/quickstart/posts-api.yml":::

This file is a minimal OpenAPI description that describes how to call the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/).

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l ruby -c PostsClient -n KiotaPosts -d posts-api.yml -o ./client
```

> [!TIP]
> Add [`--exclude-backward-compatible`](../using.md#--exclude-backward-compatible---ebc)
> if you want to reduce the size of the generated client and are not concerned about
> potentially source breaking changes with future versions of Kiota when updating the client.

## Create the client application

Create a file in the root of the project named **main.rb** and add the following code.

:::code language="ruby" source="~/code-snippets/get-started/quickstart/ruby/main.rb" id="ProgramSnippet":::

> [!NOTE]
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.

## Run the application

To start the application, run the following command in your project directory.

```bash
bundle exec ruby main.rb
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstart/ruby) contains the code from this guide.
- [Microsoft Graph sample using Microsoft identity platform authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/ruby)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
