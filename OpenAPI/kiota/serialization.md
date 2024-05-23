---
title: Serialization with Kiota API clients
description: Learn about the interfaces used for serialization and deserialization in Kiota clients.
author: baywet
ms.author: vibiret
ms.topic: conceptual
date: 03/21/2023
---

# Serialization with Kiota API clients

APIs rely on some serialization format (JSON, YAML, XML...) to be able to receive and respond to requests from their clients. Kiota generated models and request builders aren't tied to any specific serialization format or implementation library. To achieve this, Kiota generated clients rely on two key concepts:

- Models rely on a set of abstractions available in the abstractions package and implemented in a separate package for each format.
- Models self-describe their serialization/deserialization logic. For more information about models, see [Kiota API models](./models.md).

## Additional data holder

The additional data holder interface defines members implemented by models that might include properties in API responses that aren't described in the OpenAPI description of the API.

```csharp
public interface IAdditionalDataHolder
{
    IDictionary<string, object> AdditionalData { get; set; }
}

```

## Parsable

The parsable interface defines members that are required in models in order to be able to self serialize/deserialize itself. You can find a detailed description of those members in the [models](./models.md) documentation page.

```csharp
public interface IParsable
{
    IDictionary<string, Action<T, IParseNode>> GetFieldDeserializers<T>();
    void Serialize(ISerializationWriter writer);
    IDictionary<string, object> AdditionalData { get; set; }
}
```

## Parsable factory

Parsable factory defines the structure for models factories creating parsable objects according to their derived types or the discriminator value present in the payload.

```CSharp
public delegate T ParsableFactory<T>(IParseNode node) where T : IParsable;
```

## Parse node

The parse node interface defines members that are required in a deserialization library. It mostly acts as an abstractions and mapping layer between the methods that models' deserializers call, and the library in use to deserialize the payload. It heavily relies on recurrence programming design, and the corresponding factory instantiates the implementing class.

On top of the mapping methods, parse node offers two events that must be called during the deserialization process by implementers:

- `OnBeforeAssignFieldValues`: when the target object is created and before fields of the object are assigned.
- `OnAfterAssignFieldValues`: when the target object field values are assigned.

## Parse node factory

The parse node factory interface defines members that are required in a deserialization library to instantiate a new parse node implementer from a raw payload. It also describes which MIME type this factory/parse node applies to.

Whenever the request adapter needs to deserialize a response body, it looks for a factory that applies to that result based on the HTTP response `Content-Type` header, instantiate a new parse node for the corresponding type using the factory, and call in the parse node to get the deserialized result.

## Parse node factory registry

This class is a registry for multiple parse node factories that are available to the request adapter. Implementers for new deserialization libraries or formats don't need to do anything with this type other than requesting users to register their new factory at runtime with the provided singleton to make it available for the Kiota client.

## Parse node proxy factory

Parse node offers multiple events that are called during the deserialization sequence. This proxy facilitates the registration of such events with existing parse nodes. Implementers for new deserialization libraries or formats don't need to do anything with this type other than requesting users to wrap their factories with it as well should they be subscribing to deserialization events.

## Serialization writer

The serialization writer interface defines members that are required in a serialization library. It mostly acts as an abstractions and mapping layer between the methods that models' serializers call, and the library in use to serialize the payload. The corresponding factory instantiates the implementing class.

On top of the mapping methods, serialization writer offers three events that must be called during the serialization process by implementers:

- `OnBeforeObjectSerialization`: called before the serialization process starts.
- `OnStartObjectSerialization`: called right after the serialization process starts.
- `OnAfterObjectSerialization`: called after the serialization process ends.

## Serialization writer factory

The serialization writer factory interface defines members that are required in a serialization library to instantiate a new serialization writer implementer. It also describes which MIME type this factory/serialization writer applies to.

