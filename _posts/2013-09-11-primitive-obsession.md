---
layout: post
title: Primitive Obsession
---

[Primitive obsession](http://c2.com/cgi/wiki?PrimitiveObsession) is a
fairly common code smell. It happens when you use primitive data types,
like integers, strings, arrays, to represent concepts from your domain,
e.g. using float to represent money.

The fix is apparently simple. Just create a new class to represent the
concept. [James Shore supports this
idea](http://www.jamesshore.com/Blog/PrimitiveObsession.html). Even if
you end up creating small classes, they will eventually grow.

I believe creating classes thinking they will grow is just
[BDUF](http://c2.com/cgi/wiki?BigDesignUpFront). Dealing with primitive
obsession is much more than just creating new classes. It is based on:

1. deep comprehension of your domain to create meaningful classes that
   represents key concepts to make the code more communicative;
2. removing duplication and augmenting cohesion, leading to a more
   expressive and maintainable code.

J.B. Rainsberger (jbrains) shared his thoughts in this
[post](http://blog.thecodewhisperer.com/2013/03/04/primitive-obsession-obsession/)
and in this great [video](http://vimeo.com/9870277) with Corey Haines.
It is only 14-minutes long, really worth watching.

I like jbrains' thinking about __complex ideas__ and __contexts__. These
concepts are respectively related to creating meaningful classes and the
model domain said above.

[Jeff Atwood](http://www.codinghorror.com/blog/2006/05/code-smells.html)
also takes complexity in account when coping with primitive obsession.
He suggests: _"If your data type is sufficiently complex, write a class
to represent it."_

Duplicate code and low cohesion go hand in hand. Low cohesion makes
related ideas spread wildly through out the codebase, which leads to
duplicate code to handle those values. This symptoms generally are from
primitives types being overused and a new class is begging to be
created.

In practice, I would not create an `Age` class for user profiles in a
video rental software. Using primitive integer in this context works
just fine. On the other hand, I would probably create one for a biology
laboratory software which the age of months or even days can have great
impact on the system.

And even if a concept is not that essential to a domain, e.g. we want to
store the location of restaurants for an online booking service. I would
be tempted to create a `GeoLocation` class, because pairs of `[lat,
lon]` are extremely low cohesive, unexpressive and complex enough, thus
they deserve a class of its own. This will avoid duplication and provide
[better encapsulation and slim
API](http://solnic.eu/2012/06/25/get-rid-of-that-code-smell-primitive-obsession.html).

