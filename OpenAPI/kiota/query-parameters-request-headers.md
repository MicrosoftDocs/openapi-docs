---
title: Query parameters and request headers
description: Learn how to use query parameters and request headers to customize API calls.
author: baywet
ms.author: vibiret
ms.topic: conceptual
date: 08/01/2023
---

# Query parameters and request headers

Kiota API clients provide a fluent API you can use to add query parameters or headers to requests before sending them out to the service.

## Query parameters

Kiota generates a type for each operation to reflect the query parameters in the API description. The application code can then use that type to customize the request and benefit from compile time validations.

Consider the following sample:

```csharp
var result = await todoClient
    .TaskLists["taskListId"]
    .ToDos["todoId"]
    .AssignedTo
    .GetAsync(x => x.QueryParameters.Select = new string[] { "id", "title" });
```

The URL template for the request builder looks something like `{+baseurl}/taskLists/{task_list_id}/toDos/{todo_id}/assignedTo{?select,expand}`.

And the resulting URI is `https://contoso.com/taskLists/taskListId/toDos/todoId/assignedTo?select=id,title`

> [!NOTE]
> [Enumerations in query parameters](https://github.com/microsoft/kiota/issues/2306) are treated as strings.

## Request headers

Kiota doesn't generate a type specific to the operation for request headers. The HTTP specification defines a large number of standard headers, and vendors can come up with custom headers. Projecting a type for request headers would be redundant and inflate the amount of code being generated.

The application code can use the fluent API to customize headers sent to the service via a generic **Headers** type.

```csharp
var result = await todoClient
    .TaskLists["taskListId"]
    .ToDos["todoId"]
    .AssignedTo
    .GetAsync(x => x.Headers.Add("Contoso-Custom", "foo", "bar"));
```

The code above results in the following request.

```http
GET ...
Contoso-Custom: foo, bar
```

The request adapter folds headers with multiple values automatically.

> [!NOTE]
> [Header values](https://github.com/microsoft/kiota/issues/2428) are limited to strings and the application must serialize other types.
