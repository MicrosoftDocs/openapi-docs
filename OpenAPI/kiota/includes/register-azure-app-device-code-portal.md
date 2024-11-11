---
author: jasonjoh
ms.author: jasonjoh
ms.topic: include
ms.date: 03/20/2023
---

<!-- markdownlint-disable MD041 -->

1. Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com). Sign in with your Azure account.
1. Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.
1. Select **New registration**. On the **Register an application** page, set the values as follows.

    - Set **Name** to `Kiota Test Client`.
    - Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.
    - Leave **Redirect URI** blank.

1. Select **Register**. On the **Overview** page, copy the value of the **Application (client) ID** and save it.
1. Select **Authentication** under **Manage**.
1. Locate the **Advanced settings** section. Set the **Allow public client flows** toggle to **Yes**, then select **Save**.
