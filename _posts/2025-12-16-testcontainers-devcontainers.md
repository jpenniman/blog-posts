---
layout: post
title: Using TestContainers inside a Dev Container
date: 2025-12-16
image: /blog/images/test-containers-error.png
author: Jason M Penniman
excerpt: "I was getting a `NullReferenceException` when trying to use TestContainers inside a devcontainer."
tags:
- TestContainers
- devcontainer
---

__*Related GitHub Issue:*__ https://github.com/testcontainers/testcontainers-dotnet/issues/1607

I was running into an issue when using TestContainers inside a Dev Container. When trying to spin up a container in my
test fixture, I was getting the following error:

```
Exception has occurred: CLR/System.NullReferenceException

An exception of type 'System.NullReferenceException' occurred in Testcontainers.dll but was not handled in user code: 'Object reference not set to an instance of an object.'

at DotNet.Testcontainers.Clients.DockerApiClient..ctor(Guid sessionId, IDockerEndpointAuthenticationConfiguration dockerEndpointAuthConfig, ILogger logger) 
at DotNet.Testcontainers.Clients.DockerContainerOperations..ctor(Guid sessionId, IDockerEndpointAuthenticationConfiguration dockerEndpointAuthConfig, ILogger logger) 
at DotNet.Testcontainers.Clients.TestcontainersClient..ctor(Guid sessionId, IDockerEndpointAuthenticationConfiguration dockerEndpointAuthConfig, ILogger logger) 
at DotNet.Testcontainers.Containers.DockerContainer..ctor(IContainerConfiguration configuration) 
at Testcontainers.MongoDb.MongoDbContainer..ctor(MongoDbConfiguration configuration) 
at CustomerService.Tests.MongoFixture..ctor() in /workspaces/TradingPost/src/customer-service/tests/CustomerService.Tests/MongoFixture.cs:line 13 
at System.Reflection.MethodBaseInvoker.InvokeWithNoArgs(Object obj, BindingFlags invokeAttr)
```

In the same environment, the docker CLI works and the Aspire CLI works, but TestContainers would fail to create an
instance of `DockerApiClient`.

A minor tweak to the devcontainer configuration did the trick.

```json
    "ghcr.io/devcontainers/features/docker-in-docker:2": {
      "moby": true
    },
```

After making this configuration change, it worked like a charm.

In most environments, TestContainers *should* "just work". There is something about my workstation's configuration that
causes TestContainers to fail without setting this configuration option. If you find yourself getting the same error,
give it try.

Cheers!
