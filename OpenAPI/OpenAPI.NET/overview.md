---
title: Overview
description: Intorduction on how to use OpenAPI.NET library
author: njaci1
ms.author: kelvinnjaci
ms.topic: conceptual
date: 01/02/2024
---
# Overview
OpenAPI.NET is a .NET library for working with OpenAPI documents. OpenAPI is a standard specification for describing RESTful APIs in a machine-readable and human-friendly way. With OpenAPI.NET, you can create, modify, validate, and convert(between JSON and YAML) OpenAPI documents in your .NET applications. The library includes readers and writers for serializing and deserializing OpenAPI documents to and from different formats.

At a high level, OpenAPI.NET works by providing an object model for OpenAPI documents. This object model represents the different components of an OpenAPI document, such as its paths, operations, parameters, and responses. 
  
## Why consider using OpenAPI.NET
Some of the benefits for using the OpenAPI.NET library when working with OpenAPI documents include:
* **Ease of use:** OpenAPI.NET provides a useful object model for OpenAPI documents, making it easy to create and manipulate OpenAPI documents in your .NET code. 

* **Flexibility:** OpenAPI.NET includes readers and writers for serializing and deserializing OpenAPI documents to and from different formats, including JSON and YAML, giving you the flexibility to work with OpenAPI documents in the format of your choice. 

* **Reliability:** OpenAPI.NET can validate OpenAPI documents to ensure that they conform to the OpenAPI specification, helping you to catch and fix errors early in the development process. 

* **Interoperability:** OpenAPI.NET can convert OpenAPI documents between different versions (versions 2.0 and 3.0 and soon 3.1) of the OpenAPI specification, as well as between different formats (JSON and YAML), making it easy to share and collaborate on OpenAPI documents with others. 

Here is an example of some of the usecases:
Let’s say we have a service called “PetStore” that allows users to view and  post pets to the online store. We want to create an OpenAPI document to describe the PetStore service RESTful API.
* [Creating an OpenAPI document.](creating-an-openAPI-document.md).
* [Modifying an OpenAPI document.](modifying-an-openAPI-document.md)
* [Coverting an OpenAPI document.](converting-an-openAPI-document.md)
