---
layout: post
title: "Juggling Value Objects, REST APIs and HTTP bodies"
---

Domain-driven design (DDD) and REST are well-known concepts and
practices used by most of the web developers. Both help to achieve
better software by promoting a maintainable codebase and expressive
APIs. However, there are some cases these ideas do not play well
together.

## A Quick DDD Recap

DDD defines artifacts to express models from the business domains. We
will focus on three: _Entities_, _Value Objects_ and _Aggregates_.

An _Entity_ is an object that is defined by its identity. An identity is
an identifier that distinguish each entity uniquely, e.g. the social
security number of an individual or the primary key in a relational
database.

Suppose the following example. We retrieve the same `User (id=1)` twice
from the database. The `eql?` method returns `true` despite the objects
are allocated in different portions of the memory. That happens because
the comparison is based on the `User#id`.

```ruby
user_a = User.find(1)
user_b = User.find(1)

user_a.eql?(user_b) # true
```

A _Value Object_ is an object that is defined by the values of its
attributes. It does not have a conceptual identity. One good example is
a `Location(lat, lng)` object. It does not need a unique identifier
because its attributes---latitude and longitude---already define it.

They are also immutable, i.e. we cannot change an attribute. Should we
need different attributes, we have to instantiate a new object.

When comparing value objects, we take into account the values of the
attributes.

```ruby
caffe_nero  = Location.new(45, -90)
good_coffee = Location.new(45, -90)

caffe_nero.eql?(coffee_shop) # true
```

An _Aggregate_ is a collection of objects bound together by a root
entity called _aggregate root_. The objects in this collection may be
either _entities_ or _value objects_, e.g. a todo list with a collection of
tasks or a line chart with a collection of points, respectively.

Another property of an _Aggregate_ is that the root acts like a boundary
that separates objects in the inside---the collection---from the
outside. Also, it is the only object that holds references to the
internal objects. Therefore, when the root is deleted, all objects from
the aggregate are removed as well.

When in a database, only the root should be retrieved directly. The
aggregate objects should be obtained through associations.

```ruby
todo  = Todo.find(1)
tasks = todo.tasks
```

## Time to REST

REST works fine with _entities_ because all operations related to them
are based on their identities. However, things can get unease with
_value objects_ given the lack of a unique identifier.

Suppose we are creating an app to keep track of heights and weights of
people. A person has a set of body measurements along time. Our REST
(JSON) API and design would look like this:

```console
POST    /people
GET     /people/:id
DELETE  /people/:id
POST    /people/:id/measurements
DELETE  /people/:id/measurements
```

![UML class diagram](/assets/images/18eedfcb.png){: .center}

`Person` objects are _entities_ and `Measurement` are _value objects_.
In the database we will have a `people` table and the `measurements`
will be serialized in a column within the table.

Creating a new measurement is pretty straightforward. Send a `POST` to
the endpoint with the values in the request body. But how do we delete a
measurement?

## Value Objects and the HTTP spec

As said earlier, _value objects_ are identified by the values of their
attributes, so we have to find a way to pass these attributes as
parameters in the HTTP request.

Sending simple parameters in a `GET` or `DELETE` is very easy. The
parameters can be passed in the query string as key/value pairs.

However, passing complex objects is slightly complex, because the HTTP
spec does not guarantee the presence the of a request body for those
types of requests.

From [RFC 2616, HTTP/1.1, Section 4.3](http://tools.ietf.org/html/rfc2616#section-4.3)

> A server SHOULD read and forward a message-body on any request; if the
  request method does not include defined semantics for an entity-body,
  then the message-body SHOULD be ignored when handling the request.

From [RFC 7231, HTTP/1.1 Semantics and Content, Section 4.3.5](http://tools.ietf.org/html/rfc7231#section-4.3.5)

> A payload within a DELETE request message has no defined semantics;
  sending a payload body on a DELETE request might cause some existing
  implementations to reject the request.

Although the spec does not forbid the request body for `DELETE` and
`GET` requests, it indicates that it SHOULD be ignored.

## What should I do then?

One option is to pass all parameters as simple query string key/value
pairs, e.g.

```console
DELETE /people/:id/measurements?examined_at=2014-11-22&height=185&weight=105
```

Another is to follow what the [Restful Objects
Specification](http://restfulobjects.org) suggests, that is parameters
should be serialized and encoded within the URL. For example, to delete

```json
{
  "measurement": {
    "examined_at": "2014-11-22",
    "height": 185,
    "weight": 105
  }
}
```

We would do

```console
DELETE /people/:id/measurements?%7B%0A%20%20%22measurement%22%3A%20%7B%0A%20%20%20%20%22examined_at%22%3A%20%222014-11-22%22%2C%0A%20%20%20%20%22height%22%3A%20185%2C%0A%20%20%20%20%22weight%22%3A%20105%0A%20%20%7D%0A%7D%0A
```

One last option would be to send the payload as the request body
regardless what the HTTP spec say.

The first two solutions strictly follow the HTTP spec. We still need to
make sure the length of the request URI does not exceed the limit
accepted by the
[HTTP](http://nginx.org/en/docs/http/ngx_http_core_module.html#large_client_header_buffers)
[server](http://httpd.apache.org/docs/2.4/mod/core.html#limitrequestline)---very
unlikely though. On the down side, server and client need an extra
effort massaging the data.

Sending complex objects in the request body is much simpler and, IMHO,
offers a more cohesive solution. `GET`, `POST`, `PUT/PATCH` and `DELETE`
play by the same rules: they all send JSON right into the wire.

Be aware though, some servers and clients may not preserve the request
body (which could lead to long daunting debugging hours). Special
attention to proxies, they may also silently strip out the body.

## Final Words

In case you decide to take the risk and ignore the HTTP spec for the
sake of a simpler solution you are welcome. You can, like me, stand on
the shoulders of giants like elasticsearch ([request body
search](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-request-body.html),
[delete by
query](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/docs-delete-by-query.html)).

I am not advocating against query strings. It just seems to me that when
working with __complex objects__ in a JSON API, using the request body
is a better solution.

Ultimately, as always, YMMV. As long as you are aware of the possible
architectural issues and inform your API consumers, you should not have
a problem.

