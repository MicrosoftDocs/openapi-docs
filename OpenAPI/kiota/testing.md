---
title: Unit testing Kiota API clients
description: Learn about writing unit tests for Kiota API clients.
author: LockTar
ms.author: vibiret
ms.topic: conceptual
date: 07/04/2023
---

# Unit testing Kiota API clients

Unit testing API clients against the actual API can often be impractical. Issues like network failures, authentication requirements, or changing data can make testing difficult or impossible. We recommend that unit tests use mock implementations of the HTTP transport layer to control API responses.

For Kiota API clients, the HTTP layer is contained in a [request adapter](abstractions.md#request-adapter). Mocking the request adapter allows you to control API responses.

## Prerequisite

In this sample, the following frameworks/tools are used.

- [xUnit](https://xunit.net/) - unit testing framework for .NET
- [NSubstitute](https://nsubstitute.github.io/) - mocking library to mock interfaces

## Kiota client unit testing

The following unit tests are for an API client generated for the `/posts` endpoint in the [JSONPlaceholder REST API](https://jsonplaceholder.typicode.com/). The full project can be downloaded from [GitHub](https://github.com/microsoft/kiota-samples/tree/main/get-started/quickstart/dotnet).

:::code language="csharp" source="~/code-snippets/get-started/quickstart/dotnet/test/PostsRequestsTests.cs" id="UnitTestsSnippet":::
