---
layout: post
title: "Docker and openSUSE: a container full of Geekos"
date: 2013-04-12 14:45
comments: true
categories: [cloud, openSUSE, paas, hackweek]
---

[SUSE's Hackweek #9](http://hackweek.suse.com/) is over. It has
been an awesome week during which I worked hard to make [docker](http://www.docker.io)
a first class citizen on [openSUSE](http://www.opensuse.org/). I also spent
some time working on an openSUSE container that could be used by docker's users.

The project has been tracked on [this](https://github.com/SUSE/hackweek/wiki/docker.io-and-openSUSE)
page of hackweek's wiki, this is a detailed report of what I achieved.

## Installing docker on openSUSE 12.3

Docker has been packaged inside of [this](https://build.opensuse.org/project/show?project=home%3Aflavio_castelli%3Adocker)
OBS project.

So installing it requires just two commands:

```
sudo zypper ar http://download.opensuse.org/repositories/home:/flavio_castelli:/docker/openSUSE_12.3 docker
sudo zypper in docker
```

There's also a [1 Click Install](http://software.opensuse.org/ymp/home:flavio_castelli:docker/openSUSE_12.3/docker.ymp?base=openSUSE%3A12.3&query=docker)
for the lazy ones :)

Zypper will install docker and its dependencies which are:

  * `lxc`: docker's "magic" is built on top of [LXC](http://lxc.sourceforge.net/).
  * `bridge-utils`: is used to setup the bridge interface used by docker's
    containers.
  * `dnsmasq`: is used to start the dhcp server used by the containers.
  * `iptables`: is used to get containers' networking work.
  * `bsdtar`: is used by docker to compress/extract the containers.
  * [aufs3](http://aufs.sourceforge.net/) kernel module: is used by docker to
    track the changes made to the containers.

The `aufs3` kernel module is **not** part of the official kernel and is **not**
available on the official repositories. Hence adding docker will trigger the
**installation of a new kernel package** on your machine.

Both the kernel and the aufs3 packages are going to be installed from the
*home:flavio_castelli:docker* repository but they
are in fact links to the packages created by [Michal Hrusecky](https://build.opensuse.org/home?user=-miska-)
inside of his [aufs project](https://build.opensuse.org/package/show?package=aufs3&project=home%3A-miska-%3Aaufs)
on OBS.

**Note well:** docker works only on 64bit hosts. That's why there are no 32bit
packages.

## Docker appliance

If you don't want to install docker on your system or you are just curious and
want to jump straight into action there's a [SUSE Studio](http://susestudio.com)
appliance ready for you. You can find it [here](http://susestudio.com/a/CZ0T0D/docker).

If you are not familiar with SUSE Gallery let me tell you two things about it:

  * you can download the appliance on your computer and play with it or...
  * you can clone the appliance on SUSE Studio and customize it even further or...
  * you can test the appliance from your browser using SUSE Studio's Testdrive
    feature (no SUSE Studio account required!).

The latter option is really cool, because it will allow you to play with docker
immediately. There's just one thing to keep in mind about Testdrive: outgoing
connections are disabled, so you won't be able to install new stuff (or download
new docker images). Fortunately this appliance comes with the busybox container
bundled, so you will be able to play a bit with docker.

## Running docker

The docker daemon must be running in order to use your containers. The openSUSE 
package comes with a init script which can be used to manage it.

The script is `/etc/init.d/docker`, but there's also the usual symbolic link
called `/usr/sbin/rcdocker`.

To start the docker daemon just type:
```
sudo /usr/sbin/rcdocker start
```

This will trigger the following actions:

  1. The `docker0` bridge interface is created. This interface is bridged
     with `eth0`.
  2. A `dnsmasq` instance listening on the `docker0` interface is started.
  3. IP forwarding and IP masquerading are enabled.
  4. Docker daemon is started.

All the containers will get an IP on the `10.0.3.0/24` network.

## Playing with docker

Now is time to play with docker.

First of all you need to download an image: `docker pull base`

This will fetch the official Ubuntu-based image created by the
[dotCloud](http://www.dotcloud.com/) guys.

You will be able to run the Ubuntu container on your openSUSE host without any
problem, that's LXC's "magic" ;)

If you want to use only "green" products just pull the openSUSE 12.3 container
I created for you:
```
docker pull flavio/openSUSE_12.3
```

Please **experiment a lot with this image** and **give me your feedback**.
The dotCloud guys proposed me to promote it to top-level base image, but I want
to be sure everything works fine before doing that.

Now you can go through the [examples](http://docs.docker.io/en/latest/examples/running_examples/)
reported on the official
[docker's documentation](http://docs.docker.io/en/latest/concepts/containers/).

## Create your own openSUSE images with SUSE Studio

I think it would be extremely cool to create docker's images using
[SUSE Studio](http://susestudio.com).
As you might know I'm part of the SUSE Studio team, so I looked a bit into how
to add support to this new format.

**-- personal opinion --**

There are some technical challenges to solve, but I don't think it would be hard
to address them.

**-- personal opinion --**

If you are interested in adding the docker format to SUSE Studio please create
a new feature request on [openFATE](https://features.opensuse.org/) and vote it!

In the meantime there's another way to create your custom docker images, just
keep reading.

## Create your own openSUSE images with KIWI

[KIWI](http://opensuse.github.io/kiwi) is the amazing tool at the heart of
SUSE Studio and can be used to create LXC containers.

As said earlier docker runs LXC containers, so we are going to follow
[these](http://doc.opensuse.org/projects/kiwi/doc/#sec.lxc.building) instructions.

First of all install KIWI from the [Virtualization:Appliances](https://build.opensuse.org/project/show?project=Virtualization%3AAppliances) project on OBS:
```
sudo zypper ar virtualization:appliances http://download.opensuse.org/repositories/Virtualization:/Appliances/openSUSE_12.3
sudo zypper in kiwi kiwi-doc
```

We are going to use the configuration files of a simple LXC container shipped
the `kiwi-doc` package:
```
cp /usr/share/doc/packages/kiwi/examples/suse-12.3/suse-lxc-guest ~/openSUSE_12_3_docker
```

The `openSUSE_12_3_docker` directory contains two configuration files used by
KIWI (`config.sh` and `config.xml`) plus the `root` directory.

The contents of this directory are going to be added to the resulting container.
It's really important to create the `/etc/resolv.conf` file inside of the
final image since docker is going to mount the `resol.conf` file of the host
system inside of the running guest. If the file is not found docker won't be able
to start our container.

An empty file is enough:
```
touch ~/openSUSE_12_3_docker/root/etc/resolv.conf
```

Now we can create the rootfs of the container using KIWI:
```
sudo kiwi --prepare ~/openSUSE_12_3_docker --root /tmp/openSUSE_12_3_docker_rootfs
```

We can skip the next step reported on KIWI's documentation, that's not needed
with docker because it will produce an invalid container. Just execute the
following command:
```
sudo tar cvjpf openSUSE_12_3_docker.tar.bz2 -C /tmp/openSUSE_12_3_docker_rootfs/ .
```

This will produce a tarball containing the rootfs of your container.

Now you can import it inside of docker, there are two ways to achieve that:

  * from a web server.
  * from a local file.


Importing the image from a web server is really convenient if you ran KIWI
on a different machine.

Just move the tarball to a directory which is exposed by the web server. If you don't
have one installed just move to the directory containing the tarball and type the following
command:
```
python -m SimpleHTTPServer 8080
```
This will start a simple http server listening on port 8080 of your machine.

On the machine running docker just type:
```
docker import http://mywebserver/openSUSE_12_3_docker.tar.bz2 my_openSUSE_image latest
```


If the tarball is already on the machine running docker you just need to type:
```
cat ~/openSUSE_12_3_docker.tar.bz2 | docker import - my_openSUSE_image latest
```

Docker will download (just in the 1st case) and import the tarball. The resulting
image will be named *'my_openSUSE_image'* and it will have a tag named *'latest'*.

The name of the tag is really important since docker tries to run the
image with the *'latest'* tag unless you explicitly specify a different value.


## Conclusion

Hackweek #9 has been both productive and fun for me. I hope you will have fun
too using docker on openSUSE.

As usual, feedback is welcome.


