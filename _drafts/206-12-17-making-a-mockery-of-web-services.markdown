---
layout: post
title:  "Making a mockery of web services"
date:   2016-12-17 00:00:00
categories: java
---
In October 2016 I gave my first public talk at the Manchester Java Community (MJC). This blog provides the detail of what I discussed to support the [slides](https://speakerdeck.com/nickebbitt/making-a-mockery-of-web-services).

# Abstract

With the help of [WireMock](http://wiremock.org/) we will explore how to create reliable and repeatable integration/system tests for an application that depends on an external web service or HTTP-based API.

## The Problem

Until around 5 years ago all my experience was heavily focused in the client/server world of Oracle RDBMS and Oracle Forms. I then started working on web applications that followed a more three tiered architecture. These applications have generally followed the same pattern, a database (usually Oracle) exposed to the public internet via a HTTP-based web service that is consumed by a HTML/JavaScript based user interface.

The most recent system of this nature that I have worked on was a mobile platform to provide integration between mobile devices and a "backend" HTTP-based web service. The mobile platform was cloud based exposing HTTP endpoints to be consumed by the mobile clients.

We chose to architect the system to allow us to follow a Continuous Delivery (CD) approach using blue/green deployments. To allow us to deliver the system using this way it was essential that we could run automated acceptance tests to provide us with the confidence that we had a system that was always releasable.

The remainder of this blog explores the tools & techniques used to approach this problem.

## Define: Mock

First off we'll cover some of the core ideas related to testing.

According to [Google](https://www.google.co.uk/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=define%3A%20mock), to _mock_ something is to...

>  ...make a replica or imitation of something.

This definition sets us on the right path to understanding why we may want to mock something.

In relation to software engineering and specifically object oriented (OO) programming, [Martin Fowler](http://martinfowler.com/articles/mocksArentStubs.html) suggest that mocks are...

> ...special case objects that mimic real objects for testing.

I think it's important to consider mocks more generally than just in applied to objects in the OO sense. A mock can be used for various types internal or external dependencies of a software application given the right tools & techniques for the job.

### Why use mocks?

Mocks help us to write tests that are deterministic and repeatable. They provide a way of controlling the behaviour collaborative dependencies that the object (or system) under test is interacting with.

This control allows us to model the various scenarios or use-cases necessary to prove that the software meets the acceptance criteria.

_NOTE: I'm aware of some debate as to whether you should use mocks for testing or not.  All I know is they have proved useful to me when writing tests._

### When to mock?

It is likely that during the development of an application you will be writing different types of tests with the intention of increasing the confidence that what you are producing is correct.

This will usually start with unit tests in which you’ll be aiming to test a single unit of your application such as an instance of a single class. If the object you are testing collaborates with another object then using a design pattern such as [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) and a mocking framework (e.g. [Mockito](http://site.mockito.org/)) you will be able to inject a mock of the collaborator into the unit you are testing.

Similarly, if you are testing at a higher level and looking to prove the correctness of a subsystem or the application as a whole then it may be desirable to mock the application’s external dependencies such as the database, a queue, or a web service.

It’s important to note I believe mocks are very useful but are not a replacement for testing against the “real” thing. This is still necessary, whether automated or manual, and should definitely be part of an overall testing strategy.

### Mocking web services

It’s quite common for an application to depend on an external web service. The external service could be external to the company and controlled by a 3rd party e.g. Twitter.

It could be a separate service from within the same organisation but controlled by a different team. It is quite common for micro-service architectures to communication over HTTP.

## Introducing WireMock

There are various tools or frameworks available that support the mocking of HTTP based APIs or web services however I’m going to focus on WireMock that we have used to good effect to support our development and automated integration testing processes within Tracsis.

WireMock was created a few years ago by Tom Akehurst, a London based developer.

The framework is open source which appealed to us as development team and is quite mature at version 2.

The documentation on the website is pretty good, contains lots of useful info and examples.

### Deployment

In it’s simplest form, WireMock comes as a runnable JAR that can be started from the command line. This mode proves to be very useful during development to provide a reliable web service running on your local machine to develop against.

WireMock can also be deployed to a servlet container if that’s your preference.

There is a comprehensive Java API from which you can created an embedded WireMock server and configure as required.

Finally, and more interesting from a Java testing perspective, WireMock provides Junit integration using @Rules.

### Key features

In regards to the feature set of WireMock...

Stubbing allows you to pre-define a canned response that will be served when a request is made matching a specific URL pattern.

Verifying allows you to prove that your application interacts with the external service in the way that you require it to.

WireMock can be used as a proxy.

The ability to record and playback interactions with the external service is pretty cool. It is a great way to create a base set of request mappings for use in development or with your tests.

The ability to simulate faults is very useful, particularly when it comes to testing the resilience of your application. One example would be to add a delay to the interaction with the external service to prove that your app behaves as expected.

Stateful behaviour provide a way to interact with the mock service and have it alter it’s state based on the interactions made. It acts as a state machine that can move through various states during a test to allow you to model more complex scenarios.

## Conclusion

In conclusion, through my experience of using WireMock we’ve been able to...

 * reduce dependency on the real service being available - it’s important to mention however that while tools like this are great, they don’t completely replace the need to test against the real thing.
 * improve the testability of the application, allowing us to create new tests for new features with less effort
 * and therefore ultimately we have been able to improve the quality of product we are delivering.

An additional benefit has been an increase in developer productivity due to not being reliant on the “real” service being available and working as expected.
