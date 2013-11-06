---
layout: post
title: My TODO list
---

A [friend](http://about.me/victorhg)
[tweeted](https://twitter.com/victorhg/status/142079139220946944):

> I use pending specs instead of TODO lists... nice tip from @wicz... it
> gives me an interesting view of what is missing.

I told Victor to write pending specs in place of using TODO comments
within his code. I'm glad he liked the idea. I've been using the pending
specs technique for a while and I'd like to share my thoughts why you
should do the same.

First, **you shouldn't add comments unless they are extremely
necessary.** As good developers, we strive to write clean, concise and
self-explanatory code. We always refactor our code to make it as clear
as possible for others to understand and to avoid to use of unnecessary
comments. Using TODO comments go against our principles.

**TODO comments are easily forgotten and overlooked. And your tests are
not!** How often do you `ack TODO`? And how often you run your test
suite? I bet you've answered _sometimes_ and _everytime_. And how about
your CI server? I'm not sure if it knows anything about TODO comments,
but it can certainly cope with pending tests.

**TODO comments are just wrong!** Specially if you're using them to
convey something your code should be doing. What your code needs to do
is its specification, and we all know the right place to put it.

So, are you still using TODO comments? I'd love to hear your opinions on
the comments below.

