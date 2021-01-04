---
title: "The Pattern matching for instanceof"
date: 2021-01-03T17:20:34+06:00
description: "Finalizing Pattern matching for the instanceof operator in JDK 16 "
author: "Sajad Hayatlou"
type: "post"
---


### Introduction

As a Java developer, you may likely saw using of `instanceof` operator to check the type of a variable.

Here we simply check the given `shape` object is an instance of `Rectangle`:

{{< highlight java >}}

 public static double getPerimeter(Shape shape) throws IllegalArgumentException {
    if (shape instanceof Rectangle) {
        // So we're sure that shape is an instance of Rectangle, cast it and use
        Rectangle s = (Rectangle) shape;
        return return 2 * s.length() + 2 * s.width();	
       }
    }
{{< /highlight >}}

But as you may figure out, there are some tedious in this process, the type name repeated three times, we need to explicitly cast the variable after the instanceof returned true and repeatedly declaring the type name means there's more likelihood of introducing an error.

The issue gets worse each time we add a new shape!

{{< highlight java >}}
 public static double getPerimeter(Shape shape) throws IllegalArgumentException {
        if (shape instanceof Rectangle) {
	    // So we're sure that shape is an instance of Rectangle, cast it then use
         Rectangle s = (Rectangle) shape;
         return return 2 * s.length() + 2 * s.width();	
        } else if (shape instanceof Circle) {
          Circle s = (Circle) shape;
          return 2 * s.radius() * Math.PI;
        } else if (shape instace of Triangle) {
          Triangle s = (Triangle) shape;
          return s.getA() + s.getB() + s.getC();
        }

	///  More conditional statements for different shapes

    }
{{< /highlight >}}

### Enhanced version of instanceof

More specifically, JDK 14 extends the instanceof operator and a preview feature and in it's finalized in JDK 16, introduces an improved version of `instanceof` operator that `both tests the parameter and assigns it to a binding variable of the proper type`. This means the resulting code could be less tedious, shorter and understandable.

{{< highlight java >}}
 if (shape instanceof Rectangle s) {
      // The s is called a binding variable
     return 2 * s.length() + 2 * s.width();
 }
{{< /highlight >}}

The variable `s` is the new thing we see at the first sight and it's actually the casted variable after the `instanceof` operator returned `true`, there is no need to explicitly cast the type.

So our `getPerimeter(Shape shape)` could be refactored as this:

{{< highlight java >}}
 public static double getPerimeter(Shape shape) throws IllegalArgumentException {
     if (shape instanceof Rectangle s) {
         return 2 * s.length() + 2 * s.width();
     } else if (shape instanceof Circle s) {
         return 2 * s.radius() * Math.PI;
     } else if (shape instanceof Triangle s) {
         return s.getA() + s.getB() + s.getC();
     } 
     else {
         throw new IllegalArgumentException("Unrecognized shape");
     }
 }
{{< /highlight >}}

### The binding variable scope

The scope of a binding variable are the places where the program can reach only if the instanceof operator is true:

{{< highlight java >}}
 public static double getPerimeter(Shape shape) throws IllegalArgumentException {
     if (shape instanceof Rectangle s) {
         // You can use the binding variable s of type Rectangle here.
     } else if (shape instanceof Circle s) {
         // You can use the binding variable s of type Circle here, you can't use the s of type Rectangle!
     } else {
         // You can't use any of binding variables here!
     }
 }
{{< /highlight >}}


You can extend the scope of the binding varible:

{{< highlight java >}}
if (!(shape instanceof Rectangle s)) {
    // You cannot use the s here
    return false;
  }
   // You can use s here.
   assertNotNull(s);

{{< /highlight >}}

Use a binding variable in the expression:

{{< highlight java >}}
if (shape instanceof Rectangle s && s.length() > 5) {
    // ...
}
{{< /highlight >}}

### IntelliJ IDEA suggestion for using pattern matching

In `IntelliJ IDEA 2020.1`, you can invoke context sensitive actions on the variable s (by using Alt+Enter or by clicking the yellow light bulb) and select ‘Replace ‘s’ with pattern variable’, to use Pattern Matching:

{{< figure src="https://jaxenter.com/wp-content/uploads/2020/03/java-14-Pattern-matching-for-instanceof-1-e1584440760553.png" >}}

 The preceding action would modify the code as follows:

{{< figure src="https://jaxenter.com/wp-content/uploads/2020/03/java-14-Pattern-matching-for-instanceof-2-e1584440778198.png" >}}


