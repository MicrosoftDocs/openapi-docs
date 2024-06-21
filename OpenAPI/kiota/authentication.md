---
title: Authentication with Kiota API clients
description: Learn how to authenticate requests made by Kiota API clients.
author: baywet
ms.author: vibiret
ms.topic: conceptual
date: 03/21/2023
---

# Authentication with Kiota API clients

Most REST APIs are protected through some kind of authentication and authorization scheme. The default HTTP core services provided by Kiota require an authentication provider to be passed to handle authentication concerns.

## Authentication provider interface

Authentication providers are required to implement the following contract/interface defined in the abstraction library.

<!-- markdownlint-disable MD051 -->
### [.NET](#tab/csharp)

```csharp
public interface IAuthenticationProvider
{
    Task AuthenticateRequestAsync(
        RequestInformation request,
        Dictionary<string, object>? additionalAuthenticationContext = null,
        CancellationToken cancellationToken = default);
}
```

The `request` parameter is the abstract request to be executed. The return value must be a `Task` that completes whenever the authentication sequence completes and the request object is updated with the authentication/authorization information.

### [Go](#tab/go)

```go
type AuthenticationProvider interface {
    // AuthenticateRequest authenticates the provided RequestInformation.
    AuthenticateRequest(context context.Context, request *abs.RequestInformation, additionalAuthenticationContext map[string]interface{}) error
}
```

The `request` parameter is the abstract request to be executed. The method should return whenever the authentication sequence completes and the request object is updated with the authentication/authorization information.

### [Java](#tab/java)

```java
public interface AuthenticationProvider {
    @Nonnull
    CompletableFuture<Void> authenticateRequest(@Nonnull final RequestInformation request, @Nullable final Map<String, Object> additionalAuthenticationContext);
}
```

The `request` parameter is the abstract request to be executed. The return value must be a `CompletableFuture` that completes whenever the authentication sequence completes and the request object is updated with the authentication/authorization information.

### [PHP](#tab/php)

```php
interface AuthenticationProvider {
    public function authenticateRequest(RequestInformation $request): Promise;
}
```

The `request` parameter is the abstract request to be executed. The return value must be a `Promise` that completes whenever the authentication sequence completes and the request object is updated with the authentication/authorization information.

### [Python](#tab/python)

```python
class AuthenticationProvider(ABC):

    @abstractmethod
    async def authenticate_request(self, request: RequestInformation) -> None:
        pass
```

The `request` parameter is the abstract request to be executed. The method should return whenever the authentication sequence completes and the request object is updated with the authentication/authorization information.

### [Ruby](#tab/ruby)

```ruby
module AuthenticationProvider
    def authenticate_request(request, additional_properties = {})
          raise NotImplementedError.new
    end
end
```

The `request` parameter is the abstract request to be executed. The method should return whenever the authentication sequence completes and the request object is updated with the authentication/authorization information.

### [TypeScript](#tab/typescript)

```typescript
export interface AuthenticationProvider {
  authenticateRequest: (
    request: RequestInformation,
    additionalAuthenticationContext?: Record<string, unknown>
  ) => Promise<void>;
}
```

The `request` parameter is the abstract request to be executed. The return value must be a `Promise` that completes whenever the authentication sequence completes and the request object is updated with the authentication/authorization information.

<!-- markdownlint-enable MD051 -->

---

## Access token provider interface

Implementations of this interface can be used in combination with the `BaseBearerTokenAuthentication` provider class to provide the access token to the request before it executes. They can also be used to retrieve an access token to build and execute arbitrary requests with a native HTTP client.

<!-- markdownlint-disable MD024 MD051 -->
### [.NET](#tab/csharp)

```csharp
public interface IAccessTokenProvider
{
    AllowedHostsValidator AllowedHostsValidator { get; }

    Task<string> GetAuthorizationTokenAsync(
        Uri uri,
        Dictionary<string, object>? additionalAuthenticationContext = null,
        CancellationToken cancellationToken = default));
}
```

In `GetAuthorizationTokenAsync`, the `uri` parameter is the URI of the abstract request to be executed. The return value is a `Task` that holds the access token, or `null` if the request could/should not be authenticated.

### [Go](#tab/go)

```go
//AccessTokenProvider returns access tokens.
type AccessTokenProvider interface {
    // GetAuthorizationToken returns the access token for the provided url.
    GetAuthorizationToken(context context.Context, url *u.URL, additionalAuthenticationContext map[string]interface{}) (string, error)
    // GetAllowedHostsValidator returns the hosts validator.
    GetAllowedHostsValidator() *AllowedHostsValidator
}
```

In `GetAuthorizationToken`, the `url` parameter is the URL of the abstract request to be executed. The return value is the access token, or `nil` if the request could/should not be authenticated.

### [Java](#tab/java)

```java
public interface AccessTokenProvider {
    @Nonnull
    CompletableFuture<String> getAuthorizationToken(@Nonnull final URI uri, @Nullable final Map<String, Object> additionalAuthenticationContext);

    @Nonnull
    AllowedHostsValidator getAllowedHostsValidator();
}
```

In `getAuthorizationToken`, the `uri` parameter is the URI of the abstract request to be executed. The return value is a `CompletableFuture` that holds the access token, or `null` if the request could/should not be authenticated.

### [PHP](#tab/php)

```php
interface AccessTokenProvider
{
    public function getAuthorizationTokenAsync(string $url): Promise;

    public function getAllowedHostsValidator(): AllowedHostsValidator;
}
```

In `getAuthorizationTokenAsync`, the `$url` parameter is the URL of the abstract request to be executed. The return value is a `Promise` that holds the access token, or `null` if the request could/should not be authenticated.

