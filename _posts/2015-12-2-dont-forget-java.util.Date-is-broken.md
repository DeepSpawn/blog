---
layout: post
title: Dont forget java.sql.Timestamp is broken
description: One of the bugs you can encounter with java.util.Date
tags: [Java, Object-Orientied-Programming]
---

I hit a pretty interesting bug today that I thought I would share that highlights one of the reasons why you should avoid java.util.Date

Imagine you have the following class

<script src="https://gist.github.com/DeepSpawn/4b3cc7bcadf0fdf7806c.js"></script>

Now consider the following snippet

<script src="https://gist.github.com/DeepSpawn/baeab47da5a382867372.js"></script>

A naieve reader of that code would assume that firstEq should be equal to secondEq, as equality is an (equivalence relation)[https://en.wikipedia.org/wiki/Equivalence_relation].
and hence symetric. Unfortunately for us we live in the real world, where .equals() should be an equivalence relation but sometimes this is not the case:
**firstEq != secondEq** .

Notice that I am passing in a Timestamp rather than a Date when constructing the FooWithDate. Seems fine as Timestamp is a subclass of Date.
However means that if Timestamp were to override the equals method we will be calling Timestamp#equals in the the first case and Date#equals in the second.
Unfortunately for us someone did exactly that and Timestamp#equals will return false is you pass in something that is not a Timestamp, like its parent class Date.

In the second case we call equals on Date, passing in the Timestamp as the argument. This method behaves as expected so returns true.
And here we have the non symetrical equality that can puzzle the unfamiliar. 

So why does Timestamp override .equals in a symmetry breaking way? Well Timestamp is a thing wrapper around Date that also stores nanoseconds.
How are we supposed to handle the extra information with regards to equality? If we just take the equals method frm the parent class then it will ignore
nanoseconds but if we override it we break equals as shown above.

This is a good example of how you cannot extend an instantiable class and add something while preserving equals. This is a fundamental limitation of
equivalence relations in object orientied programming. After working out what was going on and noticing that there was a problem with subclasses overriding
equals I went and consultd Effective Java as I remebered it covering this but I was hazy on the details. Lo and behold Date and Timestamp and presented as
an example of this problem and what not to do. 
(Timestamp is also a real example of another .equals mistake where it intially had a .equals(Timestamp) method instead of .equals(Object), 
the 'correct' one being added later)

To be fair to writers of [java.sql.Timestamp](http://docs.oracle.com/javase/7/docs/api/java/sql/Timestamp.html) the javadoc does have a pretty big warning on the class and the equals method warning you of the fact that things
are a bit broken and they should not be mixed. 

>Due to the differences between the Timestamp class and the java.util.Date class mentioned above, it is recommended that code not view Timestamp values generically as an instance of java.util.Date. The inheritance relationship between Timestamp and java.util.Date really denotes implementation >inheritance, and not type inheritance.

This provides little solance when someone else has used a Timestamp generically as a Date and you have to figure this out after the fact.

TL:DR Just use the new [java.time](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html)

  
