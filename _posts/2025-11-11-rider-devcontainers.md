---
layout: post
title: Preparing a Devcontainer for JetBrains Rider
date: 2025-11-11
image: /blog/images/build-container.jpg
author: Jason M Penniman
excerpt: The dev containers for dotnet published by Microsoft do not have all the prerequisites for Rider or any other
         JetBrains IDE.
tags:
- dotnet
- C#
- Rider
- devcontainers
---

The dev containers for dotnet published by Microsoft do not have all the prerequisites for Rider or any other JetBrains
IDE. It is missing the Java Runtime and sshd.

The fix is simple enough. We can use features to run apt and install the package. To our `devcontainer.json`, we add

``` json
  "features": {
    "ghcr.io/rocker-org/devcontainer-features/apt-packages:1": {
      "packages": "openssh-server,openjdk-25-jre"
    }
```

An complete example `devcontainer.json`:

``` json
{
  "name": "Northwind Trading Post",
  "image": "mcr.microsoft.com/devcontainers/dotnet:dev-10.0-noble",
  "features": {
    "ghcr.io/rocker-org/devcontainer-features/apt-packages:1": {
      "packages": "openssh-server,openjdk-25-jre"
    }
  }
}
```

Cheers!
