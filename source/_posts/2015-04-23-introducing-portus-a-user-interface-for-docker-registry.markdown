---
layout: post
title: "Introducing Portus: an authorization service and front-end for Docker registry"
date: 2015-04-23 21:43:09 +0200
comments: true
categories: docker portus opensuse suse hackweek
---

One of the perks of working at SUSE is hackweek, an entire week you can dedicate
working on whatever project you want. Last week the 12th edition of hackweek
took place. So I decided to spend it working on solving one of the problems many
users have when running an on-premise instance of a Docker registry.

The Docker registry works like a charm, but it's hard to have full
control over the images you push to it. Also there's no web interface
that can provide a quick overview of registry's contents.

So [Artem](https://twitter.com/skullzeek), [Federica](https://twitter.com/eotchi)
and I created the [Portus project](https://github.com/SUSE/Portus) (BTW
*"portus"* is the Latin name for harbour).

## Portus as an authorization service

The first goal of Portus is to allow users to have a better control over the
contents of their private registries. It makes possible to write policies
like:

  * everybody can push and pull images to a certain namespace,
  * everybody can pull images from a certain namespace but only certain users can
    push images to it,
  * only certain users can pull and push to a certain namespace; making all the
    images inside of it invisible to unauthorzied users.

This is done implementing the
[token based authentication system](https://github.com/docker/distribution/blob/master/docs/spec/auth/token.md)
supported by the [latest version](https://github.com/docker/distribution) of the
Docker registry.

### Docker login and Portus authentication in action

<script type="text/javascript" src="https://asciinema.org/a/19171.js" id="asciicast-19171" async></script>


## Portus as a front-end for Docker registry

Portus listens to the [notifications](https://github.com/docker/distribution/blob/master/docs/notifications.md)
sent by the Docker registry and uses them to populate its own database.

Using this data Portus can be used to navigate through all the namespaces and
the repositories that have been pushed to the registry.

[![repositories view](/images/portus/repositories.png)](/images/portus/repositories.png)

We also worked on a client library that can be used to fetch extra
information from the registry (i.e. repositories' manifests) to extend Portus'
knowledge.

## The current status of development

Right now Portus has just the concept of users. When you sign up into Portus a
private namespace with your username will be created. You are the only one with
push and pull rights over it; nobody else will be able to mess with it.
Also pushing and pulling to the "global" namespace is currently not allowed.

The user interface is still a work in progress. Right now you can browse all
the namespaces and the repositories available on your registry. However user's
permissions are not taken into account while doing that.

If you want to play with Portus you can use the development environment managed
by Vagrant. In the near future we are going to publish a Portus appliance and
obviously a Docker image.

Please keep in mind that Portus is just the result of one week of work. A lot of
things are missing but the foundations are solid.

Portus can be found on [this repository](https://github.com/SUSE/Portus) on
GitHub. Contributions (not only code, also proposals, bugs,...) are welcome!
