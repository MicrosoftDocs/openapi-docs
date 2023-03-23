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
- [Run in browser (experimental)](#run-in-browser)
- [Build from source](#build-from-source)

## Download binaries

You can download the latest version for your operating system.

| Operating system | Download                                                                                   |
|------------------|--------------------------------------------------------------------------------------------|
| Windows          | [win-x64.zip](https://github.com/microsoft/kiota/releases/download/v1.0.1/win-x64.zip)     |
| Linux            | [linux-x64.zip](https://github.com/microsoft/kiota/releases/download/v1.0.1/linux-x64.zip) |
| macOS            | [osx-x64.zip](https://github.com/microsoft/kiota/releases/download/v1.0.1/osx-x64.zip)     |

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
dotnet tool install --global --prerelease Microsoft.OpenApi.Kiota
```

## Run in browser

You can run kiota with a modern web browser by navigating to [app.kiota.dev](https://app.kiota.dev).

> [!NOTE]
> This feature is experimental and performances for large API descriptions might be impacted, should you run into issues, we suggest you revert to using the CLI.

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

## Next steps

For details on running Kiota, see [Using the Kiota tool](using.md).
