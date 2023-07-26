---
title: Install Kiota
description: Learn how to install Kiota to build API clients.
author: jasonjoh
ms.author: jasonjoh
ms.topic: get-started
date: 03/10/2023
---

# Install Kiota

Kiota can be accessed in the following ways.

- [Download binaries](#download-binaries)
- [Run in Docker](#run-in-docker)
- [Install as .NET tool](#install-as-net-tool)
- [Build from source](#build-from-source)
- [Install the Visual Studio Code extension (preview)](#install-the-visual-studio-code-extension)
- [Install with asdf (unofficial)](#install-with-asdf)

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

```bash
docker run -v /some/output/path:/app/output \
-v /some/input/description.yml:/app/openapi.yml \
mcr.microsoft.com/openapi/kiota generate --language csharp -n namespace-prefix
```

> [!TIP]
> You can alternatively use the `--openapi` parameter with a URI instead of volume mapping.

To generate a SDK from an online OpenAPI description and into the current directory:

```bash
docker run -v ${PWD}:/app/output mcr.microsoft.com/openapi/kiota \
generate --language typescript -n gfx -d \
https://raw.githubusercontent.com/microsoftgraph/msgraph-sdk-powershell/dev/openApiDocs/v1.0/Mail.yml
```

## Install as .NET tool

If you have the [.NET SDK](https://dotnet.microsoft.com/download) installed, you can install Kiota as a [.NET tool](/dotnet/core/tools/global-tools).

Execute the following command to install the tool.

```bash
dotnet tool install --global Microsoft.OpenApi.Kiota
```

## Build from source

1. Clone the current repository.
1. Install the [.NET SDK 7.0](https://get.dot.net/7).
1. Open the solution with Visual Studio and right click *publish* **--or--** execute the following command:

    ```bash
    dotnet publish ./src/kiota/kiota.csproj -c Release -p:PublishSingleFile=true -r win-x64
    ```

1. Navigate to the output directory (usually under `src/kiota/bin/Release/net7.0`).
1. Run `kiota.exe ...`.

> [!NOTE]
> Refer to [.NET runtime identifier catalog](/dotnet/core/rid-catalog) so select the appropriate runtime for your platform.

## Install the Visual Studio Code extension

1. Open the [Marketplace page of the extension](https://aka.ms/kiota/extension)
1. Click on the **Install** button.

## Next steps

For details on running Kiota, see [Using the Kiota tool](using.md).

## Install with asdf

> [!IMPORTANT]
> The asdf Kiota plugin is maintained and distributed by the community and is not an official Microsoft plugin. Microsoft makes no warranties, express or implied, with respect to the plugin or its use. Use of this plugin is at your own risk. Microsoft shall not be liable for any damages arising out of or in connection with the use of this plugin.

The community has made Kiota available as an [asdf plugin](https://asdf-vm.com/manage/plugins.html). To install the `asdf-kiota` plugin follow [these instructions](https://github.com/asdf-community/asdf-kiota#install).
