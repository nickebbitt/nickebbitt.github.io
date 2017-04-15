---
layout: post
title:  "Looking forward to JUnit 5"
date:   2017-04-16 00:00:00
categories: java, testing
comments: true
---
In this post I explore some of the features I'm looking forward to in the upcoming JUnit 5 release.

First, a bit of background...

The process to create a new version of JUnit kicked off around the middle of 2015 when a [crowdfunding campaign](https://github.com/junit-team/junit5/wiki/Crowdfunding-Campaign) was started. At this point the initiative was referred to as **JUnit Lambda**. One of the drivers for this was the release of Java 8 that provided a number of language features, such as Streams and Lambdas, offering new ways for the test framework to be implemented and tests to be written. The project was later renamed JUnit 5 and the [roadmap](https://github.com/junit-team/junit5/wiki/Roadmap) for the project leading up to the present day can be found on their GitHub page.

JUnit 5 is not yet complete however the GA release is currently on schedule to be available during Q3 2017. The latest milestone, [5.0.0 M4](http://junit.org/junit5/docs/current/user-guide/#release-notes-5.0.0-m4), was released on 1st April 2017.

Anyway, here are my thoughts on some of the new features that will be available later this year...

* Will be replaced with the ToC
{:toc}

## Display Names

How often do you write a test case for which the name is quite long due to you wanting to clearly describe the intentions of the test. Depending on your coding style, you may use camel case that can `resultInReallyLongAndDifficultToReadTestNames`.

The new [`@DisplayName`](http://junit.org/junit5/docs/current/user-guide/#writing-tests-display-names) annotation provides a solution to this. While functionally this change isn't really offering anything too exciting, I think the clarity it adds to your tests, particularly when viewing them in the test results panel of your IDE, is really valuable.

{% highlight java %}
@DisplayName("The display name annotation provides a...")
public class DisplayNames {

    @Test
    @DisplayName("way of producing more readable test names")
    void test1() {

    }

    @Test
    void ratherThanSomethingHarderToReadLikeThis() {

    }
}
{% endhighlight %}

The improvement of the new style test names over the old is evident when viewing test results in the IDE.

![Display Names IDE Integration](/assets/junit5/display-names-ide.png){:class="img-responsive"}

## Nested Tests

The [`@Nested`](http://junit.org/junit5/docs/current/user-guide/#writing-tests-nested) annotation provides us with a new way of organising tests. When applying the annotation to a non-static inner class we are now able to create one or more logical groups of test cases within a single outer test class. The nesting can be applied to an arbitrary depth providing great flexibility.

{% highlight java %}
@DisplayName("High-level User Story")
public class NestedTests {


    @Nested
    @DisplayName("Simple Scenario")
    class FeatureOne {

        @Test
        @DisplayName("Test One")
        void testOne() {

        }

        @Test
        @DisplayName("Test Two")
        void testTwo() {

        }

    }

    @Nested
    @DisplayName("More Complex Scenario")
    class FeatureTwo {

        @Test
        @DisplayName("Test One")
        void testOne() {

        }

        @Nested
        @DisplayName("Nested Scenario")
        class SubFeature {

            @Test
            @DisplayName("Test One")
            void testOne() {

            }

        }

    }

}
{% endhighlight %}

This works great with IDE integration too to further improve test organisation and the readability of the results...

![Nested Tests IDE Integration](/assets/junit5/nested-tests-ide.png){:class="img-responsive"}

## Tagging & Filtering

The `@Tag` annotation provides us with a nice way of categorising tests. An obvious use-case would be to categorise tests based on the style of test e.g. integration, system, end-to-end, etc.

{% highlight java %}
public class TaggedTests {

    @Test
    @Tag("integration")
    void integrationTest() {

    }

    @Test
    @Tag("system")
    void systemTest() {

    }

    @Test
    @Tag("end-to-end")
    void endToEndTest() {

    }
}
{% endhighlight %}

Then, for example, the different types of tests could be run as required at the relevant stages of a delivery pipeline.

## Grouped assertions

The ability to now group assertions looks a really useful new feature. In JUnit 4, if a test had multiple assertions then execution would halt after the first failure.

{% highlight java %}
@Test
void ungroupedAssertions() {

    assertTrue(false, "this test fails here");
    assertTrue(true, "so we never perform this assertion");

}
{% endhighlight %}

Whereas in JUnit 5, every assertion will be performed and the results provided for each of them.

{% highlight java %}
@Test
void groupedAssertions() {

    assertAll(
       "all of these assertions",
            () -> assertTrue(true, "are performed"),
            () -> assertTrue(false, "even if the"),
            () -> assertTrue(false, "previous one failed")
    );
}
{% endhighlight %}

When viewing the test failure for a grouped assertion in the IDE (IntelliJ) we see that the test failed and closer inspection of the output / stack trace shows the detail related to which assertions failed.

![Grouped assertions IDE](/assets/junit5/grouped-assertions-ide.png){:class="img-responsive"}

It would be nice to see the IDE integration enhanced further to present the individual assertion failures more clearly in a similar way to the overall test case failures. I had to scroll a bit to find the summary of the assertions that failed.

## Parameterised Tests
Now parameterised tests aren't a new thing - we could write them in JUnit 4 with the help of the [`Parameterized.class`](https://github.com/junit-team/junit4/wiki/parameterized-tests) test runner. Providing the parameters was ugly though...

{% highlight java %}
@Parameters
public static Collection<Object[]> data() {
    return Arrays.asList(new Object[][] {     
             { 0, 0 }, { 1, 1 }, { 2, 1 }, { 3, 2 }, { 4, 3 }, { 5, 5 }  
       });
}
{% endhighlight %}

This feature has been revisited and a completely new way of writing parameterised tests has been provided. With the use of the `@ParameterizedTest` and `@ValueSource` annotations we have a simple but very powerful way of providing parameters to a test method.

Values can be provided in-line similarly to the example above but with much simplified syntax.

{% highlight java %}
@ParameterizedTest
@ValueSource(ints = {1, 2, 3})
void testWithParams(int param) {
    assertTrue(param < 3);
}
{% endhighlight %}

As can be seen in the following screen shot, three tests are executed of which one fails.

![Parameterised Tests IDE](/assets/junit5/parameterised-tests-ide.png){:class="img-responsive"}

As well as `@ValueSource`, other options for providing test inputs as parameters are:

 * `@EnumSource` - a set of Enum constants.
 * `@MethodSource` - the result of a method in the form of a `Stream`, `Iterable`, `Iterator` or an array of arguments.
 * `@CsvSource` - a list of comma separated values.
 * `@CsvFileSource` - a CSV file from the classpath.

 The `@CsvFileSource` stands out as a really convenient way of driving large volumes of test data through a test case.

## Final thoughts

This post only scratches the surface of what's coming in JUnit 5.

One key area I've not covered at all that looks intriguing is the ability to write [dynamic tests](http://junit.org/junit5/docs/current/user-guide/#writing-tests-dynamic-tests). I'm looking forward to seeing how this is used in practice.

The new [extension model](http://junit.org/junit5/docs/current/user-guide/#extensions) also looks like it will provide a simple and flexible new way for JUnit to be extended, improving on the previous options of `Runner`, `@Rule` and `@ClassRule`.

I'm looking forward to the making use of all of the great new features when JUnit 5 is released.

---

All code from this post is available on [GitHub](https://github.com/nickebbitt/junit5-examples).
