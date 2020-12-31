---
title: "Misinterpreting Of Final Arrays"
date: 2020-12-02T12:10:45+24:00
description: "Misinterpreting Of Final Arrays"
author: "Sajad Hayatlou"
type: "post"
---

I read a lot about the advantages of immutable classes and the ways we can achieve immutability, Although immutable classes have their own disadvantages, but the significant weight of advantages, leads most of the developers to use and build such classes when they need.


But there is a horrible pitfall when making an array, a final one:

{{< highlight java >}}

/*
* Create apple objects
*/
Apple apple1 = new Apple(...);
Apple apple2 = new Apple(...);

/*
* Create apples array populating with the apple objects
*/
public static final Apple[] apples = new Apple[]{apple1, apple2};

{{< /highlight >}}

Now, in spite of having the `final` keyword, the array is still mutable!

{{< quotate >}} ... This is a frequent source of security holes (Effective Java, 3rd Ed, Chapter 4) {{< /quotate >}}

In other words, `a non-zero length array is always mutable`, you should not have such a field or an accessor that returns a reference to it.


Because the array is an object in java, we can change the final object's state, but can't change the object reference.
The final keyword here means that you can’t re-assign the array, like this:

{{< highlight java >}}

/*
* Trying to change the array reference (object reference) will fail
*/
apples = new Apple[]{...}; // Compile time error

{{< /highlight >}}

Effectively, the `final` is applied to the `variable`, not the `entries`, So a malicious client can change the contents of the array easily:

{{< highlight java >}}

/*
* Trying to change the array content will result no exception!
*/
apples[0] = new Apple("orange1"); // It is ok!

{{< /highlight >}}

If you imagine the array like a box of `apples`, you can't change the box, but you can change the `apples` inside the box to be `oranges`. The box stays the same, but you’ve effectively changed the contents. 


### So what’s the solution?

### 1: Using an immutable list

You can make the array private and add a public immutable list to deal with this attack:

{{< highlight java >}}
/*
* Wrap the array with an unmodifiable collection
*/
public static final list<Apple> applesList = Collections.unmodifiableList(Arrays.AsList(apples));
{{< /highlight >}}

### 2: Defensive copy

Alternatively, you can use an accessor returning a defensive copy of the original array:

{{< highlight java >}}

/*
* Returning a defensive copy of the array to avoid giving a reference to the original array
*/
public static final Apple[] getApples() {
return apples.clone();
}
{{< /highlight >}}






