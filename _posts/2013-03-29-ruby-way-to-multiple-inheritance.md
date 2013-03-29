---
layout: post
title: Ruby way to multiple inheritance
---

Ruby achieves multiple inheritance by mixing modules inside other
classes. That is the reason modules used this way are called
"mixins".

Mixins are great to share common code among unrelated classes. It
augments objects behavior while reducing code duplication.

Another great aspect about mixins is that its benefits comes with a
little effort to implement. It is based on a thin contract with the
object it is mixed into, which leads to very loosed coupling. For
example, to mix Comparable or Enumerable, all it needs is the host
object to implement the methods `<=>` or `each`, respectively.

There are details, however, the developer needs to be aware. When
including a mixin, Ruby inserts the module into the class' ancestors
list, as sort of a pseudo superclass. This may make the method lookup
path to become unwieldy and manifest performance issues.

Using mixins wildly may result on "method stuffing". The amount of
inherited methods not related to the problem domain may increase
significantly, ending in bloated contracts. Ruby's Enumerable, for
instance, adds more than 40 public methods to the host object's interface.

There is a risk of name clashing, especially with instance variables of
the host class or other mixins. If the mixin needs its own state, make
sure the instance variables have unique names, e.g. use the module
name as a prefix. The same problem can happen with methods, leading to
unexpected results.

Even though the relationship between the module and host class is looser,
it is still sort of a _is-a_ association. It even returns `true` to the
`is_a?(ModuleName)` method call. Being an inheritance, its use should be
carefully compared against composition.

Developers should notice that mixins must be used to share common code,
and __not__ as "functions bags". When used as the latter---to reduce lines
of code, make classes look slim, and whatnot---it gives a false sense of
organization and makes reading code much harder. The key is to create
mixins based on _semantics_, not on _syntatics_.

Module-to-module dependency is a design smell. It often hints a missing
class, e.g. service, begging to be created. Mixins should also be
orthogonal, just like the other classes in the system.

Mixins are a convenient solution for many cases, but it requires a lot
of discipline to use effectively. Just as another tool in your toolbox,
use it wisely and with the right purpose.

