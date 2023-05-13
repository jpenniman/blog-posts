---
layout: post
title: Incorporating Snyk into Continuous Integration with Azure Yaml Pipelines
date: 2019-07-10
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: Let's take a look at how we can incorporate Snyk into our Azure Pipelines as part of Continuous Security during Continuous Integration.
category: Development
tags:
- Azure DevOps
- Azure Pipelines
- CI/CD/CS
- Snyk
---

Automate all the things! The same goes for security checks in our application. Continuous Security is the automation of these checks as part of the continuous delivery pipeline. The type of check determines where the check can, or should, go. Static testing (SaST), for example, should happen outside of, but be triggered by, CI. Dynamic Testing (DaST) happens outside of, but triggered by, deployment.

Another scan we can, and should, perform is a security analysis of packages we're pulling into our applications. This can happen during CI, and the build or pull request can be rejected if packages with known vulnerabilities are used.

There are several tools emerging in this space, one of which is [Snyk.io](https://snyk.io/). These tools compare your imported packages and versions to those listed in various CVE databases to determine if a package has a known vulnerability, and, if applicable, report the version you should upgrade to in order to patch the vulnerability.

Some tools, like Snyk or even [Github](https://help.github.com/en/articles/about-security-alerts-for-vulnerable-dependencies) themselves, will continuously monitor your repository, so if a CVE is later issued for a package you are using, you can be alerted.

Snyk also has a CLI tool that allows you to perform a scan immediately against your project. The scan is fast, since it is simply searching the databases for packages listed in your dependencies, so build time is a good place for it. This can prevent a vulnerability from getting deployed to any environments.

Let's look at how.

## Creating a CLI Token for Snyk

The first thing you need is an API token for your Snyk account. To do this, log in to your Snyk account, and go to Account Settings.

![Snyk Account Settings](/blog/images/snyk_token.png)

## Setting up the Pipeline

The first step is to install the Snyk CLI onto the build agent. The CLI is a NodeJS module, so we can simply use the NPM task to install it.

Then, we can use a script task to authenticate to Snyk, run the test, and setup the project to be continuously monitored.  The CLI will return a non-success error code if any packages have known vulnerabilities.

``` yaml
name: $(Date:yyyy.MM.dd).$(Rev:r)
variables:
  SNYK_TOKEN: <The API Token you created in the Snyk portal>
jobs:
- job:
  steps:
  - task: Npm@1
    displayName: 'Install Snyk.io'
    inputs:
      command: custom
      verbose: false
      customCommand: 'install -g snyk'

  - script: |
      snyk auth $(SNYK_TOKEN)
      snyk test
    displayName: 'Scan dependencies for vulnerabilities with Snyk'

  - script: |
      snyk monitor
    displayName: 'Setup dependency vulnerability monitoring with Snyk'


  - task: DotNetCoreCLI@2
    inputs:
      command: build
      arguments: -p:Version=$(Build.BuildNumber)
```

## Conclusion

Security vulnerabilities in our application's dependencies are a real threat. Snyk's co-founder Guy Podjarny has a [great presentation he did at QCon](https://www.infoq.com/presentations/serverless-security-2017/) where he uses a known vulnerability in a popular NodeJS module to obtain a remote SSH session into an AWS Lambda and take control of the underlying container. Scanning for these vulnerabilities is fast, easy, and cheap. There is no excuse to not be scanning for them.

This is only one piece if the puzzle but a great compliment to your static and dynamic code scans.
