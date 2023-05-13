---
layout: post
title: Code Coverage for Multiple Projects in a Single Build using Dotnet Test and Coverlet
date: 2019-10-14
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: Let's talk a look at aggregating code coverage results from multiple test projects into a single report for upload to Azure DevOps.
tags:
- dotnet
- unit test
- Azure DevOps
---

Most of the time, your solution will have more than one project and a test unit project for each of those. Azure DevOps only, as of this writing, only allows you to update a single code coverage summary. If you upload more than one, each overwrites the next and only the last one remains. To accomplish this, we can use the merge functionality in Coverlet. I use the MSBuild extension, because it is better suited for CI pipelines.

In your test project XML, add the package...

<PackageReference Include="coverlet.msbuild" Version="2.7.0">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
</PackageReference>

Then, in your dotnet test command or msbuild command, tell it to use Coverlet and to merge results. If you're using Azure DevOps, your test task looks this... (line wrapped for read ability in the article)

``` yml
- task: DotNetCoreCLI@2
  displayName: Test
  inputs:
    command: test
    publishTestResults: false
    arguments: --no-restore -c debug --logger trx 
               -r $(Agent.TempDirectory)/TestResults 
               -p:CollectCoverage=true 
               -p:CoverletOutput="$(Agent.TempDirectory)/TestResults/" 
               -p:CoverletOutputFormat="json%2cCobertura" 
               -p:MergeWith="$(Agent.TempDirectory)/TestResults/coverage.json" 
    projects: |
      test/**/*.csproj
```

What we're doing here is:

- Telling the rest runner enable code coverage
- Telling Coverlet to put the results in the agent's temp directory
- Telling Coverlet to output both it's own JSON format and the Cobertura format understood by Azure Dev Ops
- Telling Coverlet to merge the current results with the previous.

When all the tests have run, there will be a single pair of files (`coverage.json` and `coverage.cobertura.xml`) in the `$(Agent.TempDirectory)/TestResults` directory. You can then publish the cobertura summary file with the Publish Code Coverage task:

``` yml
- task: PublishCodeCoverageResults@1
  displayName: Publish code coverage results
  inputs:
    codeCoverageTool: "Cobertura"
    summaryFileLocation: "$(Agent.TempDirectory)/TestResults/coverage.cobertura.xml"
```

Cheers!
