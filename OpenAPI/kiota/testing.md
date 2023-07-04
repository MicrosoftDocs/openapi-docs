---
title: Unit testing with Kiota API clients
description: Learn about writing unit tests with Kiota API clients.
author: LockTar
ms.author: 
ms.topic: conceptual
date: 07/04/2023
---

# Unit testing with Kiota API clients

Code were Kiota API clients are used can be tested. In this document is explained how to mock the Microsoft Graph SDK.

## Prerequisite

In this sample the following frameworks/tools are used.

- [Microsoft Graph SDK v5](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [xUnit](https://xunit.net/)
- [NSubstitute](https://nsubstitute.github.io/)

## Microsoft Graph User unit tests

```C#
using System.Text.Json;
using Microsoft.Graph;
using Microsoft.Graph.Models;
using Microsoft.Kiota.Abstractions;
using Microsoft.Kiota.Abstractions.Serialization;
using Microsoft.Kiota.Serialization.Json;
using NSubstitute;

namespace Tests;

public class MicrosoftGraphUserTests
{
    [Fact]
    public async Task Microsoft_Graph_All_Users()
    {
        // Arrange
        var requestAdapter = Substitute.For<IRequestAdapter>();
        GraphServiceClient graphServiceClient = new GraphServiceClient(requestAdapter);

        var usersMock = new UserCollectionResponse()
        {
            Value = new List<User> {
                new User()
                {
                    Id = Guid.NewGuid().ToString(),
                    GivenName = "John",
                    Surname = "Doe"
                },
                new User()
                {
                    Id = Guid.NewGuid().ToString(),
                    GivenName = "Jane",
                    Surname = "Doe"
                }
            }
        };

        // Return the mocked list of users
        requestAdapter.SendAsync(
            Arg.Any<RequestInformation>(),
            Arg.Any<ParsableFactory<UserCollectionResponse>>(),
            Arg.Any<Dictionary<string, ParsableFactory<IParsable>>>(),
            Arg.Any<CancellationToken>())
            .ReturnsForAnyArgs(usersMock);

        // Act
        var users = await graphServiceClient.Users.GetAsync();

        // Assert
        Assert.NotNull(users);
        Assert.Equal(2, users.Value.Count);
    }

    [Fact]
    public async Task Microsoft_Graph_User_By_Id()
    {
        // Arrange
        var requestAdapter = Substitute.For<IRequestAdapter>();
        GraphServiceClient graphServiceClient = new GraphServiceClient(requestAdapter);

        var johnObjectId = Guid.NewGuid().ToString();
        var janeObjectId = Guid.NewGuid().ToString();

        var userJohn = new User()
        {
            Id = johnObjectId,
            GivenName = "John",
            Surname = "Doe"
        };

        var userJane = new User()
        {
            Id = janeObjectId,
            GivenName = "Jane",
            Surname = "Doe"
        };

        // Return the mocked john object if the path contains the object id of john
        requestAdapter.SendAsync(
            Arg.Is<RequestInformation>(ri => ri.PathParameters.Values.Contains(johnObjectId)),
            Arg.Any<ParsableFactory<User>>(),
            Arg.Any<Dictionary<string, ParsableFactory<IParsable>>>(),
            Arg.Any<CancellationToken>())
                .Returns(userJohn);

        // Return the mocked Jane object if the path contains the object id of jane
        requestAdapter.SendAsync(
            Arg.Is<RequestInformation>(ri => ri.PathParameters.Values.Contains(janeObjectId)),
            Arg.Any<ParsableFactory<User>>(),
            Arg.Any<Dictionary<string, ParsableFactory<IParsable>>>(),
            Arg.Any<CancellationToken>())
                .Returns(userJane);

        // Act
        var userResultJohn = await graphServiceClient.Users[johnObjectId].GetAsync();
        var userResultJane = await graphServiceClient.Users[janeObjectId].GetAsync();

        // Assert
        Assert.Equal("John", userResultJohn.GivenName);
        Assert.Equal("Jane", userResultJane.GivenName);
    }

    [Fact]
    public async Task Microsoft_Graph_User_By_GivenName()
    {
        // Arrange
        var requestAdapter = Substitute.For<IRequestAdapter>();
        GraphServiceClient graphServiceClient = new GraphServiceClient(requestAdapter);

        var johnObjectId = Guid.NewGuid().ToString();
        var janeObjectId = Guid.NewGuid().ToString();

        var userJohn = new User()
        {
            Id = johnObjectId,
            GivenName = "John",
            Surname = "Doe"
        };

        var userJane = new User()
        {
            Id = janeObjectId,
            GivenName = "Jane",
            Surname = "Doe"
        };

        // Return the mocked list of users that contains mocked John object when the query filter parameter is "givenName='John'"
        requestAdapter.SendAsync(
            Arg.Is<RequestInformation>(ri => ri.QueryParameters["%24filter"].ToString() == "givenName='John'"),
            Arg.Any<ParsableFactory<UserCollectionResponse>>(),
            Arg.Any<Dictionary<string, ParsableFactory<IParsable>>>(),
            Arg.Any<CancellationToken>())
                .Returns(new UserCollectionResponse()
                {
                    Value = new List<User>() { userJohn }
                });

        // Return the mocked list of users that contains mocked Jane object when the query filter parameter is "givenName='Jane'"
        requestAdapter.SendAsync(
            Arg.Is<RequestInformation>(ri => ri.QueryParameters["%24filter"].ToString() == "givenName='Jane'"),
            Arg.Any<ParsableFactory<UserCollectionResponse>>(),
            Arg.Any<Dictionary<string, ParsableFactory<IParsable>>>(),
            Arg.Any<CancellationToken>())
                .Returns(new UserCollectionResponse()
                {
                    Value = new List<User>() { userJane }
                });

        // Act
        var userResultJohn = await graphServiceClient.Users.GetAsync(rc =>
        {
            rc.QueryParameters.Filter = "givenName='John'";
        });

        var userResultJane = await graphServiceClient.Users.GetAsync(rc =>
        {
            rc.QueryParameters.Filter = "givenName='Jane'";
        });

        // Assert
        Assert.NotNull(userResultJohn);
        Assert.Single(userResultJohn.Value);
        Assert.Equal("John", userResultJohn.Value.First().GivenName);

        Assert.NotNull(userResultJane);
        Assert.Single(userResultJane.Value);
        Assert.Equal("Jane", userResultJane.Value.First().GivenName);
    }

    [Fact]
    public async Task Microsoft_Graph_User_Create_New()
    {
        // Arrange
        var requestAdapter = Substitute.For<IRequestAdapter>();

        // Create new JsonSerializationWriter so we can check the content of the post calls
        var applicationJsonContentType = "application/json";
        requestAdapter.SerializationWriterFactory.GetSerializationWriter(applicationJsonContentType).Returns(x => new JsonSerializationWriter());

        // Capture the request information (so we can check it) in a list because in this tests we have multiple posts.
        // Only needed when your class is doing multiple posts
        var requestInformations = new List<RequestInformation>();
        await requestAdapter.SendAsync(
            Arg.Do<RequestInformation>(ri => requestInformations.Add(ri)),
            Arg.Any<ParsableFactory<User>>(),
            Arg.Any<Dictionary<string, ParsableFactory<IParsable>>?>(),
            Arg.Any<CancellationToken>());

        GraphServiceClient graphServiceClient = new GraphServiceClient(requestAdapter);

        var johnObjectId = Guid.NewGuid().ToString();
        var userJohn = new User()
        {
            Id = johnObjectId,
            GivenName = "John",
            Surname = "Doe"
        };

        // Act
        var userResultJohn = await graphServiceClient.Users.PostAsync(userJohn);

        // Checks if the call is done (receives a call)
        await requestAdapter.Received(1).SendAsync(
            Arg.Is<RequestInformation>(ri => ri.UrlTemplate!.Contains("/users")),
            Arg.Any<ParsableFactory<User>>(),
            Arg.Any<Dictionary<string, ParsableFactory<IParsable>>>(),
            Arg.Any<CancellationToken>());

        // Get request information to check the actual posted content
        var requestInfo = requestInformations.Single(ri => ri.UrlTemplate!.Contains("/users"));
        var model = JsonSerializer.Deserialize<User>(requestInfo.Content,
            new JsonSerializerOptions()
            {
                PropertyNameCaseInsensitive = true
            });

        // Assert
        Assert.Equal(userJohn.GivenName, model.GivenName);
        Assert.Equal(userJohn.Surname, model.Surname);
        Assert.Equal(userJohn.Id, johnObjectId);
    }
}
```
