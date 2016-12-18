---
layout: post
title:  "Making a mockery of web services"
date:   2016-12-17 00:00:00
categories: java
---
Recently I gave my first public talk at the Manchester Java Community (MJC). This blog provides the detail behind the [slides](https://speakerdeck.com/nickebbitt/making-a-mockery-of-web-services).

# Abstract

With the help of [WireMock](http://wiremock.org/) we will explore how to create reliable and repeatable integration/system tests for an application that depends on an external web service or HTTP-based API.

## The Problem

Until around 5 years ago all my experience was heavily focused in the client/server world of Oracle RDBMS and Oracle Forms. I then started working on web applications that followed a more three tiered architecture. These applications have generally followed the same pattern, a database (usually Oracle) exposed to the public internet via a HTTP-based web service that is consumed by a HTML/JavaScript based user interface.

The most recent system of this nature that I have worked on was a mobile platform to provide integration between mobile devices and a "backend" HTTP-based web service. The mobile platform was cloud based exposing HTTP endpoints to be consumed by the mobile clients.

We chose to architect the system to allow us to follow a Continuous Delivery (CD) approach using blue/green deployments. To allow us to deliver the system using this way it was essential that we could run automated acceptance tests.

The remainder of this blog explores the tools & techniques used to approach this problem.

## Define: Mock

First off we'll cover some of the core ideas related to testing.

According to [Google](https://www.google.co.uk/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=define%3A%20mock), to _mock_ something is to...

>  ...make a replica or imitation of something.

This definition sets us on the right path to understanding why we may want to mock something.

In relation to software engineering and specifically object oriented (OO) programming, [Martin Fowler](http://martinfowler.com/articles/mocksArentStubs.html) suggest that mocks are...

> ...special case objects that mimic real objects for testing.

I think it's important to not just

### Why use mocks?

### When to mock?

### Mocking web services

## Introducing WireMock

### Deployment

### Key features

## Conclusion
