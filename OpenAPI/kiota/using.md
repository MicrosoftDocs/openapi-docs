---
title: Using the Kiota tool
description: Learn about the commands and options available for the Kiota CLI.
author: baywet
ms.author: vibiret
ms.topic: conceptual
date: 03/10/2023
---

# Using the Kiota tool

<!-- markdownlint-disable MD024 -->

> [!NOTE]
> For information on installing Kiota, see [Get started with Kiota](install.md).

## Configuration environment variables

The table below provides the list of environment variables that can be used to configure the experience for the Kiota CLI.

| Environment variable name      | Description                                                         | Default value |
|--------------------------------|---------------------------------------------------------------------|---------------|
| `KIOTA_CONSOLE_COLORS_ENABLED` | Enable/disable output colorization                                  | `true`        |
| `KIOTA_CONSOLE_COLORS_SWAP`    | Enable/disable inverting the color scheme of the output             | `false`       |
| `KIOTA_TUTORIAL_ENABLED`       | Enable/disable displaying additional hints after commands execution | `true`        |
| `KIOTA_OFFLINE_ENABLED`        | Enable/disable checking for updates before commands execution       | `false`       |

## Commands

Kiota offers the following commands to help you build your API client:

- [search](#description-search): search for APIs and their description from various registries.
- [download](#description-download): download an API description.
- [show](#description-show): show the API paths tree for an API description.
- [generate](#client-generation): generate a client for any API from its description.
- [update](#client-update): update existing clients from previous generations.
- [info](#language-information): show languages and runtime dependencies information.
- [login](#login): logs in to private API descriptions repositories.
- [logout](#logout): logs out from private API description repositories.

## Description search

Kiota search accepts the following parameters while searching for APIs and their descriptions.

```bash
kiota search <searchTerm>
      [(--clear-cache | --cc)]
      [(--log-level | --ll) <level>]
      [(--version | -v) <version>]
```

> [!NOTE]
> The search command requires access to internet and cannot be used offline.

### Mandatory arguments

#### Search term

The term to use during the search for APIs and their descriptions.

If multiple results are found, kiota will present a table with the different results.

The following example returns multiple results.

```bash
kiota search github
```

Which will return the following display.

```bash
Key                                  Title               Description            Versions
apisguru::github.com                 GitHub v3 REST API  GitHub's v3 REST API.  1.1.4
apisguru::github.com:api.github.com  GitHub v3 REST API  GitHub's v3 REST API.  1.1.4
apisguru::github.com:ghes-2.18       GitHub v3 REST API  GitHub's v3 REST API.  1.1.4
apisguru::github.com:ghes-2.19       GitHub v3 REST API  GitHub's v3 REST API.  1.1.4
apisguru::github.com:ghes-2.20       GitHub v3 REST API  GitHub's v3 REST API.  1.1.4
apisguru::github.com:ghes-2.21       GitHub v3 REST API  GitHub's v3 REST API.  1.1.4
apisguru::github.com:ghes-2.22       GitHub v3 REST API  GitHub's v3 REST API.  1.1.4
apisguru::github.com:ghes-3.0        GitHub v3 REST API  GitHub's v3 REST API.  1.1.4
apisguru::github.com:ghes-3.1        GitHub v3 REST API  GitHub's v3 REST API.  1.1.4
```

If the search term is an exact match with one of the results' key, the search command will display a detailed view of the result.

```bash
kiota search apisguru::github.com
```

```bash
Key: apisguru::github.com
Title: GitHub v3 REST API
Description: GitHub's v3 REST API.
Service: https://support.github.com/contact
OpenAPI: https://raw.githubusercontent.com/github/rest-api-description/main/descriptions/api.github.com/api.github.com.json
```

### Optional parameters

The search command accepts optional parameters commonly available on the other commands:

- [--clear-cache](#--clear-cache---co)
- [--log-level](#--log-level---ll)
- [--version](#--version--v)

## Description download

Kiota download downloads API descriptions to a local from a registry and accepts the following parameters.

It is not mandatory to download the description locally for the purpose of generation as the generation command can download the description directly. However, having a local copy of the description can be helpful to inspect it, determine which API paths are needed, and come up with the include/exclude filters for generation.

> [!NOTE]
> The download command requires access to internet and cannot be used offline.

```bash
kiota download <searchTerm>
      [--clear-cache | --cc]
      [--clean-output | --co]
      [(--log-level | --ll) <level>]
      [(--version | -v) <version>]
      [(--output | -o) <path>]
```

### Mandatory arguments

#### Search term

The search term to use to locate the description. The description will be downloaded only if an exact key match occurs. You can use the search command to find the key of the API description before downloading it.

```bash
kiota download apisguru::github.com
```

### Optional parameters

The download command accepts optional parameters commonly available on the other commands:

- [--clean-output](#--clean-output---co)
- [--clear-cache](#--clear-cache---co)
- [--log-level](#--log-level---ll)
- [--output](#--output--o)
- [--version](#--version--v)

## Description show

Kiota show accepts the following parameters when displaying the API paths tree for a description.

```bash
kiota show [(--openapi | -d) <path>]
      [(--log-level | --ll) <level>]
      [--clear-cache | --cc]
      [(--version | -v) <version>]
      [(--search-key | -k) <searchKey>]
      [(--max-depth | --m-d) <maximumDepth>]
      [(--include-path | -i) <glob pattern>]
      [(--exclude-path | -e) <glob pattern>]
```

### Example

The following command with the provided options will display the following result.

```bash
kiota show -d https://raw.githubusercontent.com/microsoftgraph/msgraph-sdk-powershell/dev/openApiDocs/v1.0/Mail.yml -i **/messages
```

```bash
/
 └─users
    └─{user-id}
       ├─mailFolders
       │  └─{mailFolder-id}
       │     ├─childFolders
       │     └─messages
       └─messages
```

### Optional Parameters

The show command accepts optional parameters commonly available on the other commands:

- [--openapi](#--openapi--d)
- [--clear-cache](#--clear-cache---co)
- [--log-level](#--log-level---ll)
- [--include-path](#--include-path--i)
- [--exclude-path](#--exclude-path--e)
- [--version](#--version--v)
- [--search-key](#--search-key--k)

## Client generation

Kiota generate accepts the following parameters during the generation.

```bash
kiota generate (--openapi | -d) <path>
      (--language | -l) <language>
      [(--output | -o) <path>]
      [(--class-name | -c) <name>]
      [(--namespace-name | -n) <name>]
      [(--log-level | --ll) <level>]
      [--backing-store | -b]
      [--exclude-backward-compatible | --ebc]
      [--additional-data | --ad]
      [(--serializer | -s) <classes>]
      [(--deserializer | --ds) <classes>]
      [--clean-output | --co]
      [--clear-cache | --cc]
      [(--structured-mime-types | -m) <mime-types>]
      [(--include-path | -i) <glob pattern>]
      [(--exclude-path | -e) <glob pattern>]
      [(--disable-validation-rules | --dvr) <rule name>]
```

> [!NOTE]
> The output directory will also contain a **kiota-lock.json** lock file in addition to client sources. This file contains all the generation parameters for future reference as well as a hash from the description. On subsequent generations, including updates, the generation will be skipped if the description and the parameters have not changed and if clean-output is **false**. The lock file is meant to be committed to the source control with the generated sources.

### Mandatory parameters

- [openapi](#--openapi--d)
- [language](#--language--l)

### Optional parameters

The generate command accepts optional parameters commonly available on the other commands:

- [--clear-cache](#--clear-cache---co)
- [--clean-output](#--clean-output---co)
- [--include-path](#--include-path--i)
- [--exclude-path](#--exclude-path--e)
- [--log-level](#--log-level---ll)
- [--output](#--output--o)

#### `--backing-store (-b)`

Enables backing store for models. Defaults to `false`.

```bash
kiota generate --backing-store
```

### `--exclude-backward-compatible (--ebc)`

Whether to exclude the code generated only for backward compatibility reasons or not. Defaults to `false`.

To maintain compatibility with applications that depends on generated clients, kiota emits additional code marked as obsolete. New clients do not need this additional backward compatible code. The code marked as obsolete will be removed in the next major version of kiota. Use this option to omit emitting the backward compatible code when generating a new client, or when the application using the existing client being refreshed does not depend on backward compatible code.

```bash
kiota generate --exclude-backward-compatible
```

#### `--additional-data (--ad)`

Will include the 'AdditionalData' property for generated models. Defaults to `true`.

```bash
kiota generate --additional-data false
```

#### `--class-name (-c)`

The class name to use for the core client class. Defaults to `ApiClient`.

##### Accepted values

The provided name MUST be a valid class name for the target language.

```bash
kiota generate --class-name MyApiClient
```

#### `--deserializer (--ds)`

The fully qualified class names for deserializers.

These values are used in the client class to initialize the deserialization providers.

Since version 1.11 this parameter also supports a **none** key to generate a client with no deserialization providers in order to enable better portability.

Defaults to the following values.

| Language   | Default deserializers                                                                                                                                                                                                     |
|------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| C#         | `Microsoft.Kiota.Serialization.Json.JsonParseNodeFactory`, `Microsoft.Kiota.Serialization.Json.JsonParseNodeFactory`, `Microsoft.Kiota.Serialization.Text.TextParseNodeFactory`                                           |
| Go         | `github.com/microsoft/kiota-serialization-form-go/FormParseNodeFactory`, `github.com/microsoft/kiota-serialization-json-go/JsonParseNodeFactory`, `github.com/microsoft/kiota-serialization-text-go/TextParseNodeFactory` |
| Java       | `com.microsoft.kiota.serialization.TextParseNodeFactory`, `com.microsoft.kiota.serialization.JsonParseNodeFactory`, `com.microsoft.kiota.serialization.TextParseNodeFactory`                                              |
| PHP | `Microsoft\Kiota\Serialization\Json\JsonParseNodeFactory`, `Microsoft\Kiota\Serialization\Text\TextParseNodeFactory` |
| Python | `kiota_serialization_json.json_parse_node_factory.JsonParseNodeFactory`, `kiota_serialization_text.text_parse_node_factory.TextParseNodeFactory` |
| Ruby       | `microsoft_kiota_serialization/json_parse_node_factory`                                                                                                                                                                   |
| TypeScript | `@microsoft/kiota-serialization-form.FormParseNodeFactory`, `@microsoft/kiota-serialization-json.JsonParseNodeFactory`, `@microsoft/kiota-serialization-text.TextParseNodeFactory`                                        |

##### Accepted values

One or more module names that implements `IParseNodeFactory`.

```bash
kiota generate --deserializer Contoso.Json.CustomDeserializer
```

#### `--disable-validation-rules (--dvr)`

The name of the OpenAPI description validation rule to disable. Or `all` to disable all validation rules including the rules defined in OpenAPI.net.

Kiota runs a set of validation rules before generating the client to ensure the description contains accurate information to build great client experience.

##### Accepted values

Rules:

- DivergentResponseSchema: returns a warning if an operation defines multiple different successful response schemas. (200, 201, 202, 203)
- GetWithBody: returns a warning if a GET request has a body defined.
- InconsistentTypeFormat: returns a warning if an inconsistent type/format pair is defined. (e.g. type: string, format: int32)
- KnownAndNotSupportedFormats: returns a warning if a known formats is not supported for generation. (e.g. email, uri, ...)
- MissingDiscriminator: returns a warning if an anyOf or oneOf schema is used without a discriminator property name.
- MultipleServerEntries: returns a warning if multiple servers are defined.
- NoContentWithBody: returns a warning if a response schema is defined for a 204 response.
- NoServerEntry: returns a warning if no servers are defined.
- UrlFormEncodedComplex: returns a warning if a response of type uri form encoded has a schema that contains complex properties.

```bash
kiota generate --disable-validation-rules NoServerEntry
```

#### `--namespace-name (-n)`

The namespace to use for the core client class specified with the `--class-name` option. Defaults to `ApiSdk`.

##### Accepted values

The provided name MUST be a valid module or namespace name for the target language.

```bash
kiota generate --namespace-name MyAppNamespace.Clients
```

##### Accepted values

A valid URI to an OpenAPI description in the local filesystem or hosted on an HTTPS server.

```bash
kiota generate --openapi https://contoso.com/api/openapi.yml
```

#### `--serializer (-s)`

The fully qualified class names for serializers.

These values are used in the client class to initialize the serialization providers.

Since version 1.11 this parameter also supports a **none** key to generate a client with no serialization providers in order to enable better portability.

Defaults to the following values.

| Language   | Default deserializer                                                                                                                                                                                                                                    |
|------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| C#         | `Microsoft.Kiota.Serialization.Form.FormSerializationWriterFactory`, `Microsoft.Kiota.Serialization.Json.JsonSerializationWriterFactory`, `Microsoft.Kiota.Serialization.Text.TextSerializationWriterFactory`, `Microsoft.Kiota.Serialization.Multipart.MultipartSerializationWriterFactory` |
| Go         | `github.com/microsoft/kiota-serialization-form-go/FormSerializationWriterFactory`, `github.com/microsoft/kiota-serialization-json-go/JsonSerializationWriterFactory`, `github.com/microsoft/kiota-serialization-text-go/TextSerializationWriterFactory`, `github.com/microsoft/kiota-serialization-multipart-go/MultipartSerializationWriterFactory` |
| Java       | `com.microsoft.kiota.serialization.FormSerializationWriterFactory`, `com.microsoft.kiota.serialization.JsonSerializationWriterFactory`, `com.microsoft.kiota.serialization.TextSerializationWriterFactory`, `com.microsoft.kiota.serialization.MultipartSerializationWriterFactory`|
| PHP | `Microsoft\Kiota\Serialization\Json\JsonSerializationWriterFactory`, `Microsoft\Kiota\Serialization\Text\TextSerializationWriterFactory` |
| Python | `kiota_serialization_json.json_serialization_writer_factory.JsonSerializationWriterFactory`, `kiota_serialization_text.text_serialization_writer_factory.TextSerializationWriterFactory` |
| Ruby       | `microsoft_kiota_serialization/json_serialization_writer_factory` |
| TypeScript | `@microsoft/kiota-serialization-form.FormSerializationWriterFactory`, `@microsoft/kiota-serialization-json.JsonSerializationWriterFactory`, `@microsoft/kiota-serialization-text.TextSerializationWriterFactory`, `@microsoft/kiota-serialization-multipart.MultipartSerializationWriterFactory` |

##### Accepted values

One or more module names that implements `ISerializationWriterFactory`.

```bash
kiota generate --serializer Contoso.Json.CustomSerializer
```

#### `--structured-mime-types (-m)`

The MIME types to use for structured data model generation with their preference weight. Any type without a preference will have its preference defaulted to 1. Accepts multiple values. The notation style and the preference weight logic follow the convention [defined in RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html#name-accept).

Default values :

- `application/json;q=1`
- `application/x-www-form-urlencoded;q=0.2`
- `multipart/form-data;q=0.1`
- `text/plain;q=0.9`

> [!NOTE]
> Only request body types or response types with a defined schema will generate models, other entries will default back to stream/byte array.

##### Accepted values

Any valid MIME type which will match a request body type or a response type in the OpenAPI description.

### Examples

```bash
kiota generate --structured-mime-types application/json
```

## Language information

Kiota info accepts the following parameters when displaying language information like their maturity level or dependencies needed at runtime.

```bash
kiota info [(--openapi | -d) <path>]
      [(--language | -l) <language>]
      [(--log-level | --ll) <level>]
      [--clear-cache | --cc]
      [(--version | -v) <version>]
      [(--search-key | -k) <searchKey>]
```

### Example - global

The following command with the provided options will display the following result.

```bash
kiota info
```

```bash
Language    Maturity Level
CLI         Preview
CSharp      Stable
Go          Stable
Java        Preview
PHP         Stable
Python      Stable
Ruby        Experimental
Swift       Experimental
TypeScript  Experimental
```

### Example - with language

The following command with the provided options will display the following result.

```bash
kiota info -l CSharp
```

```bash
The language CSharp is currently in Stable maturity level.
After generating code for this language, you need to install the following packages:
dotnet add package Microsoft.Kiota.Abstractions --version 1.6.1
dotnet add package Microsoft.Kiota.Http.HttpClientLibrary --version 1.3.0
dotnet add package Microsoft.Kiota.Serialization.Form --version 1.1.0
dotnet add package Microsoft.Kiota.Serialization.Json --version 1.1.1
dotnet add package Microsoft.Kiota.Authentication.Azure --version 1.1.0
dotnet add package Microsoft.Kiota.Serialization.Text --version 1.1.0
dotnet add package Microsoft.Kiota.Serialization.Multipart --version 1.1.0
```

Using the `--json` optional parameter render the output in a machine parsable format:

```bash
kiota info -l CSharp --json
```

```json
{
  "maturityLevel": "Stable",
  "dependencyInstallCommand": "dotnet add package {0} --version {1}",
  "dependencies": [
    {
      "name": "Microsoft.Kiota.Abstractions",
      "version": "1.6.1"
    },
    {
      "name": "Microsoft.Kiota.Http.HttpClientLibrary",
      "version": "1.2.0"
    },
    {
      "name": "Microsoft.Kiota.Serialization.Form",
      "version": "1.1.0"
    },
    {
      "name": "Microsoft.Kiota.Serialization.Json",
      "version": "1.1.1"
    },
    {
      "name": "Microsoft.Kiota.Authentication.Azure",
      "version": "1.1.0"
    },
    {
      "name": "Microsoft.Kiota.Serialization.Text",
      "version": "1.1.0"
    },
    {
      "name": "Microsoft.Kiota.Serialization.Multipart",
      "version": "1.1.0"
    }
  ],
  "clientClassName": "",
  "clientNamespaceName": ""
}
```

### Optional Parameters

The info command accepts optional parameters commonly available on the other commands:

- [--openapi](#--openapi--d)
- [--clear-cache](#--clear-cache---co)
- [--log-level](#--log-level---ll)
- [--language](#--language--l)
- [--version](#--version--v)
- [--search-key](#--search-key--k)

## Client update

Kiota update accepts the following parameters during the update of existing clients. This command will search for lock files in the output directory and all its subdirectories and trigger generations to refresh the existing clients using settings from the lock files.

```bash
kiota update [(--output | -o) <path>]
      [(--log-level | --ll) <level>]
      [--clean-output | --co]
      [--clear-cache | --cc]
```

### Optional parameters

The generate command accepts optional parameters commonly available on the other commands:

- [--clear-cache](#--clear-cache---co)
- [--clean-output](#--clean-output---co)
- [--log-level](#--log-level---ll)
- [--output](#--output--o)

## Login

Use kiota login to sign in to private repositories and search for/display/generate clients for private API descriptions. This command makes sub-commands available to sign in to specific authentication providers.

### Login to GitHub

```bash
kiota login github <device|pat>
      [(--log-level | --ll) <level>]
      [(--pat) <patValue>]
```

Use `kiota login github device` to sign in using a device code, you will be prompted to sign-in using a web browser.

Use `kiota login github pat --pat patValue` to sign in using a Personal Access Token you previously generated. You can use both classic PATs or granular PATs (beta). Classic PATs need the **repo** permission and to be granted access to the target organizations. Granular PATs need a **read-only** permission for the **contents** scope under the **repository** category and they need to be consented for all private repositories or selected private repositories.

> [!NOTE]
> For more information about adding API descriptions to the GitHub index, see [Adding an API to search](add-api.md).

### Optional parameters

The generate command accepts optional parameters commonly available on the other commands:

- [--log-level](#--log-level---ll)

## Logout

Use kiota logout to logout from a private repository containing API descriptions.

```bash
kiota logout github
      [(--log-level | --ll) <level>]
```

### Optional parameters

The generate command accepts optional parameters commonly available on the other commands:

- [--log-level](#--log-level---ll)

## Common parameters

The following parameters are available across multiple commands.

### `--clean-output (--co)`

Delete the output directory before generating the client. Defaults to false.

Available for commands: download, generate.

```bash
kiota <command name> --clean-output
```

### `--clear-cache (--co)`

Clears the currently cached file for the command. Defaults to false.

Cached files are stored under `%TEMP%/kiota/cache` and valid for one (1) hour after the initial download. Kiota caches API descriptions during generation and static index files during search.

```bash
kiota <command name> --clear-cache
```

### `--exclude-path (-e)`

A glob pattern to exclude paths from generation. Accepts multiple values. Defaults to no value which excludes nothing.

```bash
kiota <command name> --exclude-path **/users/** --exclude-path **/groups/**
```

You can also filter specific HTTP methods by appending `#METHOD` to the pattern, replacing `METHOD` with the HTTP method to filter. For example, `**/users/**#GET`.

> [!TIP]
> An exclude pattern can be used in combination with the include pattern argument. A path item is included when (no include pattern is included OR it matches an include pattern) AND (no exclude pattern is included OR it doesn't match an exclude pattern).

### `--include-path (-i)`

A glob pattern to include paths from generation. Accepts multiple values. Defaults to no value which includes everything.

```bash
kiota <command name> --include-path **/users/** --include-path **/groups/**
```

You can also filter specific HTTP methods by appending `#METHOD` to the pattern, replacing `METHOD` with the HTTP method to filter. For example, `**/users/**#GET`.

> [!TIP]
> An include pattern can be used in combination with the exclude pattern argument. A path item is included when (no include pattern is included OR it matches an include pattern) AND (no exclude pattern is included OR it doesn't match an exclude pattern).

### `--openapi (-d)`

The location of the OpenAPI description in JSON or YAML format to use to generate the SDK. Accepts a URL or a local path.

Available for commands: info, show, generate.

```bash
kiota <command name> --openapi https://description.url/description.yml
```

### `--language (-l)`

The target language for the generated code files or for the information.

Available for commands: info, generate.

#### Accepted values

- `csharp`
- `go`
- `java`
- `php`
- `python`
- `ruby`
- `shell`
- `swift`
- `typescript`

```bash
kiota <command name> --language java
```

### `--log-level (--ll)`

The log level to use when logging events to the main output. Defaults to `warning`.

Available for commands: all.

#### Accepted values

- `critical`
- `debug`
- `error`
- `information`
- `none`
- `trace`
- `warning`

```bash
kiota <command name> --log-level information
```

### `--output (-o)`

The output directory or file path for the generated code files. Defaults to `./output` for generate and `./output/result.json` for download.

Available for commands: download, generate.

#### Accepted values

A valid path to a directory (generate) or a file (download).

```bash
kiota <command name> --output ./src/client
```

### `--search-key (-k)`

The search key to use to fetch the Open API description, can be used in combination with the version option. Should not be used in combination with the openapi option. Default empty.

Available for commands: info, show.

```bash
kiota <command name> --search-key apisguru::github.com:api.github.com
```

### `--version (-v)`

Select a specific version of the API description. No default value.

Available for commands: download, search.

```bash
kiota <command name> --version beta
```
