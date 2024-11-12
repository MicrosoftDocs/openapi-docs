---
author: jasonjoh
ms.author: jasonjoh
ms.topic: include
ms.date: 03/20/2023
---

<!-- markdownlint-disable MD041 -->

> [!NOTE]
> The PowerShell script requires an account with the Application administrator, Cloud application administrator, or Global administrator role. If your account has the Application developer role, you can register an app in the Azure Active Directory admin center.

```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All"
$app = New-MgApplication -displayName "Kiota Test Client" -IsFallbackPublicClient
```

Save the value of the `AppId` property of the `$app` object.

```powershell
> $app.AppId
1cddd83e-eda6-4c65-bccf-920a86f220ab
```
