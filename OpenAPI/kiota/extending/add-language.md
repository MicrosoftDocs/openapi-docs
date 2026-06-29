---
title: Add a new language to Kiota
description: Learn how to add support for a new programming language target to the Kiota API client generator.
author: baywet
ms.author: vibiret
ms.topic: how-to
ms.date: 03/21/2026
---

# Add a new language to Kiota

This guide walks you through adding a new programming language target to the [Kiota](https://github.com/microsoft/kiota) API client generator. Whether you're a Microsoft engineer or a community contributor, the process is the same &mdash; Kiota was designed to support community-implemented languages.

## Prerequisites

Before you begin, you should be familiar with:

- **C# / .NET** &mdash; Kiota is written in C#; all generator code is .NET.
- **Your target language** &mdash; You need deep knowledge of its syntax, type system, and package ecosystem.
- **OpenAPI** &mdash; A basic understanding of OpenAPI descriptions and how they map to API client concepts.

Before starting, make sure you can [build and test Kiota locally](https://github.com/microsoft/kiota/blob/main/CONTRIBUTING.md). You should also read the [architecture overview](architecture.md) to understand the code generation pipeline.

## Step 1: Add the language to GenerationLanguage

**File:** `src/Kiota.Builder/GenerationLanguage.cs`

Add a new member to the `GenerationLanguage` enum:

```csharp
public enum GenerationLanguage
{
    CSharp,
    Java,
    TypeScript,
    PHP,
    Python,
    Go,
    Ruby,
    Dart,       // ← existing example
    YourLang,   // ← add your language here
    HTTP
}
```

This enum is the primary identifier for your language throughout the entire codebase. Every factory method and switch statement dispatches on it, so you must do this first.

## Step 2: Create the language writer

Create a new folder at `src/Kiota.Builder/Writers/{Language}/`. This folder contains the writer, the convention service, and one code element writer per CodeDOM node type.

### The writer class

**File:** `src/Kiota.Builder/Writers/{Language}/{Language}Writer.cs`

Extend `LanguageWriter` and register all element writers in the constructor. The following example shows how `DartWriter` is implemented:

```csharp
public class DartWriter : LanguageWriter
{
    public DartWriter(string rootPath, string clientNamespaceName)
    {
        PathSegmenter = new DartPathSegmenter(rootPath, clientNamespaceName);
        var conventionService = new DartConventionService();
        AddOrReplaceCodeElementWriter(new CodeClassDeclarationWriter(conventionService, clientNamespaceName, (DartPathSegmenter)PathSegmenter));
        AddOrReplaceCodeElementWriter(new CodeBlockEndWriter());
        AddOrReplaceCodeElementWriter(new CodeEnumWriter(conventionService));
        AddOrReplaceCodeElementWriter(new CodeMethodWriter(conventionService));
        AddOrReplaceCodeElementWriter(new CodePropertyWriter(conventionService));
        AddOrReplaceCodeElementWriter(new CodeTypeWriter(conventionService));
    }
}
```

The constructor must:

1. Instantiate and assign the **path segmenter** (see [Step 5](#step-5-create-the-path-segmenter)).
1. Create a shared **convention service** instance.
1. Call `AddOrReplaceCodeElementWriter(...)` for every CodeDOM element your language needs to render.

The element writers you typically need are listed in the following table.

| Writer class | Renders |
|---|---|
| `CodeClassDeclarationWriter` | Class/interface opening (imports, declaration, inheritance) |
| `CodeBlockEndWriter` | Closing braces or equivalent |
| `CodeEnumWriter` | Enum type declarations |
| `CodeMethodWriter` | Methods (constructors, getters, serializers, request executors) |
| `CodePropertyWriter` | Property declarations |
| `CodeTypeWriter` | Type file wrappers |

Each element writer typically extends `BaseElementWriter<TElement, TConventionService>`, which provides access to the convention service and implements `ICodeElementWriter<TElement>`. The `WriteCodeElement(TElement, LanguageWriter)` method is where rendering happens.

### The convention service

**File:** `src/Kiota.Builder/Writers/{Language}/{Language}ConventionService.cs`

Extend `CommonLanguageConventionService`. This class centralizes all language-specific naming, typing, and formatting decisions. Key members to override are listed in the following table.

| Member | Purpose | Dart example |
|---|---|---|
| `StreamTypeName` | Name for byte-stream types | `"Stream"` |
| `VoidTypeName` | Keyword for void return | `"void"` |
| `DocCommentPrefix` | Prefix for documentation comments | `"/// "` |
| `ParseNodeInterfaceName` | Name of the parse node interface | `"ParseNode"` |
| `GetAccessModifier(AccessModifier)` | Map access levels to language syntax | Returns `"_"` for private, `""` for public |
| `GetTypeString(CodeTypeBase, ...)` | Resolve a CodeDOM type to its language-specific string | `"List<String>?"` |
| `TranslateType(CodeType)` | Map Kiota type names to native names | `"integer"` → `"int"` |
| `GetParameterSignature(CodeParameter, ...)` | Format a parameter for method signatures | `"{String name = ""}"` |
| `WriteShortDescription(IDocumentedElement, ...)` | Emit a single-line doc comment | Writes `/// description` |

The `TranslateType` method is particularly important &mdash; it maps Kiota's internal type names to your language's native types. The following example shows the Dart implementation:

```csharp
public override string TranslateType(CodeType type)
{
    return type.Name.ToLowerInvariant() switch
    {
        "integer" or "sbyte" or "byte" or "int64" => "int",
        "boolean" => "bool",
        "string" => "String",
        "double" or "float" or "decimal" => "double",
        "binary" or "base64" or "base64url" => "Iterable<int>",
        "iparsenode" => "ParseNode",
        "iserializationwriter" => "SerializationWriter",
        _ => type.Name.ToFirstCharacterUpperCase() is string typeName
             && !string.IsNullOrEmpty(typeName) ? typeName : "Object",
    };
}
```

## Step 3: Create the language refiner

**File:** `src/Kiota.Builder/Refiners/{Language}Refiner.cs`

Extend `CommonLanguageRefiner` and implement `ILanguageRefiner`. The refiner transforms the language-agnostic CodeDOM tree into a form suitable for your language **before** any code is written. Its `RefineAsync` method is the single entry point.

Key responsibilities include:

- **Adding default imports** &mdash; Call `AddDefaultImports(generatedCode, defaultUsingEvaluators)` with an array of `AdditionalUsingEvaluator` rules that match predicates and add import statements.
- **Correcting names** &mdash; Call `CorrectNames(generatedCode, transformFunc)` to apply your language's casing conventions.
- **Handling reserved words** &mdash; Call `ReplaceReservedNames(generatedCode, reservedNamesProvider, x => $"{x}_")` to escape identifiers that collide with language keywords.
- **Type adjustments** &mdash; Call `CorrectCoreType(generatedCode, CorrectMethodType, CorrectPropertyType, CorrectImplements)` with language-specific callbacks.
- **Restructuring** &mdash; Methods like `MoveRequestBuilderPropertiesToBaseType`, `RemoveRequestConfigurationClasses`, and `AddParsableImplementsForModelClasses` restructure the CodeDOM.
- **Serialization** &mdash; Call `ReplaceDefaultSerializationModules` and `ReplaceDefaultDeserializationModules` with your language's serialization factory class names.

`CommonLanguageRefiner` provides dozens of protected helper methods you can compose. Studying `DartRefiner.RefineAsync()` end-to-end is the best way to understand the ordering.

## Step 4: Create the reserved names provider

**File:** `src/Kiota.Builder/Refiners/{Language}ReservedNamesProvider.cs`

Implement `IReservedNamesProvider`, which requires a single property `HashSet<string> ReservedNames`. Populate it with every keyword and built-in identifier in your language:

```csharp
public class DartReservedNamesProvider : IReservedNamesProvider
{
    private readonly Lazy<HashSet<string>> _reservedNames =
        new(() => new(StringComparer.OrdinalIgnoreCase) {
            "abstract", "as", "assert", "async", "await",
            "break", "case", "catch", "class", "const",
            // ... all language keywords
            "BaseRequestBuilder", "clone"  // also reserve Kiota base class names
        });
    public HashSet<string> ReservedNames => _reservedNames.Value;
}
```

Include not just language keywords but also names from your Kiota abstractions library that could cause collisions (for example, `BaseRequestBuilder`). Use `StringComparer.OrdinalIgnoreCase` if your language is case-insensitive for identifiers.

## Step 5: Create the path segmenter

**File:** `src/Kiota.Builder/PathSegmenters/{Language}PathSegmenter.cs`

Extend `CommonPathSegmenter`. The path segmenter controls how the CodeDOM namespace tree maps to files and directories on disk.

```csharp
public class DartPathSegmenter(string rootPath, string clientNamespaceName)
    : CommonPathSegmenter(rootPath, clientNamespaceName)
{
    public override string FileSuffix => ".dart";
    public override string NormalizeNamespaceSegment(string segmentName)
        => segmentName.ToCamelCase();
    public override string NormalizeFileName(CodeElement currentElement)
        => GetLastFileNameSegment(currentElement).ToSnakeCase();
}
```

Override these three members:

- **`FileSuffix`** &mdash; The file extension for your language (for example, `.dart`, `.py`, `.rb`).
- **`NormalizeNamespaceSegment(string)`** &mdash; How namespace/package segments are cased in directory names.
- **`NormalizeFileName(CodeElement)`** &mdash; How file names are derived from code elements.

## Step 6: Register in all factory methods

This is the most critical step. Your language must be wired into every switch statement or factory that dispatches on `GenerationLanguage`. Missing even one registration means the language won't work for that code path.

### Language writer factory

**File:** `src/Kiota.Builder/Writers/LanguageWriter.cs` &mdash; method `GetLanguageWriter`

```csharp
GenerationLanguage.YourLang => new YourLangWriter(outputPath, clientNamespaceName),
```

### Language refiner dispatcher

**File:** `src/Kiota.Builder/Refiners/ILanguageRefiner.cs` &mdash; static method `RefineAsync`

```csharp
case GenerationLanguage.YourLang:
    await new YourLangRefiner(config)
        .RefineAsync(generatedCode, cancellationToken).ConfigureAwait(false);
    break;
```

### Public API export service

**File:** `src/Kiota.Builder/Export/PublicAPIExportService.cs` &mdash; method `GetLanguageConventionServiceFromConfiguration`

```csharp
GenerationLanguage.YourLang => new YourLangConventionService(),
```

### Code renderer factory (usually not needed)

**File:** `src/Kiota.Builder/CodeRenderers/CodeRenderer.cs` &mdash; method `GetCodeRender`

Most languages use the default `CodeRenderer` and don't need an entry here. The `_ => new CodeRenderer(config)` default arm handles them. Only add a case if you need custom rendering behavior (see [Step 8](#step-8-optional-custom-code-renderer)).

> [!TIP]
> After completing your registration, run `grep -r "GenerationLanguage\." src/` and verify that every switch/match that lists all languages includes yours.

## Step 7: Configure language metadata

**File:** `src/kiota/appsettings.json`

Add a new entry under the `Languages` section. This metadata drives the `kiota info` command and dependency management:

```json
"YourLang": {
  "MaturityLevel": "Experimental",
  "SupportExperience": "Community",
  "Dependencies": [
    {
      "Name": "your_kiota_abstractions",
      "Version": "0.0.1",
      "Type": "Abstractions"
    },
    {
      "Name": "your_kiota_http",
      "Version": "0.0.1",
      "Type": "Http"
    },
    {
      "Name": "your_kiota_serialization_json",
      "Version": "0.0.1",
      "Type": "Serialization"
    }
  ],
  "DependencyInstallCommand": "your-pkg-manager install \"{0}\" \"{1}\""
}
```

| Field | Description |
|---|---|
| `MaturityLevel` | One of `Experimental`, `Preview`, or `Stable`. Start with `Experimental`. |
| `SupportExperience` | `Community` or `Microsoft`, depending on who maintains the runtime libraries. |
| `Dependencies` | Array of runtime packages. `Type` is `Abstractions`, `Http`, `Serialization`, `Authentication`, or `Bundle`. |
| `DependencyInstallCommand` | Shell command template where `{0}` is the package name and `{1}` is the version. Leave empty if the language has no standard install command. |

## Step 8: (Optional) Custom code renderer

**File:** `src/Kiota.Builder/CodeRenderers/{Language}CodeRenderer.cs`

Most languages use the base `CodeRenderer`, which iterates the CodeDOM tree and delegates to the registered element writers. You only need a custom renderer if your language requires special file-level orchestration. For example:

- **TypeScript** uses `TypeScriptCodeRenderer` to generate barrel/index files that re-export symbols.
- **Python** uses a custom element order comparator.
- **Go** uses `CodeElementOrderComparerWithExternalMethods` to control declaration ordering.

For the vast majority of languages, the default is sufficient and you can skip this step.

## Writing tests

Every new language in Kiota needs thorough test coverage across writers, refiners, and conventions. Tests live in the `tests/Kiota.Builder.Tests/` project and use xUnit.

### Where tests live

| What | Location |
|---|---|
| Writer tests | `tests/Kiota.Builder.Tests/Writers/{Language}/` |
| Refiner tests | `tests/Kiota.Builder.Tests/Refiners/{Language}LanguageRefinerTests.cs` |
| Integration tests | `it/{language}/` |

### Testing pattern

All writer tests follow the same structure: create a `LanguageWriter`, capture output into a `StringWriter`, build CodeDOM elements, write them, and assert against the string output. Each test class is self-contained and implements `IDisposable`.

The following example shows the typical skeleton:

```csharp
public sealed class CodePropertyWriterTests : IDisposable
{
    private const string DefaultPath = "./";
    private const string DefaultName = "name";
    private readonly StringWriter tw;
    private readonly LanguageWriter writer;
    private readonly CodeProperty property;
    private readonly CodeClass parentClass;

    public CodePropertyWriterTests()
    {
        writer = LanguageWriter.GetLanguageWriter(
            GenerationLanguage.YourLanguage, DefaultPath, DefaultName);
        tw = new StringWriter();
        writer.SetTextWriter(tw);

        var root = CodeNamespace.InitRootNamespace();
        parentClass = new CodeClass { Name = "parentClass" };
        root.AddClass(parentClass);
        property = new CodeProperty
        {
            Name = "myProperty",
            Type = new CodeType { Name = "string" }
        };
        parentClass.AddProperty(property);
    }

    [Fact]
    public void WritesCustomProperty()
    {
        property.Kind = CodePropertyKind.Custom;
        writer.Write(property);
        var result = tw.ToString();
        Assert.Contains("string? myProperty", result);
    }

    public void Dispose()
    {
        tw?.Dispose();
        GC.SuppressFinalize(this);
    }
}
```

Refiner tests use an async pattern:

```csharp
public class YourLanguageRefinerTests
{
    private readonly CodeNamespace root = CodeNamespace.InitRootNamespace();

    [Fact]
    public async Task AddsExceptionInheritanceOnErrorClasses()
    {
        var model = root.AddClass(new CodeClass
        {
            Name = "somemodel",
            Kind = CodeClassKind.Model,
            IsErrorDefinition = true,
        }).First();

        await ILanguageRefiner.RefineAsync(
            new GenerationConfiguration { Language = GenerationLanguage.YourLanguage },
            root);

        Assert.Equal("ApiException", model.StartBlock.Inherits.Name);
    }
}
```

### Running tests

Run the full test suite:

```bash
dotnet test tests/Kiota.Builder.Tests/
```

Run only your language's writer tests:

```bash
dotnet test tests/Kiota.Builder.Tests/ --filter "FullyQualifiedName~Writers.YourLanguage"
```

Integration tests in `it/` exercise the full generation-compile-execute pipeline. Refer to `it/generate-code.ps1` and `it/config.json` for how languages are wired into integration testing.

## Companion libraries

Generated Kiota code depends on runtime **companion libraries** that provide the core interfaces and implementations the generated code references. Without these libraries, generated code won't compile. Building these companion libraries is just as important as building the code generator itself.

### URI template dependency

Before implementing abstractions, add support for [std-uritemplate](https://github.com/std-uritemplate/std-uritemplate). Generated Kiota clients rely on URI template expansion to build request URIs before sending HTTP requests.

### Required library types

There are five categories of companion library. The first three are essential for a minimum viable SDK.

1. **Abstractions** &mdash; Core interfaces and base types that generated code compiles against: request adapters, parsable models, authentication interfaces, error types, and query parameter helpers. This is the foundational package.

1. **HTTP** &mdash; A concrete `RequestAdapter` implementation backed by an HTTP client native to the language.

1. **Serialization** &mdash; Format-specific serializers and deserializers. Kiota currently defines four formats: **JSON**, **Form** (URL-encoded), **Text** (plain text), and **Multipart**. At minimum, you need JSON serialization.

1. **Authentication** &mdash; Auth provider implementations that plug into the abstractions' authentication interfaces.

1. **Bundle** *(optional)* &mdash; A convenience meta-package that aggregates all of the above and provides a `DefaultRequestAdapter`.

### Where they live

Companion libraries live in **separate repositories** from the Kiota generator. The general naming convention is `kiota-{type}-{language}`. However, several languages consolidate everything into a monorepo:

- **[microsoft/kiota-dotnet](https://github.com/microsoft/kiota-dotnet)** &mdash; all components for C#
- **[microsoft/kiota-java](https://github.com/microsoft/kiota-java)** &mdash; all components for Java
- **[microsoft/kiota-typescript](https://github.com/microsoft/kiota-typescript)** &mdash; all components for TypeScript
- **[microsoft/kiota-dart](https://github.com/microsoft/kiota-dart)** &mdash; all components for Dart

Languages like Go, PHP, Python, and Ruby keep each component in its own repo (for example, `kiota-abstractions-php`, `kiota-http-guzzle-php`). Prefer the repository layout that follows your language and ecosystem best practices so packaging, dependency management, and contribution workflows feel idiomatic.

### Practical guidance

- **Start with URI template support, then abstractions.** Wire up `std-uritemplate` first so request URIs can be expanded correctly, then define the core interfaces (`RequestAdapter`, `Parsable`, `SerializationWriter`, `ParseNode`) that every other library depends on.
- **Minimum viable set:** abstractions + HTTP + JSON serialization is enough to generate and run a working SDK against most APIs.
- **Develop in parallel.** You can build the generator and companion libraries at the same time.
- **Add formats incrementally.** Once JSON works, add Form, Text, and Multipart serialization.
- **Choose idiomatic dependencies.** Wrap an HTTP client and auth library that feels natural to developers in your target ecosystem.
- **Consult the supported-languages table** in the [Kiota README](https://github.com/microsoft/kiota#supported-languages) to see which components exist today.

## Implementation checklist

Use this checklist to ensure nothing is missed.

### Enum and configuration

- [ ] Add enum value to `src/Kiota.Builder/GenerationLanguage.cs`
- [ ] Add language configuration block to `src/kiota/appsettings.json`

### Code writers

Create `src/Kiota.Builder/Writers/{Language}/` with:

- [ ] `{Language}Writer.cs`
- [ ] `{Language}ConventionService.cs`
- [ ] `CodeMethodWriter.cs`
- [ ] `CodePropertyWriter.cs`
- [ ] `CodeClassDeclarationWriter.cs`
- [ ] `CodeEnumWriter.cs`
- [ ] `CodeBlockEndWriter.cs`

### Refiners

- [ ] `src/Kiota.Builder/Refiners/{Language}Refiner.cs`
- [ ] `src/Kiota.Builder/Refiners/{Language}ReservedNamesProvider.cs`
- [ ] `src/Kiota.Builder/Refiners/{Language}ExceptionsReservedNamesProvider.cs`

### Path segmenter

- [ ] `src/Kiota.Builder/PathSegmenters/{Language}PathSegmenter.cs`

### Factory registrations

- [ ] `src/Kiota.Builder/Writers/LanguageWriter.cs` &mdash; `GetLanguageWriter()` factory
- [ ] `src/Kiota.Builder/Refiners/ILanguageRefiner.cs` &mdash; `RefineAsync()` switch statement
- [ ] `src/Kiota.Builder/Export/PublicAPIExportService.cs` &mdash; convention service mapping

### Tests

- [ ] Writer tests in `tests/Kiota.Builder.Tests/Writers/{Language}/`
- [ ] Refiner tests in `tests/Kiota.Builder.Tests/Refiners/{Language}LanguageRefinerTests.cs`

### Companion libraries

- [ ] **Abstractions** &mdash; core interfaces
- [ ] **HTTP** &mdash; HTTP client implementation
- [ ] **Serialization** &mdash; JSON, Form, Text, and Multipart
- [ ] **Authentication** &mdash; at minimum, Anonymous and API Key providers
- [ ] **Bundle** &mdash; convenience meta-package with a `DefaultRequestAdapter`

### Documentation

- [ ] Add row to the supported languages table in the Kiota `README.md`

## Maturity and graduation

Every language in Kiota follows a defined lifecycle tracked by the `LanguageMaturityLevel` enum in `src/Kiota.Builder/LanguageInformation.cs`. The maturity level and support experience are set in `src/kiota/appsettings.json` and surfaced to users via the CLI and IDE extension.

### Maturity levels

| Level | Description | Expectations |
|---|---|---|
| **Experimental** | Initial implementation; breaking changes are expected. Community contributions start here. | Core code generation works for common patterns. Basic writer and refiner tests exist. At least abstractions and one serialization library are available. |
| **Preview** | Approaching stability; ready for broader testing. | Full OpenAPI feature coverage. All companion libraries published. Integration tests pass. Documentation available. |
| **Stable** | Production-ready; backward compatibility guaranteed. | Comprehensive test coverage. Semantic versioning. Active maintenance commitment. |
| **Abandoned** | No longer maintained. | Retained for existing users but receives no updates. |

### Support experience

The `SupportExperience` enum distinguishes who maintains the language:

- **Microsoft** &mdash; Officially maintained by the Kiota team at Microsoft.
- **Community** &mdash; Maintained by external contributors.

New community languages should set `"SupportExperience": "Community"` and `"MaturityLevel": "Experimental"` in `appsettings.json`. Graduation from Experimental to Preview (or from Community to Microsoft) requires a proposal reviewed by the Kiota maintainers that demonstrates adequate test coverage, companion library completeness, and a maintenance commitment.
