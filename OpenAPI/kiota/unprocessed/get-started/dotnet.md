---
parent: Get started
---

# Build API clients with anonymous authentication for .NET

In this tutorial, you will build a sample app in .NET that calls a REST API that does not require authentication.

## Required tools

- [.NET SDK 7.0](https://get.dot.net/7)

## Target project requirements

Before you can compile and run the generated files, you will need to make sure they are part of a project with the required dependencies. After creating a new project, or reusing an existing one, you will need to add references to the [abstraction](https://github.com/microsoft/kiota-abstractions-dotnet), [http](https://github.com/microsoft/kiota-http-dotnet), [form](https://github.com/microsoft/kiota-serialization-form-dotnet), [JSON](https://github.com/microsoft/kiota-serialization-json-dotnet) and [text](https://github.com/microsoft/kiota-serialization-text-dotnet) serialization packages from the GitHub feed.

## Creating target projects

> **Note:** you can use an existing project if you have one, in that case, you can skip the following section.

Execute the following command in the directory you want to create a new project.

```bash
dotnet new console -o KiotaPosts
```

> **Note:** in this example the console template is used, but you can use any C# template.

## Adding dependencies

```bash
dotnet add package Microsoft.Kiota.Abstractions
dotnet add package Microsoft.Kiota.Http.HttpClientLibrary
dotnet add package Microsoft.Kiota.Serialization.Form
dotnet add package Microsoft.Kiota.Serialization.Json
dotnet add package Microsoft.Kiota.Serialization.Text
```

Only the first package, `Microsoft.Kiota.Abstractions` is required. The other packages provide default implementations that you can choose to replace with your own implementations if you wish.

## Generating the SDK

Kiota generates SDKs from OpenAPI documents. Create a file named **posts-api.yml** and add the contents of the [JSONPlaceholder REST API OpenAPI description](reference-openapi.md).

You can then use the Kiota command line tool to generate the SDK classes.

```bash
kiota generate -l CSharp -c PostsClient -n KiotaPosts.Client -d ./posts-api.yml -o ./Client
```

## Creating the client application

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

> **Note:**
>
> The [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/) doesn't require any authentication, so this sample uses the **AnonymousAuthenticationProvider**. For APIs that require authentication, use an applicable authentication provider.
>
> - If the target API relies on an API key for authentication, you can use the **ApiKeyAuthenticationProvider** instead.
> - If the target API requires an `Authorization bearer <token>` containing a token issued by the Microsoft identity platform, you can use the **AzureIdentityAuthenticationProvider** class from the **Microsoft.Kiota.Authentication.Azure** package.
> - If the target API requires an `Authorization bearer <token>` header but doesn't rely on the Microsoft identity platform, you can implement your own authentication provider by inheriting from **BaseBearerTokenAuthenticationProvider**.
> - If the target API requires any other form of authentication schemes, you can implement the **IAuthenticationProvider** interface.

## Executing the application

When ready to execute the application, execute the following command in your project directory.

```bash
dotnet run
```

## See also

- [kiota-samples repository](https://github.com/microsoft/kiota-samples/tree/main/get-started/dotnet) contains the code from this guide.
- [Microsoft Graph sample using Microsoft identity platform authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/msgraph/dotnet)
- [Some other API sample using Auth0 authentication](https://github.com/microsoft/kiota-samples/tree/main/get-started/msgraph/dotnet)
- [ToDoItem Sample API](https://github.com/microsoft/kiota-samples/tree/main/sample-api) implements a sample OpenAPI in ASP.NET Core and sample clients in multiple languages.
