---
layout: post
title: "Chef Solo: Quick and easy cooking for one"
---

Chef is a configuration management tool. It allows us to manage and
automate servers configurations. It uses a Ruby DSL for writing
blueprints---called recipes---that defines actions to be taken on the
remote servers, e.g. install apache.

The [Chef ecosystem](http://docs.opscode.com/) is immense. Servers,
nodes, workstations, agents, cookbooks; myriad of concepts. In this
article I will just scratch the surface and present examples to get you
running as quick as possible.

### About Chef

In a full Chef organization there must exist a _Chef server_ that stores
configuration details of each server---or _node_---in the infrastructure.
Developers/users write _cookbooks_ (and _recipes_) locally on their
_workstations_, and store the former in a _Chef repository_. Eventually,
users use the _knife_ command to upload data from the local Chef
repository to the Chef server.

When running a small-sized infrastructure, chances are you do not need
or want to setup a full Chef organization. Luckly, Chef provides a
slimmer version called _chef-solo_, which allows configuring nodes
without the need of a Chef server.

There is a knife plugin called
[knife-solo](http://matschaffer.github.io/knife-solo/), that makes
working with chef-solo easier and as powerful as with a Chef server.

### Set up your kitchen

To keep everything organized, create a new project (git repo) and add
`knife-solo` to the `Gemfile`.

~~~
$ git init chef-cookbooks
$ cd chef-cookbooks
$ bundle install --path vendor/bundle --binstubs
~~~
{: .bash}

Create a basic `knife` configuration. Knife uses a RSA key pair to
authenticate requests to the Chef Server. Even though you will not have
one, knife still needs this client key. Create one using `ssh-keygen`.
Ultimately, initialize your kitchen. Make sure to enable
[Librarian](https://github.com/applicationsonline/librarian-chef) to
help you manage your cookbooks dependencies.

~~~
$ bin/knife configure --defaults
$ ssh-keygen -f ~/.chef/$USER.pem
$ bin/knife solo init kitchen --librarian
~~~
{: .bash}

### Managing cookbooks

We will create a cookbook to install ruby using rbenv. Add the rbenv
cookbook to the Librarians `Cheffile`. The dependencies will be
installed in the `cookbooks` directory. There is no need to keep track
of these files in your repository. Create your own cookbooks in the
`site-cookbooks` directory.

~~~
# Cheffile
site "http://community.opscode.com/api/v1"

cookbook "rbenv", git: "https://github.com/RiotGames/rbenv-cookbook"
~~~
{: .no-highlight}

~~~
$ cd kitchen
$ ../bin/librarian-chef install
$ ../bin/knife cookbook create ruby -o site-cookbooks
~~~
{: .bash}

I will cover the creation of cookbooks in another article. For now,
will create the bare minimum to continue our example. Following the
[rbenv-cookbook](https://github.com/RiotGames/rbenv-cookbook)
documentation, add the rbenv dependency to your cookbook `metadata.rb`
and in the default recipe use the `rbenv_ruby` command---aka lightweight
resource and provider (LWRP).

~~~
# kitchen/site-cookbooks/ruby/metadata.rb
depends "rbenv"

# kitchen/site-cookbooks/ruby/recipes/default.rb
include_recipe "rbenv::default"
include_recipe "rbenv::ruby_build"
rbenv_ruby "1.9.3-p392" do
  global true
end
~~~
{: .no-highlight}

### Setup node and cook

I like to configure my servers in my SSH client in a way that is easy
to remember and use. Suppose I am working on a project called
PetProject (pp) and I have three EC2 instances: web, app and db. My
configuration looks like this:

~~~
# ~/.ssh/config
Host pp.*
  User ubuntu
  IdentityFile ~/.ssh/petproject.com/ubuntu

Host pp.web
  HostName ec2-54-215-126-10.sa-east-1.compute.amazonaws.com

Host pp.app
  HostName ec2-54-215-126-11.sa-east-1.compute.amazonaws.com

Host pp.db
  HostName ec2-54-215-126-12.sa-east-1.compute.amazonaws.com
~~~
{: .no-highlight}

Now I can easily `ssh pp.web` to connect to my web server. No need to
remember IP addresses, DNS names, user names or passwords.

Use knife-solo to prepare your server. It will install in the server
all dependencies Chef requires, and will create a local configuration
file for this server in `nodes/<hostname>.json`.

Add your recipe to the `run_list` in the JSON file and _cook_ your
server.

~~~
$ ../bin/knife solo prepare pp.app
# edit nodes/pp.app.json as shown below
$ ../bin/knife solo cook pp.app
~~~
{: .bash}

~~~
# kitchen/nodes/pp.app.json
{"run_list":["recipe[ruby]"]}
~~~
{: .json}

Finally, you have got yourself a brand new server with the latest Ruby
1.9.3 installed on it. Even better, now you have an automated task for
it. In case you need to setup an `app02` server, all you need is
_prepare_ and _cook_ it and you are ready to go.

