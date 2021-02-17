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
Every HTTP request to a service running on the cluster is ingressed via [ingress-nginx](https://github.com/kubernetes/ingress-nginx) and enters the service mesh at which point a trace for that request begins.
A trace span is generated for every connection between the Istio side-car proxies (i.e. [Envoy proxies](https://istio.io/latest/docs/ops/deployment/architecture/#envoy)) deployed alongside every service that are intercepting and handling HTTP based traffic across the cluster.

Jaeger consists of 4 main components:

* `agent` - listens for span sent over UDP, batches them and sends to the `collector`
* `collector` - receives traces from `agents` and runs them through a processing pipeline. Ultimately the `collector` is responsible for storing traces in a backend data store, in our case [Elasticsearch](https://www.elastic.co/elasticsearch/)
* `query` - retrieves traces from storage to display in the frontend/UI
* frontend/UI - used to query and visualise traces for end users

More details of Jaeger's architecture is available in their [docs](https://www.jaegertracing.io/docs/1.21/architecture/).

## The problem

With this context let's describe the problem we have been experiencing with the `collector`.

We observed the container for the `collector` unexpectedly restarting.
With some basic debugging (e.g. using `kubectl describe pod`) it turned out that the reason for this was they were being restarted by the Kubernetes OOM killer.

We were able to correlate the restarts to the unavailability of Jaeger's storage backend i.e. Elasticsearch.
We discovered that during an Elasticsearch upgrade we were experiencing some unexpected down time.
The unavailability of Elasticsearch meant that the `collector` was unable to send its trace events to the storgae backend.

This, in theory, shouldn't be a problem.

The `collector` is designed to handle the unavailability of the storage backend using an internal queue that buffers trace events when it is unable to store them. If the queue fills up then the `collector` will drop the oldest trace events and should continue working in this way until it is able to send trace events again.

Under normal conditions we expect the `collector`'s queue to near empty at all times. This shows us that we are able to save trace events to storage at a rate that keeps up with the volume being generated across the system.

There are a couple configuration options that can be used to control the size of the `collector`'s queue:

* `collector.queue-size`: limits the queue size by an absolute maximum number, for example, 50,000 spans
* `collector.queue-size-memory`: limits the queue size based on a calculation involving an amount of memory and the average span size, calcualted at runtime

In principal, the `collector.queue-size-memory` option sounds great and we decided to go with it as it allowed us to control the resources that instances of the `collector` would use regardless of the size of the trace spans.

So when we understood that the `collector` container restarts correlated with their queue filling up due to the unavailability of the storage backend, it challenged our expectations of how the `collector` should behave in this scenario.
The actual memory usage did not appear to align with how we had configured the Jaeger `collector`, which was to have a maximum queue size of `80mb` and an overall memory request for the container of `200mb`.

We expected that when the `collector` queue filled up then memory usage would be constrained to the value we configure it for, and for trace spans to be dropped when the queue was full.
What actually happened was that before we were able to observe the queue being full and trace spans being dropped the OOM killer would have already done its job.

In theory, the max queue size setting of `80mb` should constrain the queue and the amount of memory it consumed.
This, combined with the overall memory request of `200mb`, should provide ample space for the the collector to be resilient to any scenario that could cause the queue to fill up with extra space for the general overheads of the `collector` process.

# Investigation & understanding

- recreating the problem in a controlled environment
- queue
- understanding the memory usage in more detail
- Go heap profiling

# Solution

- Right sizing the memory request for Jaeger collector to ensure enough space for queue to grow to accomodate traces/spans including the additional space they require when queued
- link to Jaeger issue: <https://github.com/jaegertracing/jaeger/issues/2715>

# References