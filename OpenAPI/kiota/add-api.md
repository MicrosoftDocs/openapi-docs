---
title: Adding an API to search
description: Learn how to add an API description to the Kiota search command.
author: baywet
ms.author: vibiret
date: 03/10/2023
---

# Adding an API to search

The search command relies on multiple indices to find OpenAPI descriptions that can be used to generate clients. If you maintain APIs, you can add your own description to the results by updating one of the following indices:

- [GitHub](#github)
- [APIs.guru](#apis-guru)

## GitHub

You can add your API descriptions to the search result by following these steps:

1. Add the **kiota-index** topic to your GitHub repository.
2. Add an **apis.json** or **apis.yaml** file at the root of your repository on your main branch.
3. Make sure the repository is public and that it has a description.

### Example apis.yaml

```yaml
name: Microsoft Graph OpenAPI registry
description: This repository contains the OpenAPI definitions for Microsoft Graph APIs.
tags:
- microsoft
apis:
- name: Microsoft Graph v1.0
  description: The Microsoft Graph API offers a single endpoint, to provide access to rich, people-centric data and insights in the Microsoft cloud.
  baseUrl: https://graph.microsoft.com/v1.0
  properties:
  - type: x-openapi
    url: openapi/v1.0/openapi.yaml # can be the relative path on the repository or a publicly accessible URL

```

> [!NOTE]
> For more information about apis.yaml, see [What is APIs.yaml?](http://apisyaml.org/).

### Example apis.json

```jsonc
{
    "name": "Microsoft Graph OpenAPI registry",
    "description": "This repository contains the OpenAPI definitions for Microsoft Graph APIs.",
    "tags": [
        "microsoft"
    ],
    "apis": [
        {
            "name": "Microsoft Graph v1.0",
            "description": "The Microsoft Graph API offers a single endpoint, to provide access to rich, people-centric data and insights in the Microsoft cloud.",
            "baseUrl": "https://graph.microsoft.com/v1.0",
            "properties": [
                {
                    "type": "x-openapi",
                    "url": "openapi/v1.0/openapi.yaml" // can be the relative path on the repository or a publicly accessible URL
                }
            ]
        }
    ]
}
```

> [!NOTE]
> For more information about apis.json, see [What is APIs.json?](http://apisjson.org/).

## APIs guru

You can add your description to the APIs guru index by [filling the following form](https://apis.guru/add-api/).
