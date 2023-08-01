---
title: Query Parameters and Request Headers
description: Learn how to use Query Parameters and Request Headers to customize API calls.
author: vibiret
ms.author: vibiret
ms.topic: conceptual
date: 08/01/2023
---

# Query Parameters and Request Headers

Kiota API clients provide a fluent API you can use to customize request before sending them out to the service.

## Query Parameters

Kiota will generate an additional type for each operation to reflect the query parameters in the API description.

The application code can then use that type to customize the request and benefit from compile time validations.

Consider the following sample:

```csharp
var result = await todoClient.TaskLists["taskListId"].ToDos["todoId"].AssignedTo.GetAsync(x => x.QueryParameters.Select = new string[] { "id", "title" });
```

The URL template for the requestBuilder will look something like `{+baseurl}/taskLists/{task_list_id}/toDos/{todo_id}/assignedTo{?select,expand}`.

And the resulting URI will be `https://contoso.com/taskLists/taskListId/toDos/todoId/assignedTo?select=id,title`

> [!NOTE]
> [Enums in Query parameters](https://github.com/microsoft/kiota/issues/2306) are encoded with open `String`.

## Request Headers

Kiota does not generate a type specific to the operation for request headers. The HTTP RFC specifies a large number of standard headers, additionally, vendors can come up with custom headers, as well as API developers. Projecting a type for request headers would be redundant and inflate the amount of code being generated.

The application code can use the fluent API to customize headers sent to the service via a generic **Headers** type.

```csharp
var result = await todoClient.TaskLists["taskListId"].ToDos["todoId"].AssignedTo.GetAsync(x => x.Headers.Add("Contoso-custom", "foo", "bar"));
```

Will result in the following request

```http
GET ...
Contoso-Custom: foo, bar
```

The request adapter will fold headers with multiple values automatically.

> [!NOTE]
> [Header values](https://github.com/microsoft/kiota/issues/2428) are limited to strings and the application must serialize other types.