Whenever the request adapter needs to serialize a request body, it looks for a factory that applies to that result based on the HTTP request `Content-Type` header, instantiate a serialization writer for the corresponding type using the factory, and call in the serialization writer to get the serialized result.

## Serialization writer factory registry

This class is a registry for multiple serialization writer factories that are available to the request adapter. Implementers for new serialization libraries or formats don't need to do anything with this type other than requesting users to register their new factory at runtime with the provided singleton to make is available for the Kiota client.

## Serialization writer proxy factory

Serialization writer offers multiple events that are called during the serialization sequence. This proxy facilitates the registration of such events with existing serialization writers. Implementers for new serialization libraries or formats don't need to do anything with this type other than requesting users to wrap their factories with it as well should they be subscribing to serialization events.

## Serialization helpers

Automatic serialization and the decoupling between the serialization information and the serialization format can make some serialization scenarios more complex. For that reason, Kiota abstractions libraries offer serialization helpers to unblock such scenarios.

It's important to use the Kiota serialization infrastructure and not the native serialization capabilities from a library as:

- it supports down-casting to a derived type from discriminator. (for example, getting a student with extra fields and not just a user)
- it supports complex property types. (duration, dates, etc.)
- it doesn't rely on any runtime reflection of the code.

Let's assume the API contains a **User** model.

<!-- markdownlint-disable-next-line MD051 -->
### [C#](#tab/csharp)

```cs

using Microsoft.Kiota.Abstractions.Serialization;

var myUser = new User {
    FirstName = "Jane",
    LastName = "Smith"
};

var result = await KiotaJsonSerializer.SerializeAsStringAsync(myUser);
// gives us {"firstName":"Jane","lastName":"Smith"}
// alternatively if you need to serialize to a different content type you can use this helper instead
// var result = await KiotaSerializer.SerializeAsStringAsync("application/json", myUser);

var deserializedUser = await KiotaJsonSerializer.DeserializeAsync(result, User.CreateFromDiscriminatorValue);
// returns the deserialized value
```

<!-- markdownlint-disable-next-line MD051 -->
### [Java](#tab/java)

```java
import com.microsoft.kiota.serialization;

final User myUser = new User();
myUser.firstName = "Jane";
myUser.lastName = "Smith";

final String result = KiotaJsonSerialization.serializeAsString(myUser);
// gives us {"firstName":"Jane","lastName":"Smith"}
// alternatively if you need to serialize to a different content type you can use this helper instead
// final String result = KiotaSerialization.serializeAsString("application/json", myUser);

final User deserializedUser = KiotaJsonSerialization.deserialize(result, User::createFromDiscriminatorValue);
```

<!-- markdownlint-disable-next-line MD051 -->
### [Go](#tab/go)

```go
import (
    kiotaSerialization "github.com/microsoft/kiota-abstractions-go/serialization"
)

myUser := NewUser()
firstName := "Jane"
myUser.SetFirstName(&firstName)
lastName := "Smith"
myUser.SetLastName(&lastName)

result, err := kiotaSerialization.SerializeToJson(myUser);
// gives us {"firstName":"Jane","lastName":"Smith"}
// alternatively if you need to serialize to a different content type you can use this helper instead
// result, err := kiotaSerialization.Serialize("application/json", myUser);

deserializedUser, err := kiotaSerialization.DeserializeFromJson(result, CreateUserFromDiscriminatorValue)
```

<!-- markdownlint-disable-next-line MD051 -->
### [TypeScript](#tab/typescript)

```TS
import {serializeToJson, deserializeFromJson } from "@microsoft/kiota-abstractions";

const myUser = {
    firstName: "Jane",
    lastName: "Smith"
};

const result = serializeToJson(myUser, serializeUser);
// gives us {"firstName":"Jane","lastName":"Smith"}
// alternatively if you need to serialize to a different content type you can use this helper instead
// const result = serialize("application/json", myUser, serializeUser);

const deserializedUser = deserializeFromJson(result, createUserFromDiscriminatorValue);
```

---
