---
title: "Defensive Copying"
date: 2021-01-21T12:10:45+24:00
description: "A  use case of defensive copying for preserving immutability"
author: "Sajad Hayatlou"
type: "post"
---

Defensive Copying is a term used for making objects `immutable`, so you cannot change the state of an immutable object once created.

Preserving immutability is critical sometimes and there are two kind of approaches to achive, using immutable objects, or employing `defensive copying` carefully.

Let start the story with a simple example, suppose we're creating a period class called `Period`.

{{< highlight java >}}

public final class Period {
	private final Date start;
	private final Date end;

	/**
	* @param start the beginning of the period
	* @param end the end of the period; must not precede start
	* @throws IllegalArgumentException if start is after end
	* @throws NullPointerException if start or end is null
	*/
	public Period(Date start, Date end) {
		if (start.compareTo(end) > 0)
		  throw new IllegalArgumentException(start + " after " + end);
		this.start = start;
		this.end = end;
	}

	public Date start() {
		return start;
	}

	public Date end() {
		return end;
	}
		...
		// Remainder omitted
}

{{< /highlight >}}



At the first glance, this might look immutable and to enforce the invariant that the start of a period does not follow its end, but it's however easy to violate this invariant by exploiting the fact that `Date` is mutable:

{{< highlight java >}}

// Attack the internals of a Period instance
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);

end.setYear(80); // Modifies internals of p!


{{< /highlight >}}

If you take another look at `Period` class, you can see that we're openning a door for the client code to establish a reference to our objects' attributes (`start` and `end` in this example) and because the parameters' type is `Date` and this class is intrinsically `mutable`, now it seems obvious to change the internals of the created object upon chaning the state of one of the parameters from outside.

So now there are two kind of fixes, using equivalent immutable objects, like `Instant` or `ZonedDateTime` or employ the `Defensive Copying` when assigning the `Period`'s attributes, we'll consider the latter one in this article.

### Use defensive approach
To protect the internals of `p` from this sort of attack, it is essential to make a defensive copy of each mutable parameter to the constructor.

{{< highlight java >}}
// Repaired constructor - makes defensive copies of parameters
public Period(Date start, Date end) {
	this.start = new Date(start.getTime());
	this.end = new Date(end.getTime());

	if (this.start.compareTo(this.end) > 0) 
	   throw new IllegalArgumentException(this.start + " after " + this.end);
}
{{< /highlight >}}


Note that defensive copies are made before checking the validity of the parameters and the validity check is performed on the copies rather than on the originals.
While this may seem unnatural, it is necessary. It protects the class against changes to the parameters from another thread during the window of vulnerability between the time the parameters are checked and the time they are copied. In the computer security community, this is known as a `time-of-check/time-of-use` or `TOCTOU` attack.

### Avoid using clone method for non-final objects

Note that we have not use the `clone` method for copying, because the `Date` is non-final, it's not guaranteed to return an objetc whose class is `java.util.Date` , it could be return an instance of an untrusted subclass. So as rule of thumb, when copying defensively, avoid using `clone` method for the mutable parameters, which their type is subclaasable by an untrusted parties.

So, although the repaired constructor will protect the created `Period` object from being changed by outside, but the `Period` class is still vulnerable to a similiar attack:

{{< highlight java >}}
// Second attack on the internals of a Period instance
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // Modifies internals of p!
{{< /highlight >}}

To defend against the second attack, merely modify the accessors to return defensive copies of mutable internal fields:

{{< highlight java >}}
// Repaired accessors - make defensive copies of internal fields
public Date start() {
	return new Date(start.getTime());
}
public Date end() {
	return new Date(end.getTime());
}
{{< /highlight >}}

Now, with the new constructor and the new accessrors in place, `Period` is truly immutable. No matter how malicious a programmer, there is defenitely no way to voilate the invariants of the `Period` (of course, without resorting to extralinguistic means, such as native methods and reflection).

In our example, the `Date` is obsolete and should no longer be used in new code, use its eqivalents instead.


### Conclusion

In this article, i tried to demonstrate the importance of preserving immutability when it's a critical requirement in program and how to achive it by taking `Defensive Copying` in action. When designing an immutable class, think twice about choosing the attributes type and also avoid exposing a reference to the mutable internals of the class from the outside, either in the constructor or in accessors.

