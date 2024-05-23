---
title: Kiota backing store
description: Learn how Kiota generated models provide flexibility around data storage and events subscriptions.
author: baywet
ms.author: vibiret
ms.topic: conceptual
date: 05/01/2024
---

# Kiota backing store

By default Kiota models are generated to store the data directly within their memory space. While this option is simpler, it limits many scenarios.

The [backing store option](./using.md) (`-b | --backing-store`) allows you to generate different models that rely on a backing store implementation to store their data.

## Backing store features

### Dirty tracking

A common limitation of simple models is round trip update of data send the entire representation of the object, which is wasteful and can have side effects.
Imagine a scenario where your application gets a user, updates their title to send it back to the service. In that scenario, you want your client to automatically track which fields the application updates.

```csharp
var user = await client.Users["john"].GetAsync();
user.Title = "Dr";
await client.Users["john"].PatchAsync(user);
```

Without the backing store enabled, the client sends the entire representation:

```json
{
 "firstName": "John",
 "lastName": "Doe",
 "title": "Dr",
 "email": "john.doe@contoso.com"
}
```

Not only it leads to more data being transferred, but it can potentially result in errors if the service prevents updating the email address field without a specific permission.

With the backing store enabled, the client sends only the changed fields:

```json
{
 "title": "Dr"
}
```

Which leads to less data transfer and reduces the chances to result in an error.

The backing store also allows tracking properties being reset and sending null to the service to do so.

```csharp
var user = await client.Users["john"].GetAsync();
user.Title = null; // we want to reset the title to the default value from the service
await client.Users["john"].PatchAsync(user);
```

Without the backing store would result in:

```json
{
 "firstName": "John",
 "lastName": "Doe",
 "email": "john.doe@contoso.com"
}
```

With the backing store would result in:

```json
{
 "title": null
}
```

### Event subscription

In certain scenarios, you need your application to be notified when data changes happen. For instance, to refresh the user interface or to synchronize the data with another data store.

The backing store also enables these scenarios through a subscription mechanism.

```csharp
// in the section that displays the user profile on screen
var user = await client.Users["john"].GetAsync();
user.BackingStore.Subscribe((key, previousValue, newValue) => {
 if ("title".Equals(key)) {
  // refresh the UI component displaying the title of the user
 }
})

// in the profile form as the user updates their title
// the UI component will automatically refresh
dropDown.OnSelectedItemChange += (selectedItem) => {
 user.Title = selectedItem.Value;
};
await client.Users["john"].PatchAsync(user);
```

## Implementations

Kiota abstractions libraries provide a default implementation of the backing store **InMemoryBackingStore**. The generated client automatically registers it when the client is instantiated. The registration mechanism relies on a singleton defined in the abstractions library and the implementation in use is common for all clients in the application domain.

You can register your own implementation by calling the **EnableBackingStore** on the **RequestAdapter** in use, or by passing it as an argument to the client constructor.
