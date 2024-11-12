---
author: jasonjoh
ms.author: jasonjoh
ms.topic: include
ms.date: 03/20/2023
---

<!-- markdownlint-disable MD041 -->

Use the following steps to get an authorization code.

> [!IMPORTANT]
> Authorization codes are short-lived, typically expiring after 10 minutes. You should get a new authorization code just before running the example.

1. Open your browser and paste in the following URL, replacing `YOUR_CLIENT_ID` with the client ID of your app registration.

    ```http
    https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=YOUR_CLIENT_ID&response_type=code&redirect_uri=http%3A%2F%2Flocalhost&response_mode=query&scope=User.Read
    ```

1. Sign in with your Microsoft account. Review the requested permissions and grant consent to continue.

1. Once authentication completes, your browser will redirect to `http://localhost`. The browser displays an error that can be safely ignored.

1. Copy the URL from your browser's address bar.

1. Copy everything in the URL between `code=` and `&`. This value is the authorization code.
