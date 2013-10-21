---
layout: post
title: "Detaching Resources: Architecture foundation"
---

_Detaching Resources is as series about my personal findings and
experiments on building web applications in a service-oriented fashion.
The title and content are based on "Backing Services" from the
[Twelve-Factor App](http://12factor.net/) methodology._

## A Quick Introduction on Virtualization

[Virtualization](https://en.wikipedia.org/wiki/Virtualization), as the
name suggests, is the act of creating a virtual version of something. It
can be used to create virtual hardware, operating system, memory (swap),
network (VPN), storage (RAID), etc. We will focus on the first two.

Hardware virtualization is probably the most known type. You certainly
have heard of virtual machines (VM) and softwares like Parallels, VMware
and VirtualBox. It allows you run, for instance, Linux inside Mac OS X.
Since it creates an entire virtual computer, it can become very
heavyweight, CPU and memory hungry.

[Operating system (OS)
virtualization](https://en.wikipedia.org/wiki/Operating_system-level_virtualization)
acts in the OS kernel layer. It creates lightweight virtual environments
(VE or containers), because they use the host OS' system call. There is
no need of an intermediate VM; and still being completely isolated. On
the other hand, it is not as flexible as the hardware virtualization,
since the guest and host OS must be the same.
[Linux-VServer](http://linux-vserver.org), [OpenVZ](http://openvz.org/)
and [Linux Containers (LXC)](http://linuxcontainers.org/) are examples
of implementations of this technology.

LXC is the newest implementation of OS virtualization in Linux. It is
based on the [Linux kernel
cgroups](https://en.wikipedia.org/wiki/Cgroups) (control groups) to
limit, account and isolate hardware resources of process groups. LXC is
considered as something between a "chroot on steroids" and a full VM.
Its goal is to create an environment as close as possible to a standard
Linux installation, but without the need of a separate kernel.

[Docker](http://www.docker.io/) provides a high level API on top of LXC.
It automates the creation of containers based on a Dockerfile, making it
repeatable and consistent. It allows container versioning with git-like
capabilities. Allows reuse. Any container can be used as a base image to
create new ones. It encourages sharing. Anyone can upload a container to
their public registry. The
[registry](https://github.com/dotcloud/docker-registry) can also be used
in your private servers.

## Hands on Docker

To install Docker, check its
[documentation](http://docs.docker.io/en/latest/) for your OS. It is
comprehensive and easy to follow.

If you are using a Mac OS X, you need Linux running in a VM. Docker
already makes our lives easier by supporting Vagrant. All you need is to
install [VirtualBox](https://www.virtualbox.org/) and
[Vagrant](http://www.vagrantup.com/) and then

~~~
$ git clone https://github.com/dotcloud/docker.git # clone docker
$ cd docker   # cd to cloned directory
$ vagrant up  # tell vagrant to install the vm
$ vagrant ssh # ssh into your brand new Linux installation
~~~
{: .bash}

The docker daemon should be already running as root. Since the user
`vagrant` is in the group `docker`, it is not necessary to run the
`docker` CLI with sudo.

To get the basics on Docker, I suggest you follow their [interactive
tutorial](http://www.docker.io/gettingstarted/#h_tutorial). You will
learn how to `pull` container images from the [index
repository](https://index.docker.io/), `run` the container, `commit`
your changes and `push` it back to the repository. I will skip straight
to the use of a `Dockerfile`, where things get more interesting.

## The Dockerfile

The `Dockerfile` contains the set of
[steps](http://docs.docker.io/en/latest/use/builder/) docker will
use to reproduce the system images.

~~~
# Dockerfile
FROM ubuntu:latest
MAINTAINER Vinicius Horewicz <vinicius@horewi.cz>

RUN sed -i.bak "s/main$/main universe/" /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y
RUN locale-gen en_US en_US.UTF-8
RUN echo "root:root" | chpasswd

# initctl workaround
RUN dpkg-divert --local --rename --add /sbin/initctl
RUN ln -s /bin/true /sbin/initctl

# prevent services from starting automatically
RUN echo exit 101 > /usr/sbin/policy-rc.d
RUN chmod +x /usr/sbin/policy-rc.d

RUN apt-get install -y curl monit openssh-server rabbitmq-server

# disable upstart for SSH
RUN mv /etc/init/ssh.conf /etc/init/ssh.conf.dist

ADD ./etc/monit/sshd.conf     /etc/monit/conf.d/sshd.conf
ADD ./etc/monit/rabbitmq.conf /etc/monit/conf.d/rabbitmq.conf

# run monit in foreground
CMD ["/usr/bin/monit", "-I"]

EXPOSE 22 :5672
~~~

Dockerfiles follow the format `INSTRUCTION arguments`. __The first
instruction must be__ `FROM`. It specifies the base image from which we
will build.

Images names are in the format `user/image:tag`. Where `user` is the
username of its creator [in the repository]. When it is not specified,
means you are using official images from the Docker team. `image` is the
image name. Images can be tagged, just like git. That is the reason of
the `tag` argument. When omitted, the value `latest` is assumed.

`RUN` instruction runs a command and commits the result at the build
time. Every `RUN` means a new image is committed, which serves as a base
for the next instruction. Thanks to the this
[layered](http://docs.docker.io/en/latest/terms/layer/) approach and to
[AuFS](http://aufs.sourceforge.net/), Docker can cache build steps,
making them blazing fast.

`ADD` commands will copy files from `<src>` to `<dest>`. Source can be a
file or directory relative to the source directory being built or a
remote file URL. Destination is the path in the container. `ADD` is not
cached ([yet?](https://github.com/dotcloud/docker/issues/880)).

`CMD` specifies the command to be executed when the container is started.
__There can exist only one in a Dockerfile. If there are more than one,
only the last will take effect.__

`EXPOSE` sets port to be publicly exposed when running the container. It
is specified as `[PUBLIC:]PRIVATE`. When PUBLIC is omitted, a random port
is assigned. If you want to use the same port for public and private,
you can specify only `:PRIVATE`.

## Running the Container

You start the container with the `docker run` command. It will run the
specified `CMD`. __When this process completes, the container is fully
stopped.__ Think about containers as "process in a box". This is why we
must run the `CMD` in foreground instead of using a daemon or running
`/etc/init.d/monit start` in `CMD`.

One might ask: Why monit? Remember a container run only one process and
stops right after this process ends. Even though a container can
[start](https://github.com/dotcloud/docker/issues/223)
[with](https://github.com/dotcloud/docker/pull/898)
[/sbin/init](https://github.com/dotcloud/docker/pull/1267), docker still
has some [issues with
upstart](https://github.com/dotcloud/docker/issues/2276), so we will use
monit to start and keep our process running.

We still have to do
[some](https://github.com/dotcloud/docker/issues/1024#issuecomment-20018600)
[workarounds](http://jpetazzo.github.io/2013/10/06/policy-rc-d-do-not-start-services-automatically/)
and disable upstart for some services to be able to use init.d scripts.

## The Final Result

The final result is available in this
[repository](https://github.com/wicz/docker-rabbitmq). I have created
basic monitrc files for SSH and RabbitMQ, so monit can start these
services.  If you want to give it a try:

~~~
$ git clone git@github.com:wicz/docker-rabbitmq.git
$ cd docker-rabbitmq
$ docker build -t <yourusername>/rabbitmq .
$ docker run -t <yourusername>/rabbitmq
~~~
{: bash}

Finally, we have set up the architecture foundation. In the next post of
the series we will start implementing services which will exchange
messages through this message queue container.

