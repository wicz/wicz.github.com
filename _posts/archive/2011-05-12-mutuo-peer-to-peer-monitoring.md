---
layout: post
title: "mutuo: peer-to-peer monitoring"
published: false
---

Meet [mutuo](https://github.com/wicz/mutuo), my brand new pet project! mutuo is a resources monitoring system based on a peer-to-peer architecture.

The first idea is to create something like [pingdom](http://www.pingdom.com), but in a fully distributed architecture similar to a BitTorrent network. We'll have peers (or agents) which subscribes to a tracker and get a list of other subscribed peers to ping. Ultimately the results are sent to another server and the reports accessible via an web app.

That done, we can extend mutuo to support other monitoring services with different protocols like [WatchMouse](http://www.watchmouse.com/en/website_monitoring_features.php), or full page load evaluations like [Monitis](http://portal.monitis.com/index.php/products) and [Page Speed](http://code.google.com/speed/page-speed), and even stress tests like [Load Impact](http://loadimpact.com). I know it may look like I want to conquer the world, but that's not the point. I just want to raise the bar here and put on the table all my options. Like I said, it's a pet project, for fun and no profit =)

One thing I kept asking myself was "What I want with that?". And finally I was able to write my real intentions and motivations:

* __Learn__. I want to learn new techniques, methodologies and technologies. I want to try Node.js, CoffeeScript, Redis and other things, and understand their better use. I want to learn more about event-based programming, concurrency, statistics and how to build and maintain an acceptable heterogeneous platform.

* __Practice__. I want to practice, and I mean, A LOT! I want to try harder and harder. I consider myself an average programmer. I'm not dumb, but I'm not a genius either. I study a lot, but reading with no practicing will lead me to nowhere.

* __Push__. I want to push myself to a new challenge. I want to face new problems and push myself to come up with ideas and solutions. I want to leave this comfort zone I am in.

* __Exposure__. I want to expose my skills, my code and way of thinking. I want people to see what I am capable of and what I need to improve. And I really hope to get some feedback. Maybe I'll also have the chance to pair with some smart people like [Clemens Kofler](http://www.railway.at/2011/04/23/code-with-me/) and [Pat Maddox](http://patmaddox.com/blog/pair-with-me).

* __Give back__. And last, but not least. I want to give back to the community. Be it Ruby, Rails or any other community. I want to give back my support. I want people to learn something from me, I want to help others like I was helped. In order to do that, I'll open-source everything produced, I'll do my best to keep the service running on my own servers and I won't charge a penny!

I guess that's it. I'll start pushing some code to the repo soon and continue to write my thoughts here, so stay tuned.

If you want to help me, feel free to email, fork, send you pull requests, discuss on the comments, whatever it takes to us to communicate. Any feedback will be really appreciated.

Wish me luck!
