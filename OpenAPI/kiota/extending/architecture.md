---
title: Kiota code generation architecture
description: Learn how Kiota's code generation pipeline transforms OpenAPI descriptions into language-specific API clients.
author: baywet
ms.author: vibiret
ms.topic: conceptual
ms.date: 03/21/2026
---

# Kiota code generation architecture

Kiota generates API client SDKs by processing an OpenAPI description through a four-stage pipeline. Understanding this pipeline and the abstractions at each stage is essential before adding a new language target.

## The code generation pipeline

Generation flows through four stages, orchestrated by `KiotaBuilder` in the [Kiota repository](https://github.com/microsoft/kiota):

1. **Parse** &mdash; `KiotaBuilder` loads the OpenAPI description and builds an internal URL tree (`OpenApiUrlTreeNode`) representing every endpoint in the API.

1. **Build CodeDOM** &mdash; `CreateSourceModel()` walks the URL tree and produces a **Code Document Object Model (CodeDOM)**, a language-agnostic abstract syntax tree that represents the entire API client. This CodeDOM contains namespaces, classes, methods, properties, enums, and type references. It captures *what* the client does without committing to any language's syntax.

1. **Refine** &mdash; `ApplyLanguageRefinementAsync()` hands the CodeDOM to a language-specific `ILanguageRefiner`. The refiner post-processes the tree: adding import statements, replacing reserved names, adjusting type mappings, and restructuring elements to match target-language idioms. Each language has its own refiner class (for example, `DartRefiner`, `RubyRefiner`).

1. **Write** &mdash; `CreateLanguageSourceFilesAsync()` passes the refined CodeDOM to a `LanguageWriter`, which renders every element into source files on disk. A `PathSegmenter` determines the output file and folder structure for each element.

Your new language needs implementations at stages 3 and 4 (refiner and writer), plus supporting components described in the [adding a new language](add-language.md) guide.

## Key CodeDOM types

All CodeDOM types live in `src/Kiota.Builder/CodeDOM/` in the Kiota repository. The types you encounter most often are listed in the following table.

| Type | Purpose |
|---|---|
| `CodeElement` | Abstract base class for every node in the tree |
| `CodeNamespace` | A namespace, package, or module &mdash; the top-level container |
| `CodeClass` | A class with properties, methods, and inner declarations |
| `CodeMethod` | A method &mdash; covers request executors, serializers, constructors, and more (discriminated by `CodeMethodKind`) |
| `CodeProperty` | A typed property on a class |
| `CodeEnum` | An enumeration with named values |
| `CodeType` / `CodeTypeBase` | References to types, including generics and collection wrappers |
| `CodeParameter` | A parameter on a method |
| `CodeInterface` | An interface definition (used by languages that support them) |
| `CodeIndexer` | An indexer that enables `requestBuilder["id"]`-style navigation |

You don't modify these types when adding a language. Instead, you teach your writer and refiner how to handle them.

## The dispatcher pattern

`LanguageWriter` (`src/Kiota.Builder/Writers/LanguageWriter.cs`) uses a **type-keyed dispatcher** to render each CodeDOM element:

1. Each language writer (for example, `DartWriter`) calls `AddOrReplaceCodeElementWriter<T>()` in its constructor to register an `ICodeElementWriter<T>` for every code element type it can render.
1. These registrations are stored in a `Dictionary<Type, object>`.
1. When `Write<T>(element)` is called, the writer looks up the element's runtime type in the dictionary, casts to the matching `ICodeElementWriter<T>`, and calls `WriteCodeElement()`.

This means adding a language requires implementing a concrete `ICodeElementWriter<T>` for each element type your language must render &mdash; typically: class declarations, methods, properties, enums, block ends, and indexers.

## Components of a language implementation

At a high level, a complete language integration includes the following components. For detailed instructions on creating each one, see [Adding a new language](add-language.md).

- **`GenerationLanguage` enum entry** &mdash; Your language added to the `GenerationLanguage` enum.
- **Writer classes** &mdash; A folder under `src/Kiota.Builder/Writers/{Lang}/` containing:
  - `{Lang}Writer` &mdash; registers all element writers
  - `{Lang}ConventionService` &mdash; defines naming conventions, type mappings, and syntax helpers
  - Individual `ICodeElementWriter<T>` implementations for each code element type
- **Refiner** &mdash; `{Lang}Refiner` that applies language-specific CodeDOM transformations.
- **Reserved names provider** &mdash; `{Lang}ReservedNamesProvider` that lists language keywords for escape handling.
- **Path segmenter** &mdash; `{Lang}PathSegmenter` that controls how namespaces map to file paths.
- **Factory registrations** &mdash; Wiring your classes into the existing factory methods.
- **Language metadata** &mdash; Configuration for maturity level, dependencies, and supported features.
