---
layout: post
title:  "Discovering Test Doubles"
date:   2019-10-11 00:00:00
categories: testing, mocks, doubles
comments: true
---

I wrote most of the content for this post well over a year ago whilst working for [Auto Trader](https://careers.autotrader.co.uk/). When I joined the company I landed in a team that was pretty unique compared to any I'd worked in before. The focus on quality and technical excellence was very high, there was a real focus on collaboration and I really started to get a feel for what it might be like in an XP team. In the short time I was on the team I learned a lot from some very smart people.

It's a bit of a long read but hopefully you find it useful and interesting. If you just care about what a test double is then go straight to [that section](#vocabulary) and skip the background info.

I'd love to hear any thoughts or experiences people have on this topic. Also any feedback on the content would be awesome. 

I've got doubts about the code examples used and really struggled with the section on mocks but after lots of umming and ahing I've just decided to poublish it as is. The way I see it is that it's just a waste of my time if I keep it hidden away on personal computer, what's the worst that could happen :)

* Will be replaced with the ToC
{:toc}

# Some background

I started working with Java and object-oriented (OO) languages full-time just over 5 years ago. I made a concious decision to explore as many "best-practice" approaches to designing and implementing OO software as I could. This involved lots of research around the various techniques involved in creating well architected, _clean_ software.

What I understand by well architected, _clean_ software is:

> Software that is easy to understand (i.e. the design reveals the intent) and maintain (i.e. change).
>
> Robert C. Martin

This, however, is much easier said than done. After years of continuous trial and error there is still so much to learn. The number of times I look back at code I wrote 6 months ago, even 1 month ago (in fact, sometimes a few hours ago), and question my design choices is high. I think this is healthy though. We should always be challenging ourselves to become better developers.

When I was starting out developing OO software the idea of automated testing was quite alien to me. Previous to this I worked in an environment where testing was treated as a separate phase of the development process performed by a testing team. The only testing I carried out as a dev was manual or, at best, through writing throwaway test harnesses to exercise the code I had written. Looking back this feels so wrong but at the time it was my norm, and hindsight is a wonderful thing.  

> Testing is all about getting feedback on software.
>
> xUnit Test Patterns - Gerard Mezaros

