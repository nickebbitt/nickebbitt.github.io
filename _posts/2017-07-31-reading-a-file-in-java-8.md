---
layout: post
title:  "Reading a file in Java 8"
date:   2017-07-31 00:00:00
categories: java, files
comments: true
---

I always forget how to read the contents of a file, here's the simplest way I've come across.

{% highlight java %}
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;

public static void main(String[] args) throws IOException {
    List<String> content = Files.readAllLines(Paths.get("input.txt"));
}
{% endhighlight %}

There are other variants of `Files.readAllLines` such as `Files.readAllBytes` etc.
