---
layout: post
title: Composed Method
---

You certainly have read poorly written essays with two, three or even
more ideas all cluttered together in the one paragraph. The same can
happen with code. You certainly have seen or written long methods which
were doing much more than they should.

The mechanics to solve this code smell is fairly simple.  All you need
is to break the long method in smaller ones. The reasoning, however, on
how to break things apart can be tricky, because you do not want to
create meaningless anemic methods.

Kent Beck suggests in Smalltalk Best Practices Patterns to divide your
program into methods that perform one identifiable task and keep all of
the operations in a method at the same level of abstraction. Joshua
Kerievsky, in Refactoring to Patterns, follows the same path suggesting
to transform the logic into a small number of intention-revealing steps
at the same level of detail.

I like to think on the Unix philosophy. A method should do one thing and
do it well. And if we think about SOLID principles, we should apply the
Single Responsability Principle (SRP) to methods as well.

Developing maintainable software involves communicative and expressive
code, and small methods play an import role to achieve this. They
encapsulate the details, making the code easier to read and test.

