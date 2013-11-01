---
layout: post
title: "Detaching Resources: Detached Search Service"
---

In the [previous
post](horewi.cz/detaching-resources-architecture-foundation.html), we
have reviewed the basics about virtualization, learned about Docker and
set up a container running RabbitMQ. In this post, we will create a web
app with a search service. The interesting part is that the former will
be completely decoupled from the latter.

Regardless of being very contrived, the following example gives us a
pretty good bird's-eye view and the basics for building web apps towards
a service-oriented architecture.

## A bit more Docker

First we will need to build a container able to run our ruby services.
Thanks to docker's abilities of reusing images, we can create a new
image to suit our needs pretty easily by inheriting from another image.

I used [this image](https://index.docker.io/u/binaryphile/ruby/) as the
base for my experiment, and added on top of it essential packages like
`curl`, `monit` and `openssh-server`. You can my new image
[here](https://github.com/wicz/detaching-resources/tree/master/v1/docker-ruby).

Given our goal is to create a fully distributed system, we will start
three docker containers, each for the following services: the message
_queue_, the _web_ app and the _search_ engine.

Every time you start a container, docker assigns it a new IP address.
This means we have to get the IP address---via `docker inspect`---of the
_queue_ container, to set it on the other containers.

To avoid this intense manual labor, we can use
[pipework](https://github.com/jpetazzo/pipework/) to set up a private
network. We will define static IP addresses for the containers and share
them using environment variables.

~~~
# spin up containers
$ QUEUE=$(docker run -d wicz/rabbitmq)
$ WEB=$(docker run -e QUEUE_HOST=10.11.12.1 -d wicz/ruby)
$ SEARCH=$(docker run -e QUEUE_HOST=10.11.12.1 -d wicz/ruby)
# set IP addresses for containers
$ pipework br1 $QUEUE  10.11.12.1/24
$ pipework br1 $WEB    10.11.12.2/24
$ pipework br1 $SEARCH 10.11.12.3/24
# enable host to access the private network
$ ifconfig br1 10.11.12.254/24
~~~
{: .bash}

Docker 0.7 has a new feature to
["link"](http://docs.docker.io/en/latest/examples/linking_into_redis/)
containers together. With this, we can skip the pipework set up, because
docker will populate the environments variables with the IP addresses it
automatically assigns and with the exposed ports.

## Testing the services

With the ruby containers and the message queue running, we can set up
the web and search services. We will clone the repository in both
containers and start the respective services.

First we need to clone the repository in both _web_ and _search_
containers:

~~~
$ ssh root@10.11.12.{2,3} git clone https://github.com/wicz/detaching-resources.git /opt/dr
~~~
{: .bash}

For the _search_ container we need to start the consumer daemon:

~~~
search$ cd /opt/dr/v2/search
search$ bundle exec ruby bin/search_consumer.rb
~~~
{: .bash}

And for the _web_ container we need to start the consumer and the web
server:

~~~
web$ cd /opt/dr/v2/web
web$ bundle exec ruby bin/web_consumer.rb
web$ bundle exec ruby web.rb -o 0.0.0.0 &
~~~
{: .bash}

Now point your browser to the web server and you should see a simple
page with a search form. Whatever search you do will redirect to another
page with "Searching..." written on it. Wait a few seconds, refresh this
page and you should see "It works!".

Since I am running the docker container inside a Vagrant VM, I
forwarded a local port to the web server in the _web_ container to be
able to use my browser in the OS X.

~~~
$ vagrant ssh -- -L 8001:10.11.12.2:4567
# open http://localhost:8001
~~~
{: bash}

## Behind the scenes

The example is naive, not to mention it always returns the same result
for every search. But the significance in it, is to note __how__ the
results are generated and the services exchange messages, and not
__what__ they are.

When you hit the "Submit" button, the web app creates a new `Search`
object and invokes the `SearchService` to send it to a queue named
`dr.searches`. Then the client is redirect to a page with the search
results. Since it has not been processed yet, it shows the
"Searching..." placeholder.

On the other container, the search daemon consumes messages from
`dr.searches` queue. It unserializes the message, changes its content
and send it back to another queue named `dr.searches.results`.

The `SearchConsumer` knows nothing about the web app, the database
`SearchRepository` or even the `Search` object. All it knows is how to
handle JSON data. Its perfect because we have created a completely
decoupled solution. As long as we do not change the JSON specification,
we can change whatever we want in the web app and the search engine will
continue working. We can also create other apps that use the same search
engine.

Back in the _web_ container, the `SearchResultsConsumer` receives the
search results from the `dr.searches.results` queue. It unserializes the
JSON and updates the results in the database.

## Conclusion

We have successfully set up our detached services solution. With a few
amendments in the business logic, I think it could be used in production
environments. If you need a similar solution and do not have the time or
energy to improve this one, I suggest giving
[Hutch](https://github.com/gocardless/hutch) from GoCardless a try.

Given the message queue is fully working, we could easily set up another
services like long running jobs, e.g. image manipulation or reports
generation, by just creating new consumers.

Try this examples yourself. I would love to hear your thoughts on the
subject. Comments are open!

