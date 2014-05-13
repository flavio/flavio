---
layout: post
title: "Building docker containers with KIWI"
date: 2014-05-06 22:20
comments: true
categories: [cloud, openSUSE, SUSE, docker, KIWI]
---

I'm pleased to announce [Marcus Schäfer](https://github.com/schaefi) has
just made possible to build docker containers with [KIWI](http://opensuse.github.io/kiwi/).

For those who never heard about it, KIWI is a tool which creates Linux systems
for both physical and virtual machines. It can create openSUSE, SUSE and other types of
Linux distributions.


**Update:** I changed the required version of kiwi and the openSUSE 13.1 template.
Kiwi just received some improvements which do no longer force the container
to include the *lxc* package.

## Why is this important?

As you might know Docker has already its [build system](http://docs.docker.io/reference/builder/)
which provides a really easy way to create new containers. However these containers
must be based on existing ones, which leads to the problem of creating the 1st
parent container. That's where KIWI comes to the rescue.

Indeed Kiwi can be used to build the openSUSE/SUSE/whatever docker images that are
going to act as the foundation blocks of other ones.


## Requirements

Docker support has been added to KIWI 5.06.87. You can find this package inside
of the [Virtualization:Appliances](https://build.opensuse.org/project/show?project=Virtualization%3AAppliances)
project on OBS.

Install the `kiwi` and the `kiwi-doc` packages on your system. Then go to the
`/usr/share/doc/packages/kiwi/examples/` directory where you will find a simple
openSUSE 13.1 template.

## Building the system

Just copy the whole `/usr/share/doc/packages/kiwi/examples/suse-13.1/suse-docker-container`
directory to another location and make your changes.

The heart of the whole container is the `config.xml` file:

```xml
<?xml version="1.0" encoding="utf-8"?>

<image schemaversion="6.1" name="suse-13.1-docker-guest">
  <description type="system">
    <author>Flavio Castelli</author>
    <contact>fcastelli@suse.com</contact>
    <specification>openSUSE 13.1 docker container</specification>
  </description>
  <preferences>
    <type image="docker" container="os131">
      <machine>
        <vmdisk/>
        <vmnic interface="eth0" mode="veth"/>
      </machine>
    </type>
    <version>1.0.0</version>
    <packagemanager>zypper</packagemanager>
    <rpm-check-signatures>false</rpm-check-signatures>
    <rpm-force>true</rpm-force>
    <locale>en_US</locale>
    <keytable>us.map.gz</keytable>
    <hwclock>utc</hwclock>
    <timezone>US/Eastern</timezone>
  </preferences>
  <users group="root">
    <user password="$1$wYJUgpM5$RXMMeASDc035eX.NbYWFl0" home="/root" name="root"/>
  </users>
  <repository type="yast2">
    <source path="opensuse://13.1/repo/oss/"/>
  </repository>
  <packages type="image">
    <package name="coreutils"/>
    <package name="iputils"/>
  </packages>
  <packages type="bootstrap">
    <package name="filesystem"/>
    <package name="glibc-locale"/>
    <package name="module-init-tools"/>
  </packages>
</image>
```

This is a really minimal image which contains just a bunch of packages.


The first step is the creation of the container's root system:

```
kiwi -p /usr/share/doc/packages/kiwi/examples/suse-13.1/suse-docker-container \
     --root /tmp/mycontainer
```

The next step compresses the file system of the container into a single tarball:

```
    kiwi --create /tmp/mycontainer --type docker -d /tmp/mycontainer-result
```

The tarball can be found under `/tmp/mycontainer-result`. This can be imported
into docker using the following command:

```
docker import - myContainer < /path/to/mycontainer.tbz
```

The container named `myContainer` is now ready to be used.

## What's next

In the next days I'll make another blog post explaining how to build docker
containers using KIWI and the [Open Build Service](http://openbuildservice.org/).
This is a powerful combination which allows to achieve continuous delivery.

Stay tuned and have fun!
