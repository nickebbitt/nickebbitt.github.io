---
layout: post
title:  "Solving a slow thread leak on the JVM using Java Flight Recorder (JFR)"
date:   2021-06-23 00:00:00
categories: jvm, threads, debugging
comments: true
---

One of our product teams at Auto Trader UK was experiencing a problem with a service whereby after a number of days of running it would suffer from high memory utilisation.

![high-memory](/assets/jvm-thread-leak/high-memory.png){:class="img-responsive"}

Services are deployed to our Kubernetes based platform and we calculate memory utilisation by looking at the RSS for the process as a percentage of the memory `request` specified in Pod specification for an application's container.
This allows us to monitor and alert on situations where a workload is more resources than planned for.

Often this is normal and the workload is actually underspecified for its purpose.
Sometimes though, as in this case, it's a signal that something isn't quite right with the service and it warrants further investigation.

Due to the nature of the problem being slow to materialise the cause had proven difficult for the application owners to track down.

As this was a Java service they'd examined the JVM metrics provided as standard for all deployed Java services deployed to the platform.
The metrics showed a fairly significant increase in memory outside of the JVM's heap memory area.

Something the really stood out was that the thread count had grown from around 250 or so following its initial deployment to over 1000 after a couple of days.

![thread-leak](/assets/jvm-thread-leak/thread-leak.png){:class="img-responsive"}

This aligned with the fact that the memory growth was sitting outside of the heap as the existence of each thread, even if idle, would account for some memory due to the overheads associated with it simply existing.
The service was running with Java 11 for which the default stack size for a JVM thread is `1024kb` so as the number of threads increases so does the memory overall memory footprint of the process.
We in fact override the stack size (using the `Xss` JVM option) setting it to `512kb` to reduces the memory footprint for our services.

The owners had some theories about what might be causing the growth in threads based on how an `ExecutorService` was being used to for a recently introduced feature.
Initial attempts to validate these theories hadn't been successful though.

Analysing a thread dump taken from the JVM process only served to confirm the fact that a large number of threads existed but, as they were all sat in an `WAITING` state, the stack traces for them contained nothing useful.

```
"pool-987-thread-1" #8237 prio=5 os_prio=0 cpu=5.36ms elapsed=18570.97s tid=0x00007f517c97d800 nid=0x1cb6a waiting on condition  [0x00007f512e67a000]
   java.lang.Thread.State: WAITING (parking)
  at jdk.internal.misc.Unsafe.park(java.base@11.0.11/Native Method)
  - parking to wait for  <0x00000000ac637bc0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
  at java.util.concurrent.locks.LockSupport.park(java.base@11.0.11/LockSupport.java:194)
  at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@11.0.11/AbstractQueuedSynchronizer.java:2081)
  at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@11.0.11/ScheduledThreadPoolExecutor.java:1170)
  at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@11.0.11/ScheduledThreadPoolExecutor.java:899)
  at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@11.0.11/ThreadPoolExecutor.java:1054)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@11.0.11/ThreadPoolExecutor.java:1114)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@11.0.11/ThreadPoolExecutor.java:628)
  at java.lang.Thread.run(java.base@11.0.11/Thread.java:829)
```

Similarly, a heap dump wasn't useful in this case as the memory growth wasn't related to objects the heap, and the information related to threads in the heap dump just gave the equivalent of that in the thread dump.

This was when we decided to reach for a tool available in the JDK called the JDK Flight Recorder (JFR).

JFR has been around for a good while as part of the Oracle JDK but was open sourced in 2018 and ships with OpenJDK 11.

> JFR is a low overhead profiler for the JVM that is able to capture lots of useful data about the JVM process to assist with understanding what it is doing.

We allow teams to enable JFR for their services via a feature toggle that will configure the JVM to perform a continuous recording for a rolling 10 minutes window.
The mechanism and configuration for this is baked into our base Java docker image that all JVM workloads on the delivery platform are built upon.

Enabling the feature for a service makes a Java Flight Recording action available for all deployed pods via its service dashboard.

![dashboard-jfr-action](/assets/jvm-thread-leak/dashboard-jfr-action.png){:class="img-responsive"}

A service owner can then request a recording for a running pod.
This triggers a workflow to take a dump of the recording from the pod, upload it to Google Cloud Storage (GCS) and then send a Slack direct message to the requester with a secure download link.

The recording can then be downloaded and loaded into JDK Mission Control (JMC) where it can be analysed to gain insights into what the JVM has been up to.

JFR is event based and JDK Mission Control provides different ways to visualise the events depending on their type.
There's many event types but the one's of interest for us were related to threads.

Focusing in on the Threads view in JMC we were quickly able to identify the threads of interest.

![jmc-thread-flame-graph](/assets/jvm-thread-leak/jmc-thread-flame-graph.png){:class="img-responsive"}

## Configuring JFR

When setting this up for Auto Trader's Java workloads I took inspiration from [this article](https://foojay.io/today/how-when-to-use-jdk-flight-recorder-in-production/) published on [foojay.io](https://foojay.io/).

The advice given was to configure 2 recordings, one for the initial start-up process of the JVM and another to cater for post start-up.
This resulted in the following options being set when executing the Java process.

```bash
-XX:StartFlightRecording=duration=6m,name=app-startup,dumponexit=true,filename=$JFR_DUMP_DIR/app-startup.jfr
-XX:StartFlightRecording=delay=5m,maxage=10m,name=post-startup,dumponexit=true,filename=$JFR_DUMP_DIR/post-startup.jfr
-XX:FlightRecorderOptions=stackdepth=96
```
