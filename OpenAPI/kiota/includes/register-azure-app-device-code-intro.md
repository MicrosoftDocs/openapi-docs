---
author: jasonjoh
ms.author: jasonjoh
ms.topic: include
date: 03/20/2023
---

<!-- markdownlint-disable MD041 -->

To be able to authenticate with the Microsoft identity platform and get an access token for Microsoft Graph, you will need to create an application registration. You can install the [Microsoft Graph PowerShell SDK](https://github.com/microsoftgraph/msgraph-sdk-powershell) and use it to create the app registration, or register the app manually in the Azure Active Directory admin center.

The following instructions register an app and enable [device code flow](/azure/active-directory/develop/v2-oauth2-device-code) for authentication. This is the authentication method used by the guides in this section.
