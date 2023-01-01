---
layout: post
title: Continuous Delivery of NuGet Packages with Azure Yaml Pipelines
date: 2019-07-06
author: Jason M Penniman
excerpt: Let's take a look at an approach to building, versioning, and delivering Nuget packages with Azure Pipelines.
category: Development
tags:
- Azure DevOps
- Azure Pipelines
- CI/CD
- Nuget
---

## Goals

* Package versioning should follow SemVer
* Assembly versioning should follow Microsoft's recommend best practice
* A single binary to be promoted from pre-release to release
* Meaningful build nubmers
* Commit tagging

## Versioning
It is recommended that Nuget packages be versioned using SemVer. SemVer uses a 3 part version number:

Major.Minor.Patch

Where:

* **Major** is incremented for breaking changes
* **Minor** is incremented for feature enhancements
* **Patch** is incremented for bug fixes when no new feaatures are being added

A simple approach is to hardcode this with the build number and let the tooling do it's magic, using the build number system variable:

``` yaml
name: 1.0.$(Rev:r)
jobs:
- job:
  steps:
  - task: DotNetCoreCLI@2
    inputs:
      command: pack
      arguments: -p:Version=$(Build.BuildNumber)
```

This doesn't follow the rules exactly, since you'd get a patch revision for every build. Nuget doesn't care, and you don't need to care either. It is still stating that this is a revision of a given feature set. If you're doing manual builds, this may suffice, but automated builds a CI will make this look funny. It will also make it look like your packages have a lot of bugs.

While it's not part of the SemVer spec, Nuget does support 4 part version numbers, so you could use:

Major.Minor.Patch.Build

``` yaml
name: 1.0.0.$(Rev:r)
jobs:
- job:
  steps:
  - task: DotNetCoreCLI@2
    inputs:
      command: pack
      arguments: -p:Version=$(Build.BuildNumber)
```

This solves the problem, but doesn't adhere to SemVer. The SemVer 2.0 spec allows for the specification of a build (sort of). We'll take a look at this a bit later.

### Prerelease versions

SemVer specifies a notation for prerelease versions. To denote a prerelease, you append a "-" followed by an alphanumeric string:

1.0.0-beta1

The alpanumeric string can be anything. Some common ones are:

1.0.0-alpha1
1.0.0-beta1
1.0.0-preview1
1.0.0-rc1

I like to use what makes sense for the code I'm releasing. If it's truly unstable alpha code I want to get people playing with, I'll use alpha, otherwise, if I think it's stable enough to use in production, but I'm not quite ready to call it GA, I'll use beta or preview.

The basic approach again, is to just use the build number:

``` yaml
name: 1.0.0-beta$(Rev:r)
jobs:
- job:
  steps:
  - task: DotNetCoreCLI@2
    inputs:
      command: pack
      arguments: -p:Version=$(Build.BuildNumber)
```

It works, but this would only ever create beta builds. We'll address this in a minute.

### Post-Release Versions

SemVer 2.0 allows you to specify build numbers by appending a "+" sign followed by a number:

1.0.0+1

This version is considered greater than 1.0.0, and would be selected if you asked nuget for version 1.0.0. The downside is, this is not supported by older Nuget clients or older versions of Visual Studio. So if you want to continue to support older tooling, you can't use it.

Again, a simple approach is:


``` yaml
name: 1.0.0+$(Rev:r)
jobs:
- job:
  steps:
  - task: DotNetCoreCLI@2
    inputs:
      command: pack
      arguments: -p:Version=$(Build.BuildNumber)
```

## A More Comprehencive Approach

We can leverage variables in our pipeline to create a meaningful, unique build nubmer and SemVer compliant package versions. We'll hardcode the root version (major.minor.patch).

``` yaml
name: $(version)-build$(Rev:r)
variables:
  version: 1.0.0
  assemblyVersion: $(version).$(Rev:r)
  prereleaseVersion: $(version)-beta$(Rev:r)
  packageVersion: $(version)
jobs:
- job:
  steps:
  - task: DotNetCoreCLI@2
    inputs:
      command: build
      arguments: -p:Version=$(assemblyVersion)
  - task: DotNetCoreCLI@2
    inputs:
      command: pack
      arguments: --no-build -p:PackageVersion=$(prereleaseVersion)
  - task: DotNetCoreCLI@2
    inputs:
      command: pack
      arguments: --no-build -p:PackageVersion=$(packageVersion)
```

### Build Number

Since build numbers need to be unique but should still be meaningful, I use the version followed by `-build` and the built-in auto incrementing revision. The first time this build is run, the build number would look like:

`1.0.0-build1`

### Assembly Version

The assembly itself needs to be versioned. Microsoft uses a 4 part version number, so we'll use Major.Minor.Patch.Build. The first time this build is run, the assembly version will be:

`1.0.0.1`

### Package Version

Notice that I build once first, then pack twice without rebuilding. This is so I can create a pre-release package and a GA package using the same exact binaries. Later, in our release pipeline, we'll setup deployment approvals to release the pre-release first, then the GA when ready.

What we endup with are 2 packages, versioned like so:

`MyAwesomeLibrary.1.0.0-beta1`

`MyAwesomeLibrary.1.0.0`

