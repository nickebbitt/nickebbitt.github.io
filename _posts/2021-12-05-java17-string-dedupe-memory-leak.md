---
layout: post
title:  "The story of a Java 17 native memory leak"
date:   2021-11-05 00:00:00
categories: jaeger, tracing, debugging
comments: true
---

## Discovery

When Java 17 was released in September we were fairly quick to provide a new Docker base image to allow our devevlopers to gain the benefits of the new goodness in the JDK available since Java 11, the previous LTS version.

String Dedupe as a default for all workloads.

Great monitoring stack at AutoTrader. JMX exporter.

https://twitter.com/nickebbitt/status/1453284230964912153?s=20

## Trial & Error

Lot's of trail and error, different JDKs, options etc.

GraalVM was ok, HotSpot had the leak.

Great CI/CD process allowing safe experimentation.

Slow feedback loop.

## Progress

Tom Edge exposed custom native memory metrics, spotted that String table was growing.
Disabled StringDedeup and the leak went away.

https://twitter.com/nickebbitt/status/1465617649547849733?s=20

## The bug & the fix
Tweeted, Shipilev tagged, investigated and then fixed in a matter of hours.

https://bugs.openjdk.java.net/browse/JDK-8277981

https://github.com/openjdk/jdk/pull/6613 


## Verify
Installed the nightly build of `openjdk17u`, deployed for one of the affected services with StringDedupe enabled.

Verified that the leak was gone.

## The future

In January we should have access to Java 17.0.2 at which point we will patch our base image and roll it out, re-enabling StringDedeup.

https://twitter.com/shipilev/status/1466317208917889025?s=20