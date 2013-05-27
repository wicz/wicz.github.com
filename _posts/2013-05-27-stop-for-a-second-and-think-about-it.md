---
layout: post
title: Stop for a second and think about it
---

A while ago I got asked two questions. Two simple, not to say
unpretentious, questions. And that simplicity is the real beauty.  We
all know _the Devil is in the details_, and those questions, they
hide a lot.

I loved answering them---specially the second---because they made me
write down, in a couple of sentences, about things that are so usual in
our busy lives that we sometimes ignore or take for granted. It was a
good reflection exercise I think it is worth sharing.

> _Why is TDD a good thing? Why is it bad?_

I like Kent Beck's definition of TDD as _test-first + incremental
design_. And tests being automated, of course. It is great because it
allows the developer to alter the codebase in a sustainable way, backed
up with the safety net of the tests.

I do not think it is bad at all, unless you are doing something wrong.
For example, if your test suite takes too long to run and you think this
slows your development, chances are this is not the tests fault, but of
a tightly coupled design. Or maybe your unit tests are constantly
hitting the database, it may be a bad implementation. In some rare
cases, it is not worth to automate all your tests upfront, but not
having them is not an option.

Some may say is a waste because it takes too much time writing code the
user will not actively benefit from. Part of this may be true. It may
take some time writing code that is not a feature of the system, but it
pays off with a consistent software design and drastically reducing the
time spent finding and fixing bugs.

> _What could lead you to smash your computer because of us?_

__Rework.__ I hate wasting time, specially when I have to work on things
that should have been done right in the first place, but were not
because someone was lazy or did [not have enough
time](http://gojko.net/2012/05/31/how-to-solve-not-enough-time/).

__Implementing hypothesis that were not validated.__ People tend to
think they have "killer ideas". Do not ask me to do anything until it
has been proven worth doing. Read [Running Lean](http://runninglean.co/)
to understand my point of view.

__Bureaucracy and pointless meetings.__ I am agile, lean and pragmatic.
Let's not waste our precious time on things that will take us from
nothing to nowhere.

