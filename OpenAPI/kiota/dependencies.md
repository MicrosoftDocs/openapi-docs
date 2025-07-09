---
title: Managing dependencies of Kiota API clients
description: Learn what different dependencies bring to your project and how to manage them.
author: baywet
ms.author: vibiret
ms.topic: conceptual
ms.date: 03/21/2023
---

# Managing dependencies of Kiota API clients

Kiota clients require multiple dependencies to function properly. In this documentation page, you find details about each dependency, which package to use in each language and guidance on how to properly manage these dependencies for your application.

## Guidance

### Generated client and dependencies versions alignment

For the best experience, after generating or updating a client, always ensure the kiota dependencies are aligned with the versions recommended by the info command.

[More information about the info command](./using.md#language-information)

### Dependencies selection

In most cases, the [bundle](#bundle) package and any [additionally required package](#additional) for the language you're using brings most dependencies your application needs.

If your application requires a specific [HTTP](#http) client, [serialization](#serialization) library or format, you need to at least add the abstractions and additionally required dependencies for the language you're using. Additionally, you can select default implementations for HTTP and serialization packages, or swap them for third party or your own implementations.

Additionally, your application might need to reference an [authentication](#authentication) package depending on the target API.

## Abstractions

Any Kiota generated API client relies on a set of abstractions as a build and runtime dependency, the application will require this dependency after a new client is generated.

[More information about the abstractions package](./abstractions.md)

The following table provides the list of abstractions package per language.

| Language | Package Name |
| -------- | ------------ |
| C# | Microsoft.Kiota.Abstractions |
| Go | github.com/microsoft/kiota-abstractions-go |
| Java | com.microsoft.kiota:microsoft-kiota-abstractions |
| PHP | microsoft/kiota-abstractions |
| Python | microsoft-kiota-abstractions |
| Ruby | microsoft_kiota_abstractions |
| Swift | Not released |
| TypeScript | @microsoft/kiota-abstractions |

## Serialization

Kiota provides default implementation for diverse serialization formats. These dependencies are required at runtime. Depending the application requirements those implementations can be used as is, swapped or augmented with implementations from the application, or swapped with third party implementations.

[More information about the serialization packages](./serialization.md)

### JSON

Media type: application/json

The following table provides the list of serialization package per language.

| Language | Package Name | Relies on |
| -------- | ------------ | --------- |
| C# | Microsoft.Kiota.Serialization.Json | System.Text.Json |
| Go | github.com/microsoft/kiota-serialization-json-go | None |
| Java | com.microsoft.kiota:microsoft-kiota-serialization-json | Gson |
| PHP | microsoft/kiota-serialization-json | None |
| Python | microsoft-kiota-serialization-json | None |
| Ruby | microsoft_kiota_serialization_json | None |
| Swift | Not released | None |
| TypeScript | @microsoft/kiota-serialization-json | None |

### Multipart

Media type: multipart/form-data

The following table provides the list of serialization package per language.

| Language | Package Name | Relies on |
| -------- | ------------ | --------- |
| C# | Microsoft.Kiota.Serialization.Multipart | None |
| Go | github.com/microsoft/kiota-serialization-multipart-go | None |
| Java | com.microsoft.kiota:microsoft-kiota-serialization-multipart | None |
| PHP | microsoft/kiota-serialization-multipart | None |
| Python | microsoft-kiota-serialization-multipart | None |
| Ruby | Not released | None |
| Swift | Not released | None |
| TypeScript | @microsoft/kiota-serialization-multipart | None |

### Form

Media type: application/x-www-form-urlencoded

The following table provides the list of serialization package per language.

| Language | Package Name | Relies on |
| -------- | ------------ | --------- |
| C# | Microsoft.Kiota.Serialization.Form | None |
| Go | github.com/microsoft/kiota-serialization-form-go | None |
| Java | com.microsoft.kiota:microsoft-kiota-serialization-form | None |
| PHP | microsoft/kiota-serialization-form | None |
| Python | microsoft-kiota-serialization-form | None |
| Ruby | Not Released | None |
| Swift | Not released | None |
| TypeScript | @microsoft/kiota-serialization-form | None |

### Text

Media type: text/plain

The following table provides the list of serialization package per language.

| Language | Package Name | Relies on |
| -------- | ------------ | --------- |
| C# | Microsoft.Kiota.Serialization.Text | None |
| Go | github.com/microsoft/kiota-serialization-text-go | None |
| Java | com.microsoft.kiota:microsoft-kiota-serialization-text | None |
| PHP | microsoft/kiota-serialization-text | None |
| Python | microsoft-kiota-serialization-text | None |
| Ruby | Not Released | None |
| Swift | Not released | None |
| TypeScript | @microsoft/kiota-serialization-text | None |

## Http

Kiota provides a default request adapter implementation to convert abstract request information objects created by the generated fluent API to HTTP requests. This dependency is required at runtime. Depending the application requirements this implementation can be used as is, swapped with another implementation from the application, or swapped with a third party implementation.

[More information about the HTTP package](./middleware.md).

The following table provides the list of HTTP package per language.

| Language | Package Name | Relies on |
| -------- | ------------ | --------- |
| C# | Microsoft.Kiota.Http.HttpClientLibrary | System.Net.Http |
| Go | github.com/microsoft/kiota-http-go | net/http |
| Java | com.microsoft.kiota:microsoft-kiota-http-okHttp | None |
| PHP | microsoft/kiota-http-guzzle | None |
| Python | microsoft-kiota-http | None |
| Ruby | microsoft_kiota_faraday | None |
| Swift | Not released | None |
| TypeScript | @microsoft/kiota-http-fetchlibrary | None |

## Authentication

Depending on authentication requirements from the target REST APIs, your application might additionally need an authentication dependency. Kiota provides default implementation providers for specific scenarios. Depending the application requirements those implementations can be used as is, swapped or augmented with implementations from the application, or swapped with third party implementations.

[More information about the authentication package](./authentication.md)

### Azure Identity

These providers might be used for any API secured with Bearer token authorization for which the identity platform is Entra Identity/Microsoft Identity Platform/Azure Identity/Azure Active Directory.

The following table provides the list of authentication package per language.

| Language | Package Name | Relies on |
| -------- | ------------ | --------- |
| C# | Microsoft.Kiota.Authentication.Azure | Azure.Identity |
| Go | github.com/microsoft/kiota-authentication-azure-go | github.com/Azure/azure-sdk-for-go/sdk/azidentity |
| Java | com.microsoft.kiota:microsoft-kiota-authentication-azure | com.azure:azure-identity |
| PHP | microsoft/kiota-authentication-phpleague | phpleague |
| Python | microsoft-kiota-authentication-azure | azure-identity |
| Ruby | microsoft_kiota_authentication_oauth | oauth |
| Swift | Not released | None |
| TypeScript | @microsoft/kiota-authentication-azure | @azure/identity |

## Additional

Additional dependencies are required at build time and/or runtime.

The following table provides the list of additional package per language.

| Language | Package Name | Type |
| -------- | ------------ | ---- |
| C# | None | |
| Go | None | |
| Java | jakarta.annotation:jakarta.annotation-api:2.0.0 | Build |
| PHP | None | |
| Python | None | |
| Ruby | None | |
| Swift | None | |
| TypeScript | None | |

## Bundle

To simplify dependencies management, kiota now offers bundle packages. These packages contain the abstractions, the default serialization implementations, and the default http implementation. Bundle packages can be used as an alternative to adding references to those packages.

The following table provides the list of bundle package per language.

| Language | Package Name |
| -------- | ------------ |
| C# | Microsoft.Kiota.Bundle |
| Go | github.com/microsoft/kiota-bundle-go |
| Java | com.microsoft.kiota:microsoft-kiota-bundle |
| PHP | Not available |
| Python | Not available |
| Ruby | Not available |
| Swift | Not available |
| TypeScript | @microsoft/kiota-bundle |
