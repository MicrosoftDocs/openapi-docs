---
title: Kiota design overview
description: Learn about Kiota design and architecture.
author: jasonjoh
ms.author: jasonjoh
ms.topic: overview
date: 03/10/2023
---

# Kiota design overview

The Kiota SDK code generation process is designed to take an OpenAPI description, parse it, and generate a set of classes that make it simple for developers to make HTTP requests to the operations described by the OpenAPI.

![An image depicting the steps to generate source code from an OpenAPI description](./images/designoverview.png)

## OpenAPI description

The OpenAPI description is parsed using the `Microsoft.OpenApi.NET` library. This library can accept OpenAPI V2 or V3, in JSON or YAML format and provided as either a `String`, `Stream`, or `TextReader`. This library can parse and validate 100,000 lines of OpenAPI YAML/sec and therefore meets the scaling requirements for Kiota.

The output of this library is an `OpenAPIDocument` object that is a DOM for the OpenAPI description. It provides a dictionary of `PathItem` objects that correspond to the resources made available by the API. However, each `PathItem` object uses a URL path string as their unique identifier. Most SDK code generators take this flat list of `PathItem` objects, expand down to the `Operation` object supported for each `PathItem`, and generate a class with methods that correspond to those operations. For large APIs, this approach creates a class with a large surface area and makes it challenging to create readable and discoverable method names, especially for APIs with a deeply nested hierarchy.

HTTP APIs scale in size naturally due to the hierarchical nature of the URL path. In order to take advantage of this scaling, the generated code must use this characteristic to its advantage. However OpenAPI descriptions don't naturally represent that hierarchy and so it must be constructed.

## URI space tree

Walking the list of `PathItems` and building a resource hierarchy is a simple process. This approach presents an opportunity to apply a filter to limit the `PathItems` that have code generated. Many application developers are sensitive to the size of their client applications. By selectively including only resources that are used by a client application, libraries can be kept small. It's essential that when more functionality is adopted that existing generated code doesn't change.

## Code model

Most SDK generator efforts use a combination of API description of DOM and templates to create source code. Templates have several disadvantages:

- Syntax differences between languages cause semantic duplication between languages. This causes there to be lots of redundancies in templates.
- Small amounts of logic required when conditionally generating source can introduce complex syntax inside templates that is hard to test, debug, and maintain.
- It's easy for the behavior and capabilities of the different languages to diverge as language templates are independent. This divergence can be problematic if you're trying to provide consistent feature support across languages.

Kiota takes a different approach to code generation. The OpenAPI PathItems are analyzed and a corresponding language agnostic code model is created. This code model relies on standard object-oriented features that are available in all recent mainstream languages.

The code model built for Kiota is designed to support the features needed to create the thin "typing" veneer over a core HTTP library. It isn't intended to provide a comprehensive code model for any arbitrary code. This approach of using a common code model is feasible because we're targeting modern object oriented languages and limiting the scenarios to making HTTP requests.

## Refine code model by language

Despite the narrow scenario, there are still scenarios where, in order to create idiomatic source code, adjustments are required to the code model before emitting the source code.

The changes to the model made in this section should be as minimal as possible as they'll likely cause more maintenance in the future.

This section can be used to import language framework dependencies required for the generated source code. It can also be used for updating the casing of identifiers the casing appropriate for the language.

All the different code constructs that are modeled are derived from a base `CodeElement` class. Some `CodeElements` can contain other `CodeElements`. This approach allows traversing the `CodeElement` hierarchy uniformly and enables emitting source code in the desired order. This section can reorder peer elements based on language conventions.

## Generate source files

The last step of the process is to take the code model and translate it to language specific text. Traversing the `CodeElement` hierarchy and applying a concrete `LanguageWriter` implementation outputs the desired source code.

Implementing a language support for a new language requires, at minimum, implementing a class that derives from `LanguageWriter`.

## Kiota abstractions

The generated source code doesn't contain the logic to make HTTP requests directly. It requires being connected to a core library that makes the HTTP requests. A `Kiota.Core` library provides basic HTTP functionality, but API owners can choose to provide their own core libraries that are optimized for their API.

Core libraries take a dependency on the Kiota abstractions library for a language and provide the services of making the HTTP call to the API.

![An image depicting the relationship between the generated code and the abstractions library](./images/KiotaAbstractions.png)

This abstraction prevents Kiota from taking a dependency on a particular HTTP client library for a platform. Kiota remains focused on providing a consistent experience for discovering resources, creating requests, and deserializing responses.
