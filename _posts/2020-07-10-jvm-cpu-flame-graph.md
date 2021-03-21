---
layout: post
title:  "CPU Flame Graphs on the JVM"
date:   2020-07-10 00:00:00
categories: jvm, performance, cpu, debugging
comments: true
---

## what? / why?

## Options

https://github.com/jvm-profiling-tools/async-profiler

## Example with a containerised workload

```bash
docker exec -it b5bc125cdb61 /usr/local/async-profiler/profiler.sh -d 30 -o collapsed -e itimer -f /tmp/collapsed.txt 1
```

## Security

https://github.com/jvm-profiling-tools/async-profiler#profiling-java-in-a-container
https://docs.docker.com/engine/security/seccomp/