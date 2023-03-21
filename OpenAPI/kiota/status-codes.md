---
title: HTTP status code handling with Kiota API clients
description: Learn about the interfaces used for serialization and deserialization in Kiota clients.
author: baywet
ms.author: vibiret
ms.topic: conceptual
date: 03/21/2023
---

# HTTP status code handling with Kiota API clients

Kiota uses the specified HTTP status codes in OpenAPI descriptions to determine return types. It also attempts to handle cases where APIs return a status code other than the one specified in the OpenAPI description.

## Generation

During API client generation, Kiota follows these rules to map status codes described by the OpenAPI description. The following table is ordered which means the first rule that matches will be the one used during generation.

| Code          | Schema is present | Result type |
|---------------|-------------------|-------------|
| 200-203       | yes               | model class |
| 2XX           | yes               | model class |
| 204, 205      | N/A               | void        |
| 201, 202      | no                | void        |
| 200, 203, 206 | no                | stream      |
| 2XX           | no                | stream      |

> [!NOTE]
> For a schema to be considered present, it must be part of a structured content response (`application/json`, `application/xml`, `text/plain`, `text/xml`, `text/yaml`).

## Runtime status code handling

At runtime the client can encounter response status codes that differ from what was originally documented due to how HTTP works. Default request adapter implementations follow these rules:

| Expected return type | Response status code | Response body is present | Returned value     |
|----------------------|----------------------|--------------------------|--------------------|
| model class          | 200-203              | yes                      | deserialized value |
| model class          | 200-202, 204, 205    | no                       | null               |
| void                 | 200-205              | N/A                      | void               |
| stream               | 200-203, 206         | yes                      | stream             |
| stream               | 200-205              | no                       | null               |
