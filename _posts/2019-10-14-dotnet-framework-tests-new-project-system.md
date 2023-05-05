---
layout: post
title: .Net Full Framework Tests with the New Project System
date: 2019-10-14
author: Jason M Penniman
excerpt: The new project system in the dotnet tooling can be used to build and test .Net Framework targets too.
tags:
- dotnet
- unit test
---

I ran into a scenario where my solution had libraries targeting netStandard2.0 and/or net461. For the libraries targeting just net461, I created a “MSTest Unit Test Project (.Net Framework)” project–the “old” project system. The tests ran fine in Visual Studio, but in my CI build, I wanted to keep it simple and run all the tests with `dotnet test`. However, this tooling cannot see/run the old test project.

So, as an experiment, I tried creating a test project in the new project system, and changing the target moniker from NetCoreApp to net461.  It worked! both dotnet test and Visual Studio can run the tests. It’s worth noting that the library is also the new project system. I haven’t tested if that matters.
Test Project XML

``` xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net461</TargetFramework>
    <IsPackable>false</IsPackable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="16.3.0" />
    <PackageReference Include="MSTest.TestAdapter" Version="2.0.0" />
    <PackageReference Include="MSTest.TestFramework" Version="2.0.0" />
    <PackageReference Include="coverlet.msbuild" Version="2.7.0">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
    </PackageReference>
  </ItemGroup>

</Project>
```

Cheers!