Around the same time I started to develop an appreciation for delivering software in a more _agile_ way with the ideas behind [eXtreme Programming](https://en.wikipedia.org/wiki/Extreme_programming) resonating strongly with me. The desire for ever shorter feedback loops in the development cycle and emphasis on Test Driven Development (TDD) helped me realise that without automated tests it would be very difficult, if not impossible, to incrementally evolve a software system with any level of confidence. Following a good few years of writing automated tests and practicing continuous integration it just seems like a no-brainer.

> When a class does not depend on any other classes, testing it is relatively straightforward.
>
> xUnit Test Patterns - Gerard Mezaros

Another reality that dawned on me more recently was that the design of these tests is as important to the overall understanding and maintainability of the software as the production code. There is definitely a skill to writing good tests and, while writing tests for a simple class or function with no dependencies is relatively straightforward, as soon as we tackle a more complex body of code, with one or more dependencies, things become a bit trickier.

To tackle this complexity it is common practice to reach for a mocking framework such as [Mockito](http://site.mockito.org/). Mocking frameworks enable developers to "mock" dependencies in their tests with ease but this can come at a cost, and it is a cost that I have felt first hand. 

Mocking frameworks make it very convenient to "mock" dependencies in our tests and because of this I found myself mocking them without really thinking about the reasons why. The cost that materialised in my tests was that their intent became less clear making them difficult to understand what they are actually trying to achieve. I noticed that many tests would become quite complicated and the scope of the unit being tested became less clear.

This point is also pertintent where there is a desire for tests to provide a form of documentation for a system. If the tests aren't clear then their value significantly decreases.

Most of the time when using a mocking framework we aren't actually creating mocks. The term "mock" has become slang for test double so it's very easy to gloss over the fact that we are making use of various different types of test double.

Interestingly, the term "mock" is commonly used as a verb. We'll say things like "we need to mock this out" or "we're going to need to do some mocking in this test".

> When all you have is a mocking framework, everything looks like a mock!
>
> Me

Prior to joining Auto Trader I used Mockito extensively in my tests. It was one of first tools I picked up when I started to learn TDD and write automated tests in Java. Then, all of a sudden, I was dropped into a code base where a concious decision had been made not use a mocking framework, instead all test doubles were hand rolled. Instead of seeing "mock" everywhere, I was now seeing terms such as dummy, stub, fake, as well as mock. Now don't get me wrong, I knew of these terms but being able to create my own dummies and stubs wasn't something that came naturally. Through using Mockito I'd totally glossed over the details. In many ways having access to tools & frameworks such as Mockito is great as I felt like I became productive with automated testing fairly quickly. With hindsight it also had its problems, for example, without a second thought I would "mock" a collaborator required by my test.

> Easiness will eventually slow you down
>
> Simple Made Easy - Rich Hickey

So while mocking frameworks do make it easier to test our software I believe we also need to consider the incidental complexity that comes with them. If we don't take care to understand the tools we are using and the implications of the choices we make, the cost could be that we make choices that seem easier at the expense of longer term complexity.

Now we have some context let's get to the main focus of this post which is to explore the techniques available to us for writing simple, maintainable, and therefore _clean_, tests. We will focus on the trickier side of writing automated tests where we need to consider dependencies. In particular we will focus on the use of _test doubles_ in place of those dependencies to verify the behaviour of a system. Through this we will understand the fundamental building blocks that we can use (and mocking frameworks also use) to test code that depends on other code. 

With this knowledge we will have a better understanding of when to use a _test double_ and importantly which type of _test double_ we need, whether that is via a mocking framework or rolling our own. 

# Vocabulary

First, let's define some of the concepts we'll be exploring and the vocabulary we'll be using. The following definitions have been collected from various leaders in this space over the past 20 years.

##  System Under Test

The system under test (SUT), or test subject, is...

> short for "whatever thing we are testing" and is always defined from the perspective of the test.

> [Gerard Mezaros](http://xunitpatterns.com/SUT.html)

## Unit test

A low-level fast running test, usually written by a developer, focusing on a small part of the software system, that verifies some expectations about it.

But, what is a unit? 

There are differing opinions of what a unit is but the one I adhere to and we'll use for the purposes of this talk is nicely described by Martin Fowler...

> Although I start with the notion of the unit being a class, I often take a bunch of closely related classes and treat them as a single unit. 
>
> [Unit Test - Martin Fowler]( https://martinfowler.com/bliki/UnitTest.html)

## Collaborator

Something that the test subject depends on, also known as a dependency. I will use these terms interchangeably.

> An individual class or a large-grained component on which the system under test (SUT) depends.
>
> [xUnit Test Patterns - Gerard Mezaros](http://xunitpatterns.com/DOC.html)

## Test double

The term _test double_ is a play on the term _stunt double_ from the film industry. The idea being that we have body doubles that will stand in for the real actors. They will look the same and act similarly but they are not the real thing.

> Test Double is a generic term for any case where you replace a production object for testing purposes. 
>
> Martin Fowler - Mocks Aren't Stubs

There are various types of test double and we will cover in these in more detail next.

# Test Doubles

Now we have a bit of background and some high-level testing vocabulary to work with, using an example we'll explore the different types of test double. 

When reading up about this topic, I encountered a various descriptions of the different test doubles with subtle differences. As with a lot of things in tech, opinions differ. The descriptions I provide here aren't my own, they come from leaders in the field such as [Gerard Mezaros](http://xunitpatterns.com/), [Robert Martin](https://8thlight.com/blog/uncle-bob/2014/05/14/TheLittleMocker.html) and [Martin Fowler](https://martinfowler.com/articles/mocksArentStubs.html)

The are five main types of test double:
 - Dummy 
 - Stub 
 - Spy 
 - Mock
 - Fake 

The following examples test the algorithm for a notification service. Here's the API:

```java
class NotificationService {
   private final User user;

   NotificationService(User user) {
      this.user = user;
   }

   boolean process(Notification notification) {
      //...
   }
}
```

The job of the `NotificationService` is to determine whether the notification can be published for the user. A boolean is returned that represents whether a notification has been published or not.

While unit testing this we don't want to send real notifications. The delivery of real notifications will be performed by an external 3rd party system (e.g. AWS SNS or GCM) and for unit testing purposes we don't want to concern ourselves with this dependency. Amongst other things, using a real notification service introduces latency, non-determinism and of course the cost of using the service. If we are running the unit tests mutliple times a day, we want them to be quick, repeatable and we definitely don't expect them to have a direct financial impact to the business.

Therefore, instead of `process` depending on a real implementation of a notification, it depends on the concept of a notification. This is represented by the `Notification` interface.

```java
interface Notification {
   Result publish();
}
```

`publish` returns a `Result`, which is an enum representing successful publish, or otherwise, of the notification.

```java
enum Result {
    SUCCESS, FAIL
}
```

With this in mind, let's now take a look at our first test double.

## Dummy

The simplest of all test doubles is the dummy (or dummy object). 

> Dummy objects are passed around but never actually used. Usually they are just used to fill parameter lists.
>
> [Mocks aren't Stubs - Martin Fowler](https://martinfowler.com/articles/mocksArentStubs.html)

Importantly, if a dummy was used in a test we would expect it to fail as this would be unexpected behaviour and we should be thinking about using another type of test double.

In respect of our notification service example, to publish a notification, the user must be authorised. If they are not, we need to ensure that a notification has not been published. The last thing we want is any old user sending out notifications and running up a bill with our favourite cloud provider.

In the context of our test, one way to ensure that a notification isn't sent when a request is made by an unauthorised user is to use a dummy notification. We need to supply a notification because the API requires it however we don't expect it to be used.

One option here is to simply pass null instead of an object to play the role of the dummy notification.

```java
@Test
void notification_not_processed_when_user_is_unauthorised() {
    final NotificationService testSubject
            = new NotificationService(unauthorisedUser());

    assertFalse(testSubject.process(null));
}
```

This will work as if there any interactions with the dummy (or in this case `null`) notification then the test should blow up due to a `NullPointerException`. This isn't very obvious though and if a test fails because of it we'll need to perform a few mental hops to work out what's gone wrong. Also, I like code to be simple and express its intent. 

I want the tests to speak to me so let's create a dummy object to serve that purpose.

```java
class DummyNotification implements Notification {
   @Override
   Result publish() {
      throw new RuntimeException("This should not be called!");
   }
}
```

Using an IDE (IntelliJ for me) we can quite quickly create a simple dummy implementation of the `Notification` interface. The `DummyNotification` will throw an exception if an attempt is made to publish it. This allows us to write a test that will fail if there are any unintended interactions with the notification whilst processing it for an unauthorised user.

```java
@Test
void notification_not_processed_when_user_is_unauthorised() {
    final NotificationService testSubject
            = new NotificationService(unauthorisedUser());

    assertFalse(testSubject.process(new DummyNotification()));
}
```

An alternative, if we're using Java 8 or later, is to simply pass a lambda as the `Notification` interface meets the contract for the [`Supplier`](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html) functional interface.

```java
@Test
void notification_not_processed_when_user_is_unauthorised() {
    final NotificationService testSubject
            = new NotificationService(unauthorisedUser());

    Notification dummyNotification = () -> {
        throw new RuntimeException("This should not be called");
    };

    assertFalse(testSubject.process(dummyNotification));
}
```

This is nice as the expectations are clear in the test and we're able to do away with the extra class we created for the dummy notification, although we'll need to repeat this pattern wherever we want to use an equivalent double. It will depend on the context but it's a nice option if you want to use it. If you prefer, you could inline the lamba however I think giving it a name improves the readability of the test.

So that's a dummy, nice and simple but not very clever. Preferably we wouldn't want to put too much effort into creating a dummy for our tests. Also, if we are using lots of dummies there may be a problem with our design, for example the system under test having too many responsibilities.

While dummies are relatively quick and easy, their use is limited. A dummy won't help us to test the other scenarios that the `NotificationService` must support. One way we can do this is through the use of another test double, the Stub.

## Stub

So what is a Stub? Martin Fowler's description covers it nicely.

> Stubs provide canned answers to calls made during the test, usually not responding at all to anything outside what's programmed in for the test.
>
> [Martin Fowler](https://martinfowler.com/bliki/TestDouble.html)

You may have noticed the use of `unauthorisedUser()` and `authorisedUser()` in the tests so far and wondered where they had come from. They are in fact stubs that we've already been using on the quiet. These stubs are hard wired with the result allowing us to create types named very specifically to represent each of an authorised and unauthorised user.

```java
public class UnauthorisedUserStub implements User {
    static User unauthorisedUser() {
        return new UnauthorisedUserStub();
    }

    @Override
    public boolean authorise() {
        return false;
    }
}

public class AuthorisedUserStub implements User {
    static User authorisedUser() {
        return new AuthorisedUserStub();
    }

    @Override
    public boolean authorise() {
        return true;
    }
}
``` 

The nice thing about these stubs is that they can easily be shared across many tests, and when you see them being used their purpose is very clear.

Alternatively, switching back to the `Notification`, we may want to create a stub that's a bit more flexible.

```java
public class NotificationStub implements Notification {
   private final Result result;

   NotificationStub(Result result) {
      this.result = result;
   }

   @Override
   public Result publish() {
      return result;
   }
}
```

When we create this stub, we proivde the `Result` that we want it to return. This is known as a programmable stub.

```java
@Test
void process_succeeds_when_user_is_authorised_and_notification_is_successful() {
   final NotificationStub successfulNotificationStub = new NotificationStub(Result.SUCCESS);

   final NotificationService testSubject = new NotificationService(authorisedUser());

   assertTrue(testSubject.process(successfulNotificationStub));
}

@Test
void process_fails_when_user_is_authorised_but_notification_fails() {
   final NotificationStub failedNotificationStub = new NotificationStub(Result.FAIL);

   final NotificationService testSubject = new NotificationService(authorisedUser());

   assertFalse(testSubject.process(failedNotificationStub));
}
```

We now know how to provide a stub to stand in for a collaborator and ask it to supply canned answers when it's interacted with but this doesn't guarantee that a call was made to the collaborator. Maybe the implementation just happened to pass the test. If we want to be sure that our test subject did infact use the collaborator in its algorithm then we need the help of another test double, the spy.

## Spy

The _spy_ test double gives us a way to implement behaviour verification, that is, a way of verifying the iteractions of the test subject with a collaborator.

In the reliable words of Martin Fowler...

> Spies are stubs that also record some information based on how they were called.
>
> [Martin Fowler](https://martinfowler.com/bliki/TestDouble.html)

A spy provides a way to capture information about how the test subject interacted with it, for example, the number of invocations or even the actual arguments used in a method call.

To create a spy we need to roll a slightly different implementation of the `Notification` interface.

```java
class NotificationSpy implements Notification {
    private final Result result;
    private boolean publishWasCalled = false;

    NotificationSpy(Result result) {
        this.result = result;
    }

    @Override
    public Result publish() {
        publishWasCalled = true;
        return result;
    }

    boolean publishWasCalled() {
        return publishWasCalled;
    }
}
```

Now, as well as returning a stubbed `Result`, when `publish` is called we record the fact in a boolean flag. This allows us to perform a verification against the flag in our test.

```java
@Test
void process_succeeds_when_user_is_authorised_and_notification_is_successful() {
   NotificationSpy successfulNotificationSpy = new NotificationSpy(Result.SUCCESS);

   final NotificationService testSubject = new NotificationService(authorisedUser());

   assertTrue(testSubject.process(successfulNotificationSpy));
   assertTrue(successfulNotificationSpy.sendWasCalled());
}
```

This is nice, the test is now speaking to us and the intent is obvious.

There is a flaw though in this implementation as if `publish` was called multiple times we wouldn't know. As there is a cost associated with publishing real notifications it's definitely a behaviour with undesirable side-effects.

A minor change to the our spy allows us to verify that `publish` is called once and only once.

```java
class NotificationSpy implements Notification {
    private final Result result;
    private int publishWasCalledCount = 0;

    NotificationSpy(Result result) {
        this.result = result;
    }

    @Override
    public Result publish() {
        publishWasCalledCount++;
        return result;
    }

    int publishWasCalledCount() {
        return publishWasCalledCount;
    }
}
```

Our test can then verify the number of calls made to `publish` that have been recorded against the spy.

```java
@Test
void process_succeeds_when_user_is_authorised_and_notification_is_successful() {
   NotificationSpy successfulNotificationSpy = new NotificationSpy(Result.SUCCESS);

   final NotificationService testSubject = new NotificationService(authorisedUser());

   assertTrue(testSubject.process(successfulNotificationSpy));
   assertEquals(notificationSpy.sendWasCalledCount(), 1);
}
```

With the spy test double covered we'll move swiftly on to the next, the mock.

## Mock

Until recently I would have referred to all of the above as a mock! This was ultimately the motivation for this post. I wanted to discover the real meaning behind the term mock, and more generally test doubles.

Mocks are...
 
> ...objects pre-programmed with expectations which form a specification of the calls they are expected to receive.
>
> [Martin Fowler](https://martinfowler.com/articles/mocksArentStubs.html)

In essence, this means that the mock has the knowledge of how the test subject should interact with it and is able to verify it.

When it comes to creating our own mock object things become a bit trickier. We need to write more code in our test double meaning its complexity is going to rise and there will be a greater maintenance overhead. With this in mind, we'll have a go anyway.


```java
class NotificationMock implements Notification {
    private final Result result;
    private int actualPublishCallCount = 0;

    NotificationMock(Result result, int expectedPublishCallCount) {
        this.result = result;
        this.expectedPublishCallCount = expectedPublishCallCount;
    }

    @Override
    public Result publish() {
        actualPublishCallCount++;
        return result;
    }

    void verify() {
        assertTrue(actualPublishCallCount, expectedPublishCallCount);
    }
}
```

We can then construct our mock and pre-program it with the expectation of a single call being made to the `publish` method.

```java
@Test
void process_succeeds_when_user_is_authorised_and_notification_is_successful() {
   NotificationMock successfulNotificationMock = new NotificationMock(Result.SUCCESS, 1);

   final NotificationService testSubject = new NotificationService(authorisedUser());

   assertTrue(testSubject.process(successfulNotificationMock));
   successfulNotificationMock.verify();
}
```

In some ways though the test has become less obvious. One way to improve this is to provide a builder for the mock so we can create it using a more fluent API e.g. 

```java
new NotificationMockBuilder()
    .withResult(Result.SUCCESS)
    .withExpectedPublishCallCount(1);
```

At this point though it probably makes sense to reach for a mocking framework such as [Mockito](https://site.mockito.org/). The power of Mockito (and other mocking frameworks) removes a lot of the complexity and code we have to write and maintain ourselves. 

Of course with great power comes great responsibility but, having read this far, we'll hopefully be more informed when reaching for the more powerful tools that are available.

## Fake

When we want to verify the behaviour of the test subject but depending on the real implementation of a collaborator is not possible or desirable, one option is to use a _fake_ object.

> Fake objects actually have working implementations, but usually take some shortcut which makes them not suitable for production
>
> [Martin Fowler](https://martinfowler.com/articles/mocksArentStubs.html)

A good example is where you choose to use an in-memory database instead of the real thing. This helps to remove the dependency on the real database being available as well being much more efficient. If the database is a key/value store then an even simpler Map based implementation could fit the bill and provide even greater efficiency. After all, unit tests should be fast.

To demonstrate with a simple, possibly unlikely, example we'll create a fake `User`. The fake `User` will have a working implementation of the `authorise` method that will only allow notifications to be sent for users with a name starting with _"Nick"_.

```java
public class FakeUser implements User {
    
    private final String name;

    public FakeUser(String name) {
        this.name = name;
    }

    @Override
    public boolean authorise() {
        return name.startsWith("Nick");
    }
}
```

This can then be used in our example tests as an alernative to the authorised user stub.

```java
@Test
void process_succeeds_when_user_is_authorised_and_notification_is_successful() {
   User authorisedUser = new FakeUser("Nick Ebbitt");
   NotificationStub successfulNotificationStub = new NotificationStub(Result.SUCCESS);

   NotificationService testSubject = new NotificationService(authorisedUser);

   assertTrue(testSubject.process(successfulNotificationStub));
}
```

And with that we have covered the test doubles I set out to in this post. The Dummy, Stub, Spy, Mock and finally the Fake.

# Wrapping Up
First of all, if you've made it this far then thanks for reading, I hope you found it worthwhile :)

> Know your test doubles, not everything is a "mock".

Hopefully, if not already, you now have a better understanding of the different types of test doubles that can be used to help us test your code. 

> Try to write clean tests that speak to you.

With this shared understanding and vocabulary, as developers we can have better conversations around the design of our tests.

> When using a mocking framework, think about the test doubles in play.

This should mean we are better placed to choose the right test double for our use-case.

> Where it makes sense, use the power of a mocking framework to help write your tests, but do so with a good understanding of how they work.

We'll also be more aware of the test-cases that genuinely benefit from using a mocking framework, and when you do you'll understand the work they are doing for you under the hood.

> Agree on practices & principles with your team, be consistent.

Regardless of whether you roll your own doubles, use a mocking framework or a apply a mixture of both I believe its really valuable to discuss and agree the approach and style as a team. The value of a codebase with a consistent and coherent style shouldn't be underestimated.



