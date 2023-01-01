---
layout: post
title: Infinite loop in ADAL and ADAL-ANGULAR
date: 2016-10-09 18:55:54.000000000 -04:00
author: Jason M. Penniman
excerpt: In a recent project we came across in issue where ADAL would go into an infinite loop when renewing a token.  It's a known issue with many causes/fixes, some of which were bugs fixed in the 1.11 release...
category: Blog
---
In a recent project we came across in issue where ADAL would go into an infinite loop when renewing a token.  It's a known issue with many causes/fixes, some of which were bugs fixed in the 1.11 release:

<a href="https://github.com/AzureAD/azure-activedirectory-library-for-js/issues/216">https://github.com/AzureAD/azure-activedirectory-library-for-js/issues/216</a>

<a href="https://github.com/AzureAD/azure-activedirectory-library-for-js/issues/298">https://github.com/AzureAD/azure-activedirectory-library-for-js/issues/298</a>

<a href="http://stackoverflow.com/questions/37211367/adal-adal-angular-refresh-token-infinite-loop">http://stackoverflow.com/questions/37211367/adal-adal-angular-refresh-token-infinite-loop</a>

It all applied to our situation.

### Our scenario:

* ASP.Net Core 1.0.1 =&gt; for a Rest API and a hybrid app (we use layout and index.cshtml as the page for the angular SPA)
* AngularJS 1.5.7
* RequireJS and AngularAMD
* ngRoute
* A root controller on our body tag
* AzureAD, Multi-Tenant, API Secured with JWTBearerToken

### The solution for us:

1. Upgrade to 1.11
2. Ensure there were no http requests int he root controller
3. Ensure the "otherwise" in the route wasn't to the root--we made it go to a custom 404 page
4. Explicitly define our endpoints
5. Explicitly define our anonymousEndpoints

``` js
adalProvider.init(
  {
      instance: 'https://login.microsoftonline.com/',
      tenant: 'common',
      clientId: '00000000-0000-0000-0000-000000000000',
      extraQueryParameter: 'nux=1',
      anonymousEndpoints: ['/app/', '/js/', '/css/', '/GeneratedCode/', 'templates/'],
      endpoints: { 'api': '00000000-0000-0000-0000-000000000000' }

      //cacheLocation: 'localStorage', // enable this for IE, as sessionStorage does not work for localhost.
  },
  $httpProvider
);
```