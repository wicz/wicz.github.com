---
layout: post
title: First mutuo sketches
---

Here is the digital version of the very first sketch I've made of mutuo architecture.

![mutuo prototype](http://www.lucidchart.com/publicSegments/view/4dcbf144-143c-4c11-ab26-08950a56d291/image.png)

I know it's a raw draft, but doing this made me understand things clearly, so I guess it could ease things for you too. I'm not trying to follow any convention here, it's just a free-format sketch. Here we have the four core parts of mutuo: _agent_, _tracker_, _receiver_ and the _app_.

## The agent

The agent is the service which will be running in the peers. He's responsible for doing the heavy duty of the platform, which is monitor the others peers. It will somehow get a list of peers from the tracker, do whatever it has to do and report the results to the receiver.

I'm not being vague here on purpose, it's because there are lots of questions to be answered. For instance, I haven't defined yet how the agent will communicate with the tracker. Almost certainly it will run over HTTP, but I haven't defined if it will be long polling a REST API or use some kind of publish-subscribe mechanism, like [pubsubhubbub](http://code.google.com/p/pubsubhubbub/) or [Faye](http://faye.jcoglan.com/). Probably I'll stick with the pubsub approach.

What I know, is that it's going to be implemented using [node.js](http://nodejs.org/) with [CoffeeScript](http://jashkenas.github.com/coffee-script/). Since I don't want to overwhelm the peers, I believe node.js will give the performance and small footprint I'm looking for.

## The tracker

The tracker is supposed to be very similar to a BitTorrent tracker. It will keep track of the alive peers and serve them as a list.

It will also need to know if a agent who's trying to subscribe is a valid one. I'm thinking in a token-based authentication. Users will register agents in the app, which in turn will generate the tokens for them. That's why the tracker will need access to the app database.

The _status_ database is where the tracker will store the peers' status. It could be a simple container data structure, or I could use something more interesting like [memcached](http://memcached.org/) or [redis](http://redis.io/) or [Kyoto Tycoon](http://fallabs.com/kyototycoon/). It will be much easier to implement multiple trackers in the future using some of these. Now, which one? Probably I'll create an abstraction layer and delegate to the appropriate drivers. Thus I could play with all of them.

I want both tracker and receiver to be written in Ruby, and since both will be struggling with [the C10k problem](http://www.kegel.com/c10k.html), I think it'll be a great opportunity to get  my hands on [Goliath](http://postrank-labs.github.com/goliath/), [EventMachine](https://github.com/eventmachine/eventmachine) or [cool.io](http://coolio.github.com/).

## The receiver

As it's name suggests, it basically receives all the data retrieved by the agents. It will also need to authenticate the agents, so it needs access to the app database (though it's not on the graph, I've just realized it... my bad!)

I'm still thinking on how I'll store the massive amount of data. Could be redis, or for some type of data I could use [RRD](http://en.wikipedia.org/wiki/RRDtool) or [Graphite](http://graphite.wikidot.com/). That would be nice when generating the reports.

Maybe I just answered my own question: It depends! It will depend on the type of the data. Is it time-based? Is it a sort of report, like page load? My brain's hurting now, so let's decide it later.

## The app

The app will be the user interface for all the platform. I'll probably use Rails for this. Haven't thought much about it, but I do know it will have a simple, clean and very user-friendly interface. And maybe some realtime statistics using Websockets? [Pusher](http://pusher.com/) looks interesting.

That's pretty much it, for a start at least. Was it worth reading all this? What do you think? Am I missing something? Please be my guest to post your opinions on the comments.