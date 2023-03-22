---
author: jasonjoh
ms.author: jasonjoh
ms.topic: include
date: 03/20/2023
---

<!-- markdownlint-disable MD041 -->

1. Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com). Login with your Azure account.
1. Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.
1. Select **New registration**. On the **Register an application** page, set the values as follows.

    - Set **Name** to `Kiota Test Client`.
    - Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.
    - Set **Redirect URI** platform to **Web**, and set the value to `http://localhost`.

1. Select **Register**. On the **Overview** page, copy the value of the **Application (client) ID** and save it.
1. Select **Certificates & secrets** under **Manage**. Select the **New client secret** button. Enter a value in **Description** and select one of the options for **Expires** and select **Add**.
1. Copy the **Value** of the new secret before you leave this page. It will never be displayed again. Save the value for later.
