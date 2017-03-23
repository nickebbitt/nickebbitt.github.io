---
layout: post
title:  "Asynchronous web service using CompletableFuture"
date:   2017-03-22 00:00:00
categories: java, springboot
---

Until recently I've not spent a lot time looking into the enhancements provided in
`java.util.concurrent` as part of the Java 8 release. This has been predominantly due to
the fact I've been using the [Akka](http://akka.io/) toolkit to handle any concurrency
concerns within the Java app I've been working on.

The app in question uses the [Spring framework](https://projects.spring.io/spring-framework/) and exposes a number of "RESTful" HTTP endpoints implemented using a Spring [`@RestController`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html) and the [`DeferredResult`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/context/request/async/DeferredResult.html) class. This provides a simple way to create an asynchronous service where a request can be handled concurrently on a thread of implementers choice. In our case things then get slightly more complicated as, rather than submit the request to a thread pool or the like for processing, we forward the `DeferredResult` to an actor. A result is provided for the `DeferredResult` at some point in the future by an actor that uses a thread managed by the actor system.

This approach works fine but the level of complexity in the implementation isn't desirable for a couple of reasons. One is that the bridge into the actor system makes it more difficult to test. There is also the additional maintenance overhead caused by the code being more difficult to understand and reason about at the seams between the actor/non-actor code.

Then I saw the following tweet from Jamie Allen (former employee of [Lightbend](https://www.lightbend.com/), stewards of Akka)...

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Really, if you’re using Akka purely for simpler local concurrency, you’re doing it wrong. That’s for Futures; much more composable, &amp; typed.</p>&mdash; Jamie Allen (@jamie_allen) <a href="https://twitter.com/jamie_allen/status/838932419264700417">March 7, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

...and it got me thinking more about the approach we had used and whether the standard concurrency utilities available in Java could provide a much simpler alternative to our current design based on `DeferredResult` and Akka.

### A bit about CompletableFuture...

One of the enhancements to the standard concurrency utilities that arrived in Java 8 comes in the form of the
[`CompletableFuture`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)
class. `CompletableFuture` implements the [`Future`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html)
interface (available since Java 5) that provides the result of an asynchronous computation. It also implements the [`CompletionStage`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html)
interface that defines the contract for "...a stage of possibly asynchronous computation, that
performs an action or computes a value when another `CompletionStage` completes."

> In short, the `CompletableFuture` class provides us with a way to compose, combine and execute
asynchronous computation steps.

I'm not going to go into detail about `CompletableFuture` however the following video from Tomasz Nurkiewicz provides a great primer for understanding what it offers.

<iframe width="560" height="315" src="https://www.youtube.com/embed/-MBPQ7NIL_Y" frameborder="0" allowfullscreen></iframe>


What I'm specifically interested in is how we can use `CompletableFuture` in combination with a Spring [`@RestController`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html) to implement HTTP endpoints that handle requests asynchronously.

Given a method that simulates some time consuming computation...

{% highlight java %}
private String processRequest() {
    log.info("Start processing request");
    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    log.info("Completed processing request");
    return RESULT;
}
{% endhighlight %}

...the following examples demonstrate a few different approaches to handling requests.

### A synchronous example

Here we have a HTTP endpoint that the handles a request synchronously.

{% highlight java %}
@RequestMapping(path = "/sync", method = RequestMethod.GET)
public String getValueSync() {

  log.info("Request received");

  return processRequest();

}
{% endhighlight %}

This generates the following log output...

{% highlight bash %}

21:21:19.204 [                main] Before request
21:21:19.347 [http-nio-auto-1-exec-1] Request received
21:21:19.347 [http-nio-auto-1-exec-1] Start processing request
21:21:24.351 [http-nio-auto-1-exec-1] Completed processing request
21:21:24.374 [                main] After request

{% endhighlight %}

The key thing to note here is that all processing occurs on the same HTTP thread (`http-nio-auto-1-exec-1`). This means that for the duration of the request the HTTP thread is blocked and unavailable to serve other requests made to the system. This is undesirable due to the potential for the servlet container's thread pool to be exhausted when an app is under heavy load thus impacting an apps scalability.

### An asynchronous example using DeferredResult

Here we have a HTTP endpoint that handles a request asynchronously via Spring's `DeferredResult`. As mentioned earlier, this has been my go to approach for implementing asynchronous behaviour in a RESTful HTTP service.

{% highlight java %}
@RequestMapping(path = "/asyncDeferred", method = RequestMethod.GET)
public DeferredResult<String> getValueAsyncUsingDeferredResult() {

    log.info("Request received");

    DeferredResult<String> deferredResult = new DeferredResult<>();

    ForkJoinPool.commonPool()
            .submit(() -> deferredResult.setResult(processRequest()));

    log.info("Servlet thread released");

    return deferredResult;

}
{% endhighlight %}

This generates the following log output...

{% highlight bash %}

21:21:24.418 [                main] Before request
21:21:24.421 [http-nio-auto-1-exec-2] Request received
21:21:24.427 [http-nio-auto-1-exec-2] Servlet thread released
21:21:24.427 [ForkJoinPool.commonPool-worker-1] Start processing request
21:21:29.429 [ForkJoinPool.commonPool-worker-1] Completed processing request
21:21:29.448 [                main] After request

{% endhighlight %}

The key difference to note here is that, unlike the synchronous endpoint, the request processing occurs on a different thread (`ForkJoinPool.commonPool-worker-1`). The HTTP thread (`http-nio-auto-1-exec-2`) is released pretty much immediately making it available to serve additional requests. The actual request processing is performed at some later point in time on a different thread. For demonstration purposes I've use the standard [`ForkJoinPool`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html) thread pool however in practice you would likely create and configure a suitable thread pool based on your use-case.

### An asynchronous example using CompletableFuture

Finally we have a HTTP endpoint that handles a request asynchronously using Java 8's `CompletableFuture`.

{% highlight java %}
@RequestMapping(path = "/asyncCompletable", method = RequestMethod.GET)
public CompletableFuture<String> getValueAsyncUsingCompletableFuture() {

  log.info("Request received");

  CompletableFuture<String> completableFuture =
      CompletableFuture.supplyAsync(this::processRequest);

  log.info("Servlet thread released");

  return completableFuture;

}
{% endhighlight %}

This generates the following log output...

{% highlight bash %}

21:21:29.453 [                main] Before request
21:21:29.456 [http-nio-auto-1-exec-4] Request received
21:21:29.458 [http-nio-auto-1-exec-4] Servlet thread released
21:21:29.458 [ForkJoinPool.commonPool-worker-1] Start processing request
21:21:34.463 [ForkJoinPool.commonPool-worker-1] Completed processing request
21:21:34.468 [                main] After request

{% endhighlight %}

The result is pretty much identical to the `DeferredResult` example in that the request processing occurs on a different thread. It's worth noting that this example also uses the `ForkJoinPool` due it being the default when no explicit thread pool is provided to the `supplyAsync` method.

`CompletableFuture` has the advantage over `DeferredResult` due to the various composable methods that it provides. This allows for more complex chains of processing to be defined that are completely asynchronous.

The following example shows a simple chain of asynchronous processing...

{% highlight java %}
@RequestMapping(path = "/asyncCompletableComposed", method = RequestMethod.GET)
public CompletableFuture<String> getValueAsyncUsingCompletableFutureComposed() {

    return CompletableFuture
            .supplyAsync(this::processRequest)
            .thenApplyAsync(this::reverseString);

}
{% endhighlight %}

## In summary...

The examples in this post demonstrate a couple of different approaches to creating an asynchronous HTTP endpoint.

The simplicity of `CompletableFuture` makes it a no-brainer to use in place of `DeferredResult` due to the additional power and flexibility that comes with it.

The next step to investigate further how `CompletableFuture` can integrate with, if not replace, some of the actor based functionality currently in use.

---

All code from this post is available in [GitHub](https://github.com/nickebbitt/rest-async-completable-future).
