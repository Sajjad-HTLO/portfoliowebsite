---
title: "About Junit"
date: 2019-12-31T12:14:34+06:00
description: "About Junit"
author: "Sajad Hayatlou"
type: "post"
---


Thanks to `Junit` 4’s Parameterize testing, parameterized tests make it possible to run a test multiple times with different arguments.The source of parameters can be a method, or simple self declared variables:

First of all, we should declare that our test suit is going to be lunched with ​`JUnitParamsRunner`:


{{< highlight java >}}
@RunWith(JUnitParamsRunner.class)
public class MyTests {
{{< /highlight >}}

This test’s parameters are self declared variables:

{{< highlight java >}}
@Test
@Parameters(value = {"foo","bar"})
void testWithValueSource(String argument) {
    assertNotNull(argument);
}
{{< /highlight >}}

Using a method as the source of parameters:

{{< highlight java >}}
@Test
@Parameters(method = "provideStringParameters")
void testWithValueSource(String argument) {
    assertNotNull(argument);
}

private String[] provideStringParameters(){
    return new String[]{"foo","bar"};
}
{{< /highlight >}}

But, if you want to have a `TestConfiguration` in your test suit, and defining some `Bean` there (for example, a service object bean) Dependency injecttion will be omitted in case of using `JUnitParamsRunner` , and the solution is to add this `Rules` to your test suit:

{{< highlight java >}}
/**
 * These JUnit 4 rules, introduced in Spring 4.2, have the advantage of being independent of
 * any {@link org.junit.runner.Runner} implementation and can therefore be combined with existing
 * alternative runners like the one used in the this test class, {@link JUnitParamsRunner}.
 *
 * @see <a href="http://bit.ly/2tkzggJ">Spring JUnit 4 Rules</a>
 */
@ClassRule
public static final SpringClassRule SPRING_CLASS_RULE = new SpringClassRule();

@Rule
public final SpringMethodRule springMethodRule = new SpringMethodRule();
{{< /highlight >}}

### Summary:

In this short post we see how to use JUnit 4 for parameterized tests.




