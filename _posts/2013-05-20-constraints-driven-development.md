---
layout: post
title: Constraints Driven Development
---

> _". . . every time you write code, you’re experimenting. . . .
> Usually, I’ll make a specific kind of constraint for myself . . . and
> see how it shapes my code. And if I like it, I might roll that into
> the main project or I might just throw it out . . ."_

[Dan Kubb](https://twitter.com/dkubb) said that during the [047 Ruby
Rogues podcast](http://rubyrogues.com/047-rr-coding-disciplines/).

This is, without a doubt, the most inspiring thing I have listened in
the last 12 months. I have been following the same technique for a
while, and I am confident to say it is a great tool to help us hone
our craft.

So, I invite you to experiment new things in your code. It does not
matter if it is an elaborated design pattern or a simple variable
naming rule. Pick a constraint, anyone, and force yourself using it for
the next couple of hours. If you feel it is improving your code, e.g.
easier testing, more readability or expressiveness, loosely coupling,
etc., keep it. But if it is not, drop it, otherwise you may be
wasting time. Next are some examples you can try.

During the same podcast, Dan mentioned he had been using __strict
command query separation (CQS) for every method__ he wrote. And if it
was a command, he was returning `self` to enable chaining.

Dan also commented how he solved the problem of
[Heckle](https://github.com/seattlerb/heckle) taking too long to mutate
his code. He [__split test files per
method__](https://github.com/dkubb/axiom/blob/41991aa4e97baba55dee144ea1aa98ed57d4b2d1/spec/unit/axiom/relation/class_methods/coerce_spec.rb),
so he could mutate specific tiny parts of the system and cut down
execution time.

Speaking of
[CQS](http://en.wikipedia.org/wiki/Command%E2%80%93query_separation),
[Sandi Metz](https://twitter.com/sandimetz) gave a [talk](http://www.justin.tv/confreaks/c/2247122)
[(slides)](https://speakerdeck.com/skmetz/magic-tricks-of-testing-railsconf)
at [RailsConf 2013](http://www.railsconf.com) on how to unit test
command and queries. She proposed some testing rules and defined a
schema called: __The Unit Testing Minimalist__.

Sandi also proposed __four rules for developers__ in the [087 Ruby Rogues
podcast](http://rubyrogues.com/087-rr-book-clubpractical-object-oriented-design-in-ruby-with-sandi-metz/).
The guys from thoughtbot also
[shared](http://robots.thoughtbot.com/post/50655960596/sandi-metz-rules-for-developers)
their experience applying these rules, and how elegantly they had satisfied
one of the constraints.

[Jeff Bay](http://www.xpteam.com) proposed nine "rules of thumb" to
better software design in the essay entitled __Object Calisthenics__ in
the [ThoughtWorks
Anthology](http://pragprog.com/book/twa/thoughtworks-anthology).

These are enough examples to start. You <del>can</del> shall
define your own constraints too. One of my actual constraints is to
completely __take apart routing definitions from controllers specs__.
This helps me define better [URL
design](http://warpspire.com/posts/url-design/) and treat
controllers more like normal classes instead of some magic black box
which receives requests. And I think controllers should not know
anything about routing and HTTP verbs anyway, so it is making sense for
now.

I ended turning this:

```ruby
# spec/controllers/posts_controller_spec.rb
# Tighly coupling between routing and controller actions
describe "GET show" do
  # ...
end
```

into this:

```ruby
# spec/routing/posts_routing_spec.rb
# Routing knows about HTTP verbs
describe "Access a post" do
 it { expect(get("/posts/1")).to route_to("posts#show", id: "1") }
end

# spec/controllers/posts_controller_spec.rb
# show is an ordinary method in the controller
describe "#show" do
  # ...
end
```

