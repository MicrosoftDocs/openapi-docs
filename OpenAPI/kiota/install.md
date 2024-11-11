---
title: Install Kiota
description: Learn how to install Kiota to build API clients.
author: jasonjoh
ms.author: jasonjoh
ms.topic: get-started
ms.date: 03/10/2023
---

# Install Kiota

Kiota can be accessed in the following ways.

- [Download binaries](#download-binaries)
- [Run in Docker](#run-in-docker)
- [Install as .NET tool](#install-as-net-tool)
- [Build from source](#build-from-source)
- [Install the Visual Studio Code extension (preview)](#install-the-visual-studio-code-extension)
- [Run in GitHub Actions (preview)](#run-in-github-actions)
- [Install with asdf (UNOFFICIAL)](#install-with-asdf)
- [Install with Homebrew (UNOFFICIAL)](#install-with-homebrew)
- [REST API Client Code Generator extension for Visual Studio (UNOFFICIAL)](#rest-api-client-code-generator)

## Download binaries

You can download the latest version for your operating system.

| Operating system | Download                                                                                   |
|------------------|--------------------------------------------------------------------------------------------|
| Linux (x64)      | [linux-x64.zip](https://aka.ms/get/kiota/latest/linux-x64.zip) |
| macOS (arm64)    | [osx-arm64.zip](https://aka.ms/get/kiota/latest/osx-arm64.zip) |
| macOS (x64)      | [osx-x64.zip](https://aka.ms/get/kiota/latest/osx-x64.zip)     |
| Windows (x64)    | [win-x64.zip](https://aka.ms/get/kiota/latest/win-x64.zip)     |
| Windows (x86)    | [win-x86.zip](https://aka.ms/get/kiota/latest/win-x86.zip)     |

[All releases](https://github.com/microsoft/kiota/releases/latest)

## Run in Docker

You can run Kiota in our Docker container with one of the following commands.

<!-- markdownlint-disable MD051 -->
### [bash](#tab/bash)

```bash
docker run -v /some/output/path:/app/output \
-v /some/input/description.yml:/app/openapi.yaml \
mcr.microsoft.com/openapi/kiota generate --language csharp -n namespace-prefix
```

### [PowerShell](#tab/powershell)

```powershell
docker run -v /some/output/path:/app/output `
-v /some/input/description.yml:/app/openapi.yaml `
mcr.microsoft.com/openapi/kiota generate --language csharp -n namespace-prefix
```

<!-- markdownlint-enable MD051 -->

---

> [!TIP]
> You can alternatively use the `--openapi` parameter with a URI instead of volume mapping.

To generate a client library from an online OpenAPI description and into the current directory:

<!-- markdownlint-disable MD024 MD051 -->
### [bash](#tab/bash)

```bash
docker run -v ${PWD}:/app/output mcr.microsoft.com/openapi/kiota \
generate --language typescript -n gfx -d \
https://raw.githubusercontent.com/microsoftgraph/msgraph-sdk-powershell/dev/openApiDocs/v1.0/Mail.yml
```

### [PowerShell](#tab/powershell)

```powershell
docker run -v ${PWD}:/app/output mcr.microsoft.com/openapi/kiota `
generate --language typescript -n gfx -d `
https://raw.githubusercontent.com/microsoftgraph/msgraph-sdk-powershell/dev/openApiDocs/v1.0/Mail.yml
```

<!-- markdownlint-disable MD024 MD051 -->

---

## Install as .NET tool

If you have the [.NET SDK](https://dotnet.microsoft.com/download) installed, you can install Kiota as a [.NET tool](/dotnet/core/tools/global-tools).

To install the tool, execute the following command.

```bash
dotnet tool install --global Microsoft.OpenApi.Kiota
```

## Build from source

1. Clone the [Kiota repository](https://github.com/microsoft/kiota).
1. Install the [.NET SDK 8.0](https://get.dot.net/8).
1. Open the solution with Visual Studio and right select *publish* **or** execute the following command:

    ```bash
    dotnet publish ./src/kiota/kiota.csproj -c Release -p:PublishSingleFile=true -r <runtime-identifiers>
    ```

    | Runtime identifier | Value     |
    |--------------------|-----------|
    | Linux (x64)        | linux-x64 |
    | macOS (arm64)      | osx-arm64 |
    | macOS (x64)        | osx-x64   |
    | Windows (x64)      | win-x64   |
    | Windows (x86)      | win-x86   |

    > [!NOTE]
    > Refer to [.NET runtime identifier catalog](/dotnet/core/rid-catalog) so select the appropriate runtime for your platform.

    For example, the following command builds for Windows (x64).

     ```bash
    dotnet publish ./src/kiota/kiota.csproj -c Release -p:PublishSingleFile=true -r win-x64
    ```

1. Navigate to the output directory (usually under `src/kiota/bin/Release/net7.0`).
1. Run `kiota.exe ...`.

## Install the Visual Studio Code extension

1. Open the [Marketplace page of the extension](https://aka.ms/kiota/extension)
1. Select on the **Install** button.

> [!NOTE]
> The Kiota Visual Studio Code extension is currently in public preview and is subject to change.

## Run in GitHub actions

You can use the [setup Kiota GitHub Action](https://github.com/microsoft/setup-kiota/) in your workflow to automate client refresh and more by adding a reference to it in your workflow definition and using the CLI just like you would locally.

```yaml
steps:
  - uses: actions/checkout@v3

  - uses: microsoft/setup-kiota@v0.5.0

  - name: Update kiota clients in the repository
    run: kiota update -o . # for a complete documentation of the CLI commands see https://aka.ms/kiota/docs
    working-directory: src # assumes client is under the src path in your repository
```

> [!NOTE]
> The setup Kiota GitHub Action in public preview and is subject to change.

## Install with asdf

> [!IMPORTANT]
> The asdf Kiota plugin is maintained and distributed by the community and is not an official Microsoft plugin. Microsoft makes no warranties, express or implied, with respect to the plugin or its use. Use of this plugin is at your own risk. Microsoft shall not be liable for any damages arising out of or in connection with the use of this plugin.

The community made Kiota available as an [asdf plugin](https://asdf-vm.com/manage/plugins.html). To install the `asdf-kiota` plugin, follow these instructions:

```bash
asdf plugin add kiota
# or
asdf plugin add kiota https://github.com/asdf-community/asdf-kiota.git

# Show all installable versions
asdf list-all kiota

# Install specific version
asdf install kiota latest

# Set a version globally (on your ~/.tool-versions file)
asdf global kiota latest

# Now kiota commands are available
kiota --version
```

## Install with Homebrew

> [!IMPORTANT]
> The Homebrew formula for Kiota is maintained and distributed by the community and is not an official Microsoft plugin. Microsoft makes no warranties, express or implied, with respect to the plugin or its use. Use of this plugin is at your own risk. Microsoft shall not be liable for any damages arising out of or in connection with the use of this plugin.

The community made Kiota available as a [Homebrew formula](https://formulae.brew.sh/formula/kiota) for macOS running on x64 and arm64 architectures. To install, follow these instructions:

```bash
brew install kiota
```

## REST API Client Code Generator

> [!IMPORTANT]
> The REST API Client Code Generator extension for Visual Studio is maintained and distributed by the community and is not an official Microsoft Visual Studio extension. Microsoft makes no warranties, express or implied, with respect to the extension or its use. Use of this extension is at your own risk. Microsoft shall not be liable for any damages arising out of or in connection with the use of this extension.

[REST API Client Code Generator](https://github.com/christianhelle/apiclientcodegen) is a collection of Visual Studio C# custom tool code generators for OpenAPI specifications. This extension installs Kiota on-demand and adds the required NuGet packages to build the generated code to the project. The generated code is created as a "code-behind" file to the OpenAPI specifications file in the .NET project. This extension offers same-day releases for new Kiota versions, but this requires updating the extension, which can be configured to be automatically.

### Installation

1. From Visual Studio (Windows), using the [Manage Extensions](/visualstudio/ide/finding-and-using-visual-studio-extensions) dialog box (**Tools** -> **Manage Extensions**), search for extension  called **REST API Client Code Generator**. This extension is available for [Visual Studio 2022 for AMD64 and ARM64](https://marketplace.visualstudio.com/items?itemName=ChristianResmaHelle.ApiClientCodeGenerator2022) and [Visual Studio 2019](https://marketplace.visualstudio.com/items?itemName=ChristianResmaHelle.ApiClientCodeGenerator)

    > [!NOTE]
    > For Visual Studio for Mac, follow [these instructions](https://github.com/christianhelle/apiclientcodegen/blob/master/docs/VisualStudioForMac.md#installation)

1. Select on the **Download** button. To complete the installation, restart Visual Studio.

1. Follow [these instructions](https://github.com/christianhelle/apiclientcodegen/blob/master/docs/KiotaUsage.md) for generating code.

## Next steps

For details on running Kiota, see [Using the Kiota tool](using.md).
