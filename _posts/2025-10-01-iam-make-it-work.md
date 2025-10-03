---
layout: post
title: Tortis IAM - Making it work
date: 2025-10-01
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: 
  Earlier this year, I set out to build an IAM solution. In this article, I discuss my experience and findings in
  making it work.
tags:
- dotnet
- C#
- IAM
- OpenId Connect
- Authentication
- Authorization
---

Earlier this year, I set out to build an IAM solution. In this article, I discuss my experience and findings to "make 
it work. In subsequent articles, I will discuss how I "make it right" and "make it fast".

The initial catalyst behind the project was the need for a full IAM solution for a project and the licensing
changes to Identity Server 4 (now the commercial, non-open source product "Duende Identity Server"). I have nothing
against Duende or Dominick's and Brock's decision--Duende Identity Server is an amazing offering for the money.
The community, though, still has a strong desire for something free as in free beer.
Being an free open source advocate, I also believe in free software as in freedom or free speech.

The work in progress in here: https://github.com/tortis-dev/iam/tree/develop

## Build vs "Buy"

I need a full IAM solution supporting interactive user authentication and role-based authorization in addition
to the OAuth/OpenId Connect bits. Something like [KeyCloak](https://www.keycloak.org/) but in the .Net ecosystem. I set
out to build my own everything. Step 1: read the OpenId Connect specification.

I realized that anything beyond a simple secure token service for machine-to-machine communication with a basic
client_credential grant using a client id and secret was going to be a lot of work. I mean a LOT of work. I didn't have 
the time to invest in something that involved, so I looked at [OpenIddict](https://documentation.openiddict.com/). 
While I'm not crazy about the implementation approach, Kevin did a great job implementing the spec. It contains enough 
of the spec to support a Financial API compliant product, which was a huge plus at the time I was evaluating solutions.
Ok, so OpenIddict for the OpenID Connect bits, fine. I can abstract it away if needed so when I want to have some fun
and implement the spec from scratch, its tentacles aren't too tangled in the solution. 

What about the user and role management pieces? Seems simple enough to write from scratch, and is from the perspective 
of "not complicated". However, there is a lot there too in a full featured solution. Proper salting and peppering of
password hashes, multi-factor authentication support, and so on. Again, it came down to time. Do I have the time to
invest in building a solution from scratch? I opted for 
[AspNet Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity).
This can also be abstracted away to avoid being too tightly coupled. 

So there you have it. Tortis IAM will be built atop AspNet Core Identity and OpenIddict. In the future this may change,
but for now, it gets me a working solution with minimal effort...in theory.


## The UI

I opted for Blazor. The decision was mostly to learn Blazor.

For the controls, I went back and forth with just using Bootstrap, but ultimately settled on FluentUI.
I knew I would be making use of data grids and using a suite like FluentUI simplifies that work.

Aside: when using plain old HTML, CSS, and Javascript, my go-to data grid is 
[DataTables](https://datatables.net/). If using Angular or React, you can't beat 
[Ag-Grid](https://www.ag-grid.com/) IMHO.

## Data Access and the Database

I want Tortis IAM to support multiple database engines for flexibility in deployment, so Entity Framework Core was a top
choice. Since Identity and OpenIddict both have first class support for it, it was an easy choice to make.

To keep development and deployment of a V1 as simple as possible, I started with SQLite. When I get to the
"Make It Fast" phase, I'll add support for SQL Server, Oracle, Postgres, and MySQL. Even SQLite will allow Tortis IAM
to scale vertically to thousands users of monolithic applications.

Why not document databases like MongoDB? That would require a lot more work and just wasn't in the budget. OpenIddict uses
[Quartz Scheduler](https://www.quartz-scheduler.net/) for its cleanup job(s). Quartz was the lowest common denominator
in choosing which databases to support. For my needs, SQL Server, Oracle, and Postgres are all that is needed.

## The Complete Day 1 Stack

- .Net/AspNet Core
- AspNet Core Identity
- Blazor for the UI
- FluentUI Blazor for the UI controls and theming
- Entity Framework Core for data access
- SQLite for the database

## License Considerations and Product vs Library

As I alluded to earlier, the aim here was a solution that was free (as in beer and as in speech) open source. The
initial holistic Tortis IAM "product" is under the 
[GNU Public License version 3 (GPL-3)](https://www.gnu.org/licenses/gpl-3.0.en.html#license-text).
This ensures the software freedom I'm looking for with this solution. I want anyone to be able to learn from the code,
modify it, and pay-it-forward. Non-restrictive licenses like MIT would allow someone to relicense the code under a
proprietary license and close off their version of the source.

I started by saying a replacement for Duende Identity Server was a catalyst. Since I'm not building my own
OpenId Connect library, that really isn't going to happen for the initial releases. However, the configuration of OpenIddict
is very flexible but not terribly straight forward. I may release a package that compresses that complexity into a few
opinionated one-liner registration scenarios, reducing a few hundred lines of OpenIddict config to one or two lines. If/
when that happens, it will likely be released under Apache-2.0.

## The Journey to "Make it work"

With those key decisions made, it was time to play with the tech a bit and cobble together a "working" solution. A step
beyond prototype to "can I run the app and do all the things required?", though perhaps not "production ready" and the
UI not as polished as it could be.

Fred Brooks said (paraphrasing), plan on writing software twice: once to figure it out and a second time to do it right.
That is the approach I took here. I was just trying to figure out how these libraries worked, how they would need to
interact if at all, how Blazor would need to interact with them, and for the future, how a Rest-ful API layer would need
to interact with them. Just figure it out and make it work. 
As such, the code is a bit of a mess. I'll tidy things up as part of making it right.

### The Good

OpenIddict "just worked" for the most part. AspNet Core Identity did as well except for a Blazor compatibility issue
I'll discuss in the next section. FluentUI Blazor is pretty clean and easy to work with. Their data grid worked
perfectly for my needs. Despite the bad and ugly things I needed to work around, all in all, I feel using Identity and
OpenIddict did save time and effort, allowing me to focus on the Blazor pieces.

### The Bad

#### Compatibility

I started with the simplest use-case: Login and get a hello world page.

Simple, right? Just create a new Blazor Server App with Identity and run it. Sigh. Not quite. Already I ran into some 
compatibility issues. Blazor's Interactive Server render mode doesn't play nice with AspNet Core Identity. It has to do
with the way Identity manages cookies during login, logout, and profile self-management tasks such as resetting
passwords and setting up MFA.

But I wanted to stick with Blazor, so I made it work. It did require separate layouts and ensuring that everything under
`/account` was static server-side rendering (SSR) instead. The challenge here is now you lose interactivity--you
basically just have normal Razor pages at that point. Luckily FluentUI Blazor has solutions for using some of their
controls in SSR; namely the profile control.

#### Hybrid Flow

It seems AspNet Core has its own auto-magical way of dealing with the Hybrid Flow and doesn't play nice with OpenIddict.
Kevin implemented the spec. Microsoft...well, they did Microsoft. The OpenId Connect middleware in AspNet Core only gets
claims from the `id_token`, not the `access_token`. They expect OIDC to only be used to identify the user and not return
any roles or scopes. That defeats the purpose of an IAM solution that is separate from the application itself, though
does fall in line with the "the application is responsible for authorization management" approach. The middleware can be
tweaked with event handlers to grab role claims off the `access_token`. 

Back to they Hybrid Flow. The only way to get the `id_token` is to ask for it. Unfortunately, a `code id_token` response
type triggers a hybrid flow. Microsoft and OpenIddict seem to be incompatible in how that dance happens. I haven't
reached out to Kevin yet since the middleware can automatically call the `/userinfo` endpoint to get the identity
claims, but I will be keen to getting hybrid working in the future. 


### The Ugly

#### Different API surfaces

Identity and OpenIddict handle commands differently and that made the UI code ugly. Identity uses the Result Pattern.
Every command in the Managers return an `IdentityResult`. In OpenIddict, commands either work or throw an exception. 
This made the approach to error handling in the Blazor components inconsistent. Creating my own UserManager, 
ApplicationManager, etc. that either extend or wrap Identity's and OpenIddict's managers as well as my own core models 
for User, Role, Application, etc., I can ensure consistency across the entire UI and API surface. Both libraries support
this type of extension/customization.

This abstraction isn't so much for making it easy to add features as it is to ensure consistency between UI and API
and if I do decide to replace Identity and/or OpenIddict in the future, it won't be as big a deal as all the Tortis IAM
specific logic will be in my managers. Yeah, I know YAGNI. But in this case, I think it is warranted. The justification
is a good user experience.

#### Quartz and SQLite

The Quartz SQLite provider uses System.Data.SQLite. EntityFramework Core uses Microsoft.Data.Sqlite. So, both must be
included.

#### Lots of Database Trips

And I mean lots. There is no reason to go to the database 8 times to login and create a claims principal. The problem
is in how Identity and OpenIddict data access works out of the box. This alone may be a reason to write my own once I
get to the "make if fast" stage. Speaking from experience with similar Identity Server 4 implementations, the
excessive database calls is going to kill performance and scalability.
Yes, I could cache the data, but that creates a whole other level of complexity just to fix someone else's less than
ideal data access design.

#### Entity Framework and Multiple Providers

EF migrations and multiple providers is a terribly painful and error prone experience. I will likely use 
[Fluent Migrator](https://fluentmigrator.github.io/) to handle database scripting.
This means defining the database model in code twice. Once in Fluent Migrator so it can generate the SQL and 
again in EF to map the entities. It also means having to reverse engineer each time there is a schema change from 
Identity or OpenIddict. Seems there is an opportunity for a power tool there--elegant multi-provider migration
support for EF. I may play around with something when the time comes.

For now, using SQLite, the application just creates the database itself if doesn't exist.

## Make it Right

My next task is to "make it right". The plan for this phase and a "v1" release is:

- Abstracting away Identity and OpenIddict so I have a consistent API surface.
- Clean up the UI. Focusing on consistency and stability.
- Make is secure:
  - Implement certificates
  - Encryption at rest for PII data
  - Client Secret Management. Currently, the secret is hashed (good), but I never show it to the user when creating the
    Application. There is currently also no way to generate a new client secret.
- Documentation

## Make if Fast

Once the first solid production ready release is out the door, I'll focus on performance and scalability.

## The Future

Future plans include things like client assertion support (in lieu of secrets), OpenId Connect Certification and
Financial API baseline certification. LDAP integration. OpenId Connect Federation.
