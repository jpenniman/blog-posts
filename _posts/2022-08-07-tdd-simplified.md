---
layout: post
title: Test Driven Development (TDD) Simplified
date: 2022-08-07
image: /blog/images/tdd.png
author: Jason M Penniman
excerpt: Test Driven Development (TDD) is a development/design approach in which we write our tests first, then implement the code to make these tests pass. These initial tests we write are not unit tests in the traditional sense. I'll repeat that... TDD tests are not unit tests.
tags:
- dotnet
- C#
- TDD
---
Test Driven Development (TDD) is a development/design approach in which we write our tests first, then implement the code to make these tests pass.

## What TDD is Not

These initial tests we write are not unit tests in the traditional sense. I'll repeat that: TDD tests are not unit tests.

## What are they then?

TDD tests are specification tests. They are an example of how to use the thing being built. Sure, we'll use a "unit testing" framework to build, them, but don't get hung up on terminology. The idea is, we're testing, and showing an example of, the API, not testing the logic of an individual piece of the several pieces that make up the implementation of that API.  Let's look at a simple example:

Let's take a feature and a scenario: Vendor Lookup

``` text
Feature: Users can search for Vendors in the system.

Scenario: Search by a company name beginning with the search term.
	Given the following vendors
		| CompanyName |
		| Acme Supply |
	When the company name begins with the term "Acme"
	Then then 1 record should be returned: "Acme Supply".
```

We know we need to build a component with an API that allows us to search for a vendor whose company name starts with the term provided. What should this API look like? How do we want developers to interact with this API? What type should it return: the domain entity? a summarized projection? What collection type should it return: an array? a List? an observable stream? Do we want an interface or just a class implementation? Let's write a test to answer those questions.

```csharp
IVendorLookup lookup = new VendorLookup();
Vendor[] vendors = lookup.FindByCompanyNameStartingWith("Acme");

vendors.Length.Should().Be(1);
vendors[0].CompanyName.Should().Be("Acme Supply");
```

Simple enough. It certainly tests our feature scenario and gives us a feel for now a developer will interact with this API. It also raises some discussion points (and is why I like pair or mob programming):  Here we chose to return the entire vendor entity. Do we really want that? Should it be a summarized projection--e.g., just the Id, CompanyName, and a PhoneNumber? Should we limit the number of results--what happens if this returns 1,000's of vendors? If we decide to page the results, how do we convey that to the developer and ultimately the user? Given this will ultimately involve an IO operation of some kind, should the API be non-blocking (async)?

We can make some design decisions as a team and iterate over the test:

``` csharp
string term = "Acme";
int skip = 0;
int take = 100;
IVendorLookup lookup = new VendorLookup();
PagedVendorLookupResult pageOfResults = await lookup.FindByCompanyNameStartingWithAsyc(term, skip, take);

pageOfResults.TotalCount.Should().Be(1);
pageOfResults.VendorLookupResults[0].CompanyName.Should().Be("Acme Supply");
```

There. We have a pretty good initial API spec...er...TDD test.  Now we can implement our component. When we do, additional things will come up that will require us to come back and write more tests to cover more cases for this scenario:

- What do we want to happen if the search term is null or empty?
- Should the page size be constrained, and to what? What happens when a value outside that range is provided?
- What if no vendors match the criteria?

These answers will require collaboration with the customer.

The tests for these will feel a bit more like unit tests, as we are testing specific code paths within our find method, but we're still really testing the specification: a null search term should throw an AgrumentNullException.

## Taking it a step further

In this example I kept it simple.  We just looked at it from the service component interface. The "API" example being spec'd out and tested can just as easily have been a REST-ful API or GraphQL or gRPC service. Or the test as we wrote it could be against a client library that makes that backend call, providing a more integration-style test where we are testing the protocol/anti-corruption layer in our tests as well.

The goal of these tests, like any test, it to give us confidence that our code works without any known bugs and the confidence to make changes--e.g., optimizations--to the code and know we didn't break anything.

## A side note on agile and rapid feedback

As you can see, TDD allows us to raise questions about design early on. It also fosters a much more agile culture. The initial implementation of the happy path for our customer lookup could easily be built and demo-able in a running (*cough* production) environment in just a couple hours so we can get that feedback we need to answer the additional questions.

One of the best customers/Product Owners I ever worked with said: "I don't know what it needs to look/work until you show me." Her approach meant we collaborated constantly. We would show her the work several times a day and get that rapid feedback. The result was high a higher quality product, offering only value and no waste, no known bugs, and a significantly shorter time-to-market.
