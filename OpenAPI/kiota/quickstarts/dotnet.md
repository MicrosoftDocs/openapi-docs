---
title: Build API clients for .NET
description: Learn how use Kiota to build API clients in .NET.
author: jasonjoh
ms.author: jasonjoh
ms.topic: quickstart
date: 03/20/2023
---

# Build API clients for .NET

In this tutorial, you will build a sample app in .NET that calls a REST API that does not require authentication.

## Required tools

- [.NET SDK 7.0](https://get.dot.net/7)

## Create a project

Run the following command in the directory you want to create a new project.

```bash
dotnet new console -o KiotaPosts
dotnet new gitignore
```

## Add dependencies

Before you can compile and run the generated API client, you will need to make sure the generated source files are part of a project with the required dependencies. Your project must have a reference to the [abstraction package](https://github.com/microsoft/kiota-abstractions-dotnet). Additionally, you must either use the Kiota default implementations or provide your own custom implementations of of the following packages.

- HTTP ([Kiota default HttpClient-based implementation](https://github.com/microsoft/kiota-http-dotnet))
- Form serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-form-dotnet))
- JSON serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-json-dotnet))
- Text serialization ([Kiota default](https://github.com/microsoft/kiota-serialization-text-dotnet))

For this tutorial, you will use the default implementations.

Run the following commands to get the required dependencies.

```bash
dotnet add package Microsoft.Kiota.Abstractions
dotnet add package Microsoft.Kiota.Http.HttpClientLibrary
dotnet add package Microsoft.Kiota.Serialization.Form
dotnet add package Microsoft.Kiota.Serialization.Json
dotnet add package Microsoft.Kiota.Serialization.Text
```

## Generate the API client

Kiota generates API clients from OpenAPI documents. Create a file named **posts-api.yml** and add the following.

:::code language="yaml" source="~/code-snippets/get-started/quickstart/posts-api.yml":::

This is a minimal OpenAPI description that describes how to call the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/).

You can then use the Kiota command line tool to generate the API client classes.

```bash
kiota generate -l CSharp -c PostsClient -n KiotaPosts.Client -d ./posts-api.yml -o ./Client
```

## Create the client application

The final step is to update the **Program.cs** file that was generated as part of the console application to include the code below.

```csharp
using KiotaPosts.Client;
using KiotaPosts.Client.Models;
using Microsoft.Kiota.Abstractions.Authentication;
using Microsoft.Kiota.Http.HttpClientLibrary;

// API requires no authentication, so use the anonymous
// authentication provider
var authProvider = new AnonymousAuthenticationProvider();
// Create request adapter using the HttpClient-based implementation
var adapter = new HttpClientRequestAdapter(authProvider);
// Create the API client
var client = new PostsClient(adapter);

// GET /posts
var allPosts = await client.Posts.GetAsync();
Console.WriteLine($"Retrieved {allPosts?.Count} posts.");

// GET /posts/{id}
var specificPostId = "5";
var specificPost = await client.Posts[specificPostId].GetAsync();
Console.WriteLine($"Retrieved post - ID: {specificPost?.Id}, Title: {specificPost?.Title}, Body: {specificPost?.Body}");

// POST /posts
var newPost = new Post
{
    UserId = 42,
    Title = "Testing Kiota-generated API client",
    Body = "Hello world!"
};

var createdPost = await client.Posts.PostAsync(newPost);
Console.WriteLine($"Created new post with ID: {createdPost?.Id}");

// PATCH /posts/{id}
var update = new Post
{
    // Only update title
    Title = "Updated title"
};
var updatedPost = await client.Posts[specificPostId].PatchAsync(update);
Console.WriteLine($"Updated post - ID: {updatedPost?.Id}, Title: {updatedPost?.Title}, Body: {updatedPost?.Body}");

// DELETE /posts/{id}
await client.Posts[specificPostId].DeleteAsync();
```

> [!NOTE]
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.

## Run the application

Run the following command in your project directory to start the application.

```bash
dotnet run
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstarts/dotnet) contains the code from this guide.
- [Microsoft Graph sample using Microsoft identity platform authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/azure-auth/dotnet)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