### [Python](#tab/python)

```python
class AccessTokenProvider(ABC):

    @abstractmethod
    async def get_authorization_token(self, uri: str) -> str:
        pass

    @abstractmethod
    def get_allowed_hosts_validator(self) -> AllowedHostsValidator:
        pass
```

In `get_authorization_token`, the `uri` parameter is the URI of the abstract request to be executed. The return value is the access token, or `null` if the request could/should not be authenticated.

### [Ruby](#tab/ruby)

```ruby
module AccessTokenProvider
    def get_authorization_token(uri, additional_properties = {})
        raise NotImplementedError.new
    end
end
```

In `get_authorization_token`, the `uri` parameter is the URI of the abstract request to be executed. The return value is the access token, or `null` if the request could/should not be authenticated.

### [TypeScript](#tab/typescript)

```typescript
export interface AccessTokenProvider {
  getAuthorizationToken: (
    url?: string,
    additionalAuthenticationContext?: Record<string, unknown>
  ) => Promise<string>;

  getAllowedHostsValidator: () => AllowedHostsValidator;
}
```

In `getAuthorizationToken`, the `url` parameter is the URL of the abstract request to be executed. The return value is a `Promise` that holds the access token, or `null` if the request could/should not be authenticated.

<!-- markdownlint-enable MD024 MD051 -->

---

## Allowed hosts validator

This utility class is in charge of validating the host of any request is present on an allowlist. The validator is used by the base access token provider to check whether it should return a token to the request.

## Base bearer token authentication provider

A common practice in the industry for APIs is to implement authentication and authorization via the `Authorization` request header with a bearer token value.

If you want to add support for more authentication providers for that scheme, Kiota abstractions offer a class to use in combination with your own implementation of an access token provider. You only need to implement the access token acquisition sequence and not the header composition and addition.

<!-- markdownlint-disable MD024 MD051 -->
### [.NET](#tab/csharp)

```csharp
public class BaseBearerTokenAuthenticationProvider
{
    public BaseBearerTokenAuthenticationProvider(IAccessTokenProvider tokenProvider) {

    }
}
```

### [Go](#tab/go)

```go
type BaseBearerTokenAuthenticationProvider struct {
    accessTokenProvider AccessTokenProvider
}
```

### [Java](#tab/java)

```java
public class BaseBearerTokenAuthenticationProvider {
    public BaseBearerTokenAuthenticationProvider(@Nonnull final AccessTokenProvider accessTokenProvider) {
        this.accessTokenProvider = Objects.requireNonNull(accessTokenProvider);
    }
}
```

### [PHP](#tab/php)

```php
class BaseBearerTokenAuthenticationProvider {
    public function __construct(AccessTokenProvider $accessTokenProvider) {

    }
}
```

### [Python](#tab/python)

```python
class BaseBearerTokenAuthenticationProvider:

    def __init__(self, access_token_provider: AccessTokenProvider) -> None:
        self.access_token_provider = access_token_provider
```

### [Ruby](#tab/ruby)

```ruby
class BaseBearerTokenAuthenticationProvider
    def initialize(access_token_provider)
    end
end
```

### [TypeScript](#tab/typescript)

```typescript
export class BaseBearerTokenAuthenticationProvider {
    public constructor (
        public readonly accessTokenProvider: AccessTokenProvider
    ) {}
}
```

<!-- markdownlint-enable MD024 MD051 -->

---

> [!NOTE]
> Please leverage the same approach if you want to add support for new authentication schemes where the authentication scheme composition logic is implemented in a base class so it can be reused across multiple providers.

## Microsoft Entra Identity authentication provider

The Azure authentication package contains an authentication provider that relies on Microsoft Entra Identity to get access tokens and implements bearer authentication. It can effectively be used for any client making requests to APIs secured by the Microsoft Entra Identity Platform.

## Anonymous authentication provider

Some APIs don't require any authentication and can be queried anonymously. For this scenario, the abstractions packages provide an `AnonymousAuthenticationProvider`, which serves as a placeholder and performs no operation.

## API key authentication provider

Some APIs simply rely on an API key for authentication in the request query parameters or in the request headers. For this scenario, the abstractions packages provide an `ApiKeyAuthenticationProvider`.

This provider allows you to:

- Set the name of the request header/query parameter. (for example `Authorization`)
- Set the value of the request header/query parameter. (for example `Basic base64encodedValue`)
- Choose whether the provided name and value are used for a request header or for a query parameter.

> [!NOTE]
> This authentication provider does not perform any encoding of the key or the value, if the authentication scheme requires any encoding, it needs to be performed before the values are passed to the provider. For example with basic authentication, you'll need to base64 encode the "userId:password" pair before passing it to the provider.

## Choose your authentication provider

- Does the target API require no authentication? Use the anonymous authentication provider.
- Is the API protected by Microsoft Identity Platform? Use the Azure Identity authentication provider.
- Is the authentication implemented via an API key in the query parameters or headers? Use the API key authentication provider.
- Is the authentication implemented via a bearer token in the `Authorization` header? Use a custom access token provider with the Base bearer token authentication provider.
- Anything else, use a custom authentication provider.

## Design considerations

The default request adapters used by generated clients to make an HTTP request require an authentication provider as a constructor parameter, and call the **authenticate request** method themselves. An alternative could be to use [HTTP client middleware handlers](./middleware.md) instead. This approach was a deliberate design decision to:

- Lead application developer to make a conscious choice about authenticating with the API.
- Enable reusability through interfaces defined to work with Request Information, limiting interdependencies.
- Enable dependency injection scenarios.
