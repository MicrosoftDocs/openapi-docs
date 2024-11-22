---
title: Releases and support for Kiota
description: Learn about versioning and support options for Kiota.
author: jasonjoh
ms.author: jasonjoh
ms.topic: conceptual
ms.date: 03/10/2023
---

# Releases and support for Kiota

Microsoft ships major releases, minor releases, and patches for Kiota. This article explains release types and support options for Kiota.

## Release types

Kiota follows [Semantic Versioning 2.0.0](https://semver.org/). Information about the type of each release is encoded in the version number in the form `major.minor.patch`.

### Major releases

Major releases include new features, new public API surface area, and bug fixes. Due to the nature of the changes, these releases are expected to have breaking changes. Major releases can install side by side with previous major releases. We only maintain the latest major release, except for security and privacy related issues where we might release patches to prior major versions. Major releases are published as needed.

### Minor releases

Minor releases also include new features, public API surface area, and bug fixes. The difference between these and major releases is that minor releases don't contain breaking changes to the tooling experience or the generated API surface area of Stable maturity languages. Minor releases can install side by side with previous minor releases. Minor releases are targeted to publish on the first Tuesday of every month.

### Patches

Patches ship as needed, and they include security and nonsecurity bug fixes.

## Language support

### Maturity level

Language support in Kiota is either stable, preview or experimental for each language.

The following criteria are used to determine the maturity levels.

- **Stable**: Kiota provides full functionality for the language and is used to generate production API clients.
- **Preview**: Kiota is at or near the level of stable but isn't being used to generate production API clients.
- **Experimental**: Kiota provides some functionality for the language but is still in early stages. Some features might not work correctly or at all.
- **Abandoned**: The language is not maintained anymore and will be removed in a future major release, users should start migrating off generated client for the language.

Breaking changes to languages that are stable result in major version change for Kiota tooling. Code generation changes to a stable language aren't considered breaking when they rely on additions in the corresponding abstractions library. These changes only require updating the abstractions library to the latest minor/patch version under the same major version.

### Support experience

Languages can be implemented by different authors, which impacts the support experience users can expect:

- **Microsoft**: The language has been implemented by the Kiota team at Microsoft, users can expect support from the team as any regular product.
- **Community**: The language has been implemented by the broader Kiota community, users can expect help from other community members but should not expect direct support from Microsoft.

### Getting support information for a language

The current status of language support can be queried by using the following command.

```bash
kiota info
```

## Kiota roll-forward and compatibility

Major and minor updates can be installed side by side with previous versions. Installing new versions of the Kiota tooling doesn't affect existing Kiota generated code. To update applications to the latest Kiota generate code, the code must be regenerated with the new Kiota tooling. The app doesn't automatically roll forward to use a newer version of Kiota Tooling.

We recommend rebuilding the app and testing against a newer major or minor version before deploying to production.

## Support

The primary support channel for Kiota is GitHub. You can open GitHub issues in the [Kiota repository](https://github.com/microsoft/kiota) to track bugs and feature requests.
