---
layout: post
title:  "Debugging a Jaeger memory issue"
date:   2021-02-12 00:00:00
categories: jaeger, tracing, debugging
comments: true
---

* Will be replaced with the ToC
{:toc}

At Auto Trader UK we use useu Jaeger as part of the Istio service mesh to collect & visualise HTTP request traces across the system.

Tracing provides a lot of value for us when debugging problems and understanding the behaviour of services across the system.

*DESIRED BEHAVIOUR:* when queue fills up traces/spans should be dropped

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