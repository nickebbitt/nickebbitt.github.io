---
layout: post
title:  "Debugging a Jaeger memory issue"
date:   2021-02-14 00:00:00
categories: jaeger, tracing, debugging
comments: true
---

* Will be replaced with the ToC
{:toc}

At [Auto Trader UK](https://careers.autotrader.co.uk/) we use [Jaeger](https://www.jaegertracing.io/) as the distributed tracing component within our [Istio](https://istio.io/latest/docs/concepts/what-is-istio/) based service mesh to collect & visualise HTTP request traces for the services we run on our [GKE](https://cloud.google.com/kubernetes-engine) based delivery platform.

We have around 400 (micro)services running in production and [distributed tracing](https://microservices.io/patterns/observability/distributed-tracing.html) provides a lot of value for us when debugging problems and understanding behaviour of requests across the system.
As mentioned previously, we also operate Istio on the cluster which is the source of request traces.
Every HTTP request to a service running on the cluster is ingressed via NGINX and enters the service mesh and a trace for that request begins.
A trace span is generated for every connection between the Istio side-car proxies (i.e. Envoy proxies) deployed alongside every service that are intercepting and handling HTTP based traffic across the cluster.

Jaeger consists of 3 core components:

* `agent` - listens for span sent over UDP, batches them and sends to the `collector`
* `collector` - receives traces from `agents` and runs them through a processing pipeline. Ultimately the `collector` is responsible for storing traces in a backend data store, in our case Elasticsearch.
* `query` - retrieves traces from storage to display in the UI

There is also the front-end/UI which is used to visualises traces for end users.

More details of Jaeger's architecture is available in their [docs](https://www.jaegertracing.io/docs/1.21/architecture/).

## The problem

With this context let's describe the problem we have been experiencing with the Jaeger collectors.  sufffering from container restarts as they were consuming an unexpected amount of memory and being killed by the Kubernetes OOM killer.

We were able to correlate the container restarts to the unavailability of Jaeger's storage backend i.e. Elasticsearch.
We discovered that during an Elasticsearch upgrade we were experiencing a period of down time

Initial analysis of the problem showed the `collector` queues filling up which was as expected due the unavailability of the backend. 

What we expected to happen was that the queue would max out at which point traces events would start to be dropped. What actually happened was that before we were able to observe the queue max out the container would crash.

This was surprising as we had configured the Jaeger collector to have a maximum queue size of 80mb and an overall memeory request for the conatiner of 200mb. 

In theory, the max queue size setting should constrain the queue and the amount of memory it consumed. This, combined with the overall memory request of 200mb, should provide ample space for the the collector to be resilient to any scenario that could cause the queue to fill up (for example, the storage backend being unavailable or maybe an unexpected/unplanned increase in load i.e. traces per second).



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