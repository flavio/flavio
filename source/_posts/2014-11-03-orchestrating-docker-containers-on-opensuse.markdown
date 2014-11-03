---
layout: post
title: "Orchestrating Docker containers on openSUSE"
date: 2014-11-03 14:51:42 +0100
comments: true
categories: docker openSUSE orchestration
---

A couple of weeks ago the 11th edition of SUSE's [hackweek](https://hackweek.suse.com/)
took place. This year I decided to spend this time to look into the different
orchestration and service discovery tools build around Docke.

In the beginning I looked into the [kubernetes](https://github.com/GoogleCloudPlatform/kubernetes)
project. I found it really promising but AFAIK not yet ready to be used. It's
still in its early days and it's in constant evolution. I will surely keep
looking into it.

I also looked into other projects like [consul](https://consul.io/) and
[geard](http://openshift.github.io/geard/) but then I focused on using
[etcd](https://github.com/coreos/etcd) and
[fleet](https://github.com/coreos/fleet), two of the tools part of
[CoreOS](https://coreos.com/).

I ended up creating a small testing environment that is capable of running a
simple guestbook web application talking with a MongoDB database. Both the web
application and the database are shipped as Docker images running on a small
cluster.

The whole environment is created by [Vagrant](https://www.vagrantup.com/). That
project proved to be also a nice excuse to play with this tool. I found Vagrant
to be really useful.

You can find all the files and instructions required to reproduce my experiments
inside of [this](https://github.com/flavio/docker-orchestration-demo) repository
on GitHub.


Happy hacking!
