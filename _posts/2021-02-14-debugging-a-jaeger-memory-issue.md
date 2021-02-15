---
layout: post
title:  "Debugging a Jaeger memory issue"
date:   2021-02-14 00:00:00
categories: jaeger, tracing, debugging
comments: true
---

* Will be replaced with the ToC
{:toc}

At Auto Trader UK we use Jaeger as the tracing component within our Istio based service mesh to collect & visualise HTTP request traces for the services running on our GKE based delivery platform.

Tracing provides a lot of value for us when debugging problems and understanding the behaviour of services across the system.

Recently we experienced a problem with the Jaeger collectors sufffering from container restarts as they were consuming an unexpected amount of memory and being killed by the Kubernetes OOM killer.

We were able to correlate the failures to the unavailability of the Jaeger storage backend (in our case, Elasticsearch). 

Initial analysis of the problem showed the `collector` queues filling up which was as expected due the unavailability of the backend. 

What we expected to happen was that the queue would max out at which point traces events would start to be dropped. What actually happened was that before we were able to observe the queue max out the container would crash.

This was surprising as we had configured the Jaeger collector to have a maximum queue size of 80mb and an overall memeory request for the conatiner of 200mb. 

In theory, the max queue size setting should constrain the queue and the amount of memory it consumed. This, combined with the overall memory request of 200mb, should provide ample space for the the collector to be resilient to any scenario that could cause the queue to fill up (for example, the storage backend being unavailable or maybe an unexpected/unplanned increase in load i.e. traces per second).


# The Problem

- Architecture overview
  - Jaeger
    - collector
    - query
  - Elastic as the backend for storing traces and spans
- Jaeger config, queue size based on allowed memory usage
- ES connectivity issue
- container restarts due to K8s OOM killer
- what is the OOM killer?
- why/when does it kick in to kill a process?

# Investigation & understanding

- recreating the problem in a controlled environment
- queue
- understanding the memory usage in more detail
- Go heap profiling

# Solution

- Right sizing the memory request for Jaeger collector to ensure enough space for queue to grow to accomodate traces/spans including the additional space they require when queued
- link to Jaeger issue: <https://github.com/jaegertracing/jaeger/issues/2715>

# References