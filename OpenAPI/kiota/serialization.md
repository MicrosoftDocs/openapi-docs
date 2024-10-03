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

The additional data holder interface defines members implemented by models that might include properties in API responses that aren't described in the OpenAPI description of the API. As the type information for these types is unknown at generation time, default serializers will make a best effort attempt during deserialization using [`UntypedNode`](#untyped-node) types.

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

## Types mapping

The table below describes the mapping between [OpenAPI type-format pairs](https://spec.openapis.org/registry/format/) and language types.

| OpenAPI type | OpenAPI format | C# Type | Go Type | Java Type | TypeScript Type | PHP Type | Python Type |
| ------------ | -------------- | ------- | ------- | --------- | --------------- | -------- | ----------- |
| number | int16 |
| number | int16 |

## Untyped Node

In scenarios where the type information for a property/parameter in the input OpenAPI description is not present, Kiota will generate with the generic `UntypedNode` type which represent that the property/parameter could be a primitive,object or a collection. As the type information is unknown at compile/generation time, the `UntypedNode` object can be parsed at runtime by calling the `GetValue()` methods of the derived types to get the native representation of the node.

The example below shows a sample generic function that parses an `UntypedNode` object parse out the various elements inside it.

<!-- markdownlint-disable-next-line MD051 -->
### [C#](#tab/csharp)

```cs

using Microsoft.Kiota.Abstractions.Serialization;

private static void ParseUntypedNode(UntypedNode untypedNode)
{
    switch (untypedNode)
    {
        case UntypedObject untypedObject:
            Console.WriteLine("Found object value: ");
            foreach (var (name, node) in untypedObject.GetValue())
            {
                Console.WriteLine("Property Name: " + name);
                ParseUntypedNode(node);// handle the nested nodes
            }
            break;

        case UntypedArray untypedArray:
            Console.WriteLine("Found array value: ");
            foreach (var item in untypedArray.GetValue())
            {
                Console.WriteLine("Array Item: ");
                ParseUntypedNode(item);// handle the nested nodes
            }
            break;

        case UntypedString:
        case UntypedBoolean:
        case UntypedDouble:
        case UntypedFloat:
        case UntypedInteger:
        case UntypedLong:
        case UntypedNull:
            string scalarAsString = KiotaJsonSerializer.SerializeAsString(untypedNode);
            Console.WriteLine("Found scalar value : " + scalarAsString);
            break;
    }
}
```

<!-- markdownlint-disable-next-line MD051 -->
### [Java](#tab/java)

```java
import com.microsoft.kiota.serialization.*;

public static void parseUntypedNode(UntypedNode node) throws IOException {
    switch (node) {
        case UntypedArray array -> {
            System.out.println("Found array value");
            for(var item: array.getValue()){
                System.out.println("New Item");
                parseUntypedNode(item);// handle the nested nodes
            }
        }
        case UntypedObject object -> {
            System.out.println("Found object value");
            for(var item: object.getValue().entrySet()){
                System.out.println("Property name: " + item.getKey());
                parseUntypedNode(item.getValue());// handle the nested nodes
            }
        }
        default  -> {
            String scalarAsString = KiotaJsonSerialization.serializeAsString(node);
            System.out.println("Found scalar value: " + scalarAsString);
        }
    };
}
```

<!-- markdownlint-disable-next-line MD051 -->
### [Go](#tab/go)

```go
import (
    kiotaSerialization "github.com/microsoft/kiota-abstractions-go/serialization"
)

func parseUntypedNode(node absser.UntypedNodeable) {
    switch value := node.(type) {
        case *absser.UntypedObject:
            fmt.Printf("Found object value: \n")
            for key, val := range value.GetValue() {
                fmt.Printf("Property Name: %v \n", key)
                parseUntypedNode(val)
            }
        case *absser.UntypedArray:
            fmt.Printf("Found array value: \n")
            for _, item := range value.GetValue() {
                fmt.Printf("New Item: \n")
                parseUntypedNode(item)
            }
        case *absser.UntypedBoolean:
            fmt.Printf("Found bool value: %v \n", *value.GetValue())
        case *absser.UntypedFloat:
            fmt.Printf("Found float value: %v \n", *value.GetValue())
        case *absser.UntypedDouble:
            fmt.Printf("Found double value: %v \n", *value.GetValue())
        case *absser.UntypedInteger:
            fmt.Printf("Found int value: %v \n", *value.GetValue())
        case *absser.UntypedLong:
            fmt.Printf("Found long value: %v \n", *value.GetValue())
        case *absser.UntypedNull:
            fmt.Printf("Found null value: %v \n", value.GetValue())
        case *absser.UntypedString:
            fmt.Printf("Found string value: %v \n", *value.GetValue())
    }
}
```

<!-- markdownlint-disable-next-line MD051 -->
### [TypeScript](#tab/typescript)

```TS
import {UntypedNode, isUntypedArray, isUntypedObject, serializeToJson, serializeUntypedNode} from "@microsoft/kiota-abstractions";

function parseUntypedJson(node: UntypedNode) {
    if (isUntypedArray(node)) {
        console.log("Found array type: ");
        for (const dataSet of node.getValue()) {
            parseUntypedJson(dataSet);// handle the nested nodes
        }
    }
    else if (isUntypedObject(node)) {
        const properties = node.getValue();
        console.log("Found object type: ");
        for (const [key,value] of Object.entries(properties)) {
            console.log("Found property with name: " + key);
            parseUntypedJson(value);// handle the nested nodes
        }
    }
    else {
        var scalarAsString = serializeToJson(node, serializeUntypedNode);
        console.log("Found scalar value: " + scalarAsString);
    }
}
```

---

> [!NOTE]
> As objects of `UntypedNode` implement the `IParsable` interface it is possible to use the serialization helpers from Kiota to serialize them to a string/stream and parse it with a preferable parser based on scenario.

The next example below shows how one can use the `UntypedNode` objects to build a payload that sends a multi-dimensional array(tabular data).

<!-- markdownlint-disable-next-line MD051 -->
### [C#](#tab/csharp)

```cs

using Microsoft.Kiota.Abstractions.Serialization;

var table = new UntypedArray(new List<UntypedNode>
{
    new UntypedArray(new List<UntypedNode>{ new UntypedString("1"),new UntypedString("2"), new UntypedString("3"),}),// row 1
    new UntypedArray(new List<UntypedNode>{ new UntypedString("4"),new UntypedString("5"), new UntypedString("6"),}),// row 2
});

var jsonString = KiotaJsonSerializer.SerializeAsString(table);
```

<!-- markdownlint-disable-next-line MD051 -->
### [Java](#tab/java)

```java
import com.microsoft.kiota.serialization.*;
import java.util.Arrays;

var table = new UntypedArray(Arrays.asList(
        new UntypedArray(Arrays.asList(
                new UntypedString("1"), new UntypedString("2"), new UntypedString("3")
        )),//row 1
        new UntypedArray(Arrays.asList(
                new UntypedString("4"), new UntypedString("5"), new UntypedString("6")
        ))//row 2
));

String jsonString = KiotaJsonSerialization.serializeAsString(table);
```

<!-- markdownlint-disable-next-line MD051 -->
### [Go](#tab/go)

```go
import (
    kiotaSerialization "github.com/microsoft/kiota-abstractions-go/serialization"
)

table := make([]kiotaSerialization.UntypedNodeable, 2)

firstRow := make([]kiotaSerialization.UntypedNodeable, 3)
firstRow[0] = kiotaSerialization.NewUntypedString("1")
firstRow[1] = kiotaSerialization.NewUntypedString("2")
firstRow[2] = kiotaSerialization.NewUntypedString("3")

table[0] = kiotaSerialization.NewUntypedArray(firstRow) // row 1

secondRow := make([]kiotaSerialization.UntypedNodeable, 3)
secondRow[0] = kiotaSerialization.NewUntypedString("4")
secondRow[1] = kiotaSerialization.NewUntypedString("5")
secondRow[2] = kiotaSerialization.NewUntypedString("6")
table[1] = kiotaSerialization.NewUntypedArray(secondRow) // row 2

tableNode := kiotaSerialization.NewUntypedArray(table)

jsonString, err := kiotaSerialization.SerializeToJson(tableNode)
```

<!-- markdownlint-disable-next-line MD051 -->
### [TypeScript](#tab/typescript)

```TS
import {createUntypedArray, createUntypedString, serializeToJson, serializeToJson} from "@microsoft/kiota-abstractions";

const table = createUntypedArray([
    createUntypedArray([
        createUntypedString("1"), createUntypedString("2"), createUntypedString("3"),// row 1
    ]),
    createUntypedArray([
        createUntypedString("4"), createUntypedString("5"), createUntypedString("6"),// row 2
    ]),
]);

var json = serializeToJson(table, serializeUntypedNode);
```

---
