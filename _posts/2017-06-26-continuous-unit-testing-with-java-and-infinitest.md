---
layout: post
title:  "Continuous unit testing with Java & Infinitest"
date:   2017-06-26 00:00:00
categories: java, testing, tdd
comments: true
---

* Will be replaced with the ToC
{:toc}

As my awareness around automated testing, and particularly TDD, has risen over the last few years I've been keeping my ear to the ground for new ideas and techniques I can experiment with.

I was recently at a [meetup](https://www.meetup.com/Expert-Talks-Manchester/events/238434029/) hosted by Equal Experts and there was a talk by [Oliver Shaw](https://twitter.com/olly_shaw) on what "Doing TDD" is, and isn't. The talk provided a great live demo of TDD exploring a fairly simple use-case but showing how natural the TDD process can be. The demo was done using Scala and he used Vim as his IDE which was impressive and very effective.

The relevance to this post is that he was using an [SBT](http://www.scala-sbt.org/) task ([`~test-quick`](http://www.scala-sbt.org/0.13/docs/Testing.html#testQuick)) to continuously watch for file changes and automatically run the unit tests against them when they changed. This allowed him to remain focused on the code and the cycles involved in TDD rather than breaking flow through repeatedly having to manually run the tests.

I like the idea of this! Anything that can help to keep focus and make the process more efficient is worth a try.

# What is continuous unit testing?
_Continuous unit testing_ is basically the idea that every time you make a change to your code, be that functional or test code, the tests that are associated with that change will automatically run. This is not be confused with [Continuous Testing](https://en.wikipedia.org/wiki/Continuous_testing) as part of a build pipeline.

> _Continuous unit testing_ tools or frameworks use the concept of a _test watcher_.

The _test watcher_ runs in the background and watches for changes on the source files you are working on. When a file changes the _test watcher_ triggers the execution of any related tests automatically.

The standard TDD cycle, as per [Wikipedia](https://en.wikipedia.org/wiki/Test-driven_development#Test-driven_development_cycle), is:

 * Add a test
 * Run all tests to see if the new test fails
 * Write the code
 * Run tests
 * Refactor code
 * Repeat

Using a _continuous unit testing_ approach, the cycle becomes:

 * Add a test _(tests run automatically and see new test fail)_
 * Write the code _(tests run automatically and hopefully see all tests pass)_
 * Refactor _(tests run automatically and hopefully continue to see all tests pass)_
 * Repeat

Each time the _tests run automatically_, any failures are brought to the developers attention to be dealt with immediately. A fix can be applied, or the change that caused the failure reverted, at which point the tests automatically run again until you eventually get to the position of all tests passing.

The _continuous unit testing_ approach has reduced the number of steps (not including _Repeat_) in the TDD cycle from 5 to 3.

Some benefits of this are:

 * reduced context switching due to not having to manually run the tests
 * shorter cycles (feedback loops) between making a change and finding if a test has passed, or failed

There are various tools available depending on your language of choice. The natural choice for Scala is likely SBT as mentioned above. This can also be used for Java based projects however another option that integrates with my IDE of choice, IntelliJ, is [Infinitest](https://infinitest.github.io/).

# Infinitest
_Infinitest_ provides plugins for both Eclipse and IntelliJ. We will look at how it works with IntelliJ for the purposes of this post.

The project's website describes what _Infinitest_ does very nicely:

> Each time a change is made on the source code, Infinitest runs all the tests that might fail because of these changes.

The [documentation](http://infinitest.github.io/doc/user_guide.html) is pretty brief but from my experience it covers everything you need to know to get started and configure it in various ways.

## IntelliJ Setup

The first thing to do is install the plugin in IntelliJ. It can be found easily by searching the plugin repos.

Once the plugin is installed, it needs to be configured for your project. This is achieved by adding a _facet_ to the project. Navigate to the _Project Structure_ view in IntelliJ (`⌘;` on MacOS), select _Facets_ and click the add (`+`) button. Select _Infinitest_ from the list and associate it with the module(s) in your project that you want _Infinitest_ to run against.

![Infinitest facet](/assets/infinitest/infinitest-facet.png){:class="img-responsive"}

A new tool window should now be visible in the bottom panel of your IDE that indicates that _Infinitest_ is watching for changes.

 ![Infinitest window start](/assets/infinitest/infinitest-window-start.png){:class="img-responsive"}

If configuring _Infinitest_ for a brand new project the window will be empty as there won't yet be any tests. When adding _Infinitest_ to an existing project the window may also be empty as _Infinitest_ will only react to changes when the files you are changing are compiled.

> To support the continuous unit testing process, it makes sense to enable the "compile on save" feature in your IDE to ensure that the tests run automatically.

In IntelliJ this means enabling the feature to [_build the project automatically_](https://www.mkyong.com/intellij/intellij-idea-how-to-build-project-automatically/).

Let's write a failing test and take a another look at the window.

{% highlight java %}
@Test
public void thisFails() {
   assertEquals(1, 2);
}
{% endhighlight %}

On saving the test file, the _Infinitest_ window automatically refreshes, we see red and details of the test failure.

![Infinitest window failure](/assets/infinitest/infinitest-window-failure.png){:class="img-responsive"}

Fixing the test and saving causes the window to automatically refresh again and we see green.

![Infinitest window passing](/assets/infinitest/infinitest-window-passing.png){:class="img-responsive"}

That's it, you're good to go with your continuous unit testing setup.

## More on test failures
When a test fails you obviously want to understand why and fix it as soon as possible.

Double clicking the failure in the _Infinitest_ window will take you straight to the line of code that caused the test to fail, usually an assertion but possibly an exception.

In addition to this, _Infinitest_ will highlight the line of code that is failing by adding its icon to the side panel. Hovering over the icon will display the reason the test failed. Nice.

![Infinitest failure icon](/assets/infinitest/infinitest-failure-icon.png){:class="img-responsive"}

## Filtering tests
In many applications, there will be a mix of tests of different styles such as unit, integration and load. In general, it is unlikely you will want to automatically want run integration or load tests every time a file changes that they depend on due to them being slow running or requiring more complex external setup.

_Infinitest_ provides the ability to filter tests for this exact reason. Simply adding an `infinitest.filters` file to the root of your project and adding regular expressions (one per line) will filter out the tests that match.

A simple example to filter out integration tests, assuming they have names such as `MyAppIT`, is to create a file with the following content.

`.*IT`

---

To summarise, if you are following a TDD approach and are looking to shorten the cycles, or even just writing unit tests and want quicker feedback, then following a continuous unit testing approach can help achieve this.

If you're developing in Java and using IntelliJ, or Eclipse, then _Infinitest_ is your friend. It's simple to setup and provides exactly the features you need to start continuously unit testing on your projects.
