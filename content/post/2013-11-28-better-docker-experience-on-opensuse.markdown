---
layout: post
title: "Better Docker experience on openSUSE"
date: 2013-11-28
comments: true
categories: [cloud, openSUSE, paas]
---

I don't know if you are aware of that, but Docker 0.7.0 has been released a
couple of days ago.

You can read the full announcement [here](http://blog.docker.io/2013/11/docker-0-7-docker-now-runs-on-any-linux-distribution/),
but let me talk about the biggest change introduced by this release: storage drivers!

Docker has always used [AUFS](http://aufs.sourceforge.net/),
a "[unionfs-like](https://en.wikipedia.org/wiki/UnionFS)" file system, to power
its containers. Unfortunately AUFS is neither part of the official kernel nor
of the openSUSE/SLE one.

[In the past](http://flavio.castelli.name/2013/04/12/docker-and-opensuse/) I had
to build a custom patched kernel to run Docker on openSUSE. That proved to be
a real pain both for me and for the end users.

Now with storage drivers Docker can still use AUFS, but can also opt for something
different. In our case Docker is going to use [thin provisioning](https://lwn.net/Articles/465740/),
a consolidated technology which is part of the mainstream kernel since quite some time.

Moreover Docker's community is working on experimental drivers for BTRFS, ZFS,
Gluster and Ceph.

## What changes now for openSUSE?

Running Docker is incredibly simple now: just use the [1 click install](http://software.opensuse.org/package/docker)
and download it from the *'home:flavio_castelli:docker'* project.

As I said earlier: **no custom kernel is required**. You are going to keep the
one shipped by the official openSUSE repositories.

Just keep in mind that Docker does some initialization tasks on its very first
execution (it configures thin provisioning). So just wait a little before hitting its
API with the Docker cli tool (you will just get an error because `docker.socket`
is not found).

## The road ahead

### Support SLE

Right now Docker works fine on openSUSE 12.3 and 13.1 but not on SLE 11 SP3. During
the next days I'm going to look into this issue. I want to have a stable and working
package for SLE.

### Make it more official

Once the package is proved to be stable enough I'll submit it for inclusion inside
of the [Virtualization](https://build.opensuse.org/project/show/Virtualization)
project on OBS.

So please, checkout Docker package  and provide me your feedback!
