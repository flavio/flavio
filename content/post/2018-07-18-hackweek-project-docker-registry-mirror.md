---
layout: post
title: "Hackweek Project Docker Registry Mirror"
date: 2018-07-18T15:48:16+02:00
comments: true
categories:
- docker
- portus
- containers
- openSUSE
---

As part of [SUSE Hackweek 17](https://hackweek.suse.com/) I decided to work on a
fully fledged docker registry mirror.

You might wonder why this is needed, after all it's already possible to run a
docker distribution (aka registry) instance as a
[pull-through cache](https://docs.docker.com/registry/recipes/mirror/). While
that's true, this solution doesn't address the needs of more "sophisticated"
users.

## The problem

Based on the feedback we got from a lot of SUSE customers it's clear that a simple
registry configured to act as a pull-through cache isn't enough.

Let's go step by step to understand the requirements we have.

### On-premise cache of container images

First of all it should be possible to have a mirror of certain container images
locally. This is useful to save time and bandwidth. For example there's no
reason to download the same image over an over on each node of a Kubernetes
cluster.

A docker registry configured to act as a pull-through cache can help with that.
There's still need to warm the cache, this can be left to the organic pull
of images done by the cluster or could be done artificially by some script
run by an operator.

Unfortunately a pull-through cache is not going to solve this problem for
nodes running inside of an *air-gapped* environment. Nodes operated in such an
environment are located into a completely segregated network, that would make it
impossible for the pull-through registry to reach the external registry.

### Retain control over the contents of the mirror

Cluster operators want to have control of the images available inside of the
local mirror.

For example, assuming we are mirroring the Docker Hub, an operator might be
fine with having the `library/mariadb` image but not the `library/redis` one.

When operating a registry configured as pull-through cache, all the images of
the *upstream registry* are at the reach of all the users of the cluster. It's
just a matter of doing a simple `docker pull` to get the image cached into
the local pull-through cache and sneak it into all the nodes.

Moreover some operators want to grant the privilege of adding images to the local
mirror only to trusted users.

### There's life outside of the Docker Hub

The Docker Hub is certainly the most known container registry. However there are
also other registries being used: SUSE operates its own registry, there's Quay.io,
Google Container Registry (aka `gcr`) and there are even user operated ones.

A docker registry configured to act as a pull-through cache can mirror only one
registry. Which means that, if you are interested in mirroring both the Docker
Hub and Quay.io, you will have to run two instances of docker registry
pull-through caches: one for the Docker Hub, the other for Quay.io.

This is just overhead for the final operator.

## A better solution

During the last week I worked to build a PoC to demonstrate we can create a docker
registry mirror solution that can satisfy all the requirements above.

I wanted to have a single box running the entire solution and I wanted all the
different pieces of it to be containerized. I hence resorted to use a
node powered by [openSUSE Kubic](https://kubic.opensuse.org/).

I didn't need all the different pieces of Kubernetes, I just needed `kubelet` so
that I could run it in *disconnected mode*. Disconnected means the `kubelet`
process is not connected to a Kubernetes API server, instead it reads PODs
manifest files straight from a local directory.

### The all-in-one box

I created an openSUSE Kubic node and then I started by deploying a standard
docker registry.
This instance is **not** configured to act as a pull-through cache. However it
is configured to use an external authorization service. This is needed to allow
the operator to have full control of who can *push*/*pull*/*delete* images.

I configured the registry POD to store the registry data to a directory on the
machine by using a Kubernetes [hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)
volume.

On the very same node I deployed the authorization service needed by the
docker registry. I choose [Portus](https://port.us.org), an open source solution
created at SUSE
[a long time ago](http://flavio.castelli.me/2015/04/23/introducing-portus-an-authorization-service-and-front-end-for-docker-registry/).

Portus needs a database, hence I deployed a containerized instance of MariaDB
on the same node. Again I used a Kubernetes `hostPath` to ensure the persistence
of the database contents. I placed both Portus and its MariaDB instance into the
same POD. I configured MariaDB to listen only to localhost, making it reachable
only by the Portus instance (that's because they are in the same
[Kubernetes POD](https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727)).

I configured both the registry and Portus to bind to a local unix socket,
then I deployed a container running HAProxy to expose both of them to
the world.

The HAProxy is the only container that uses the host network. Meaning it's
actually listening on port 80 and port 443 of the openSUSE Kubic node.

I went ahead and created two new DNS entries inside of my local network:

  * `registry.kube.lan`: this is the FQDN of the registry
  * `portus.kube.lan`: this is the FQDN of portus

I configured both the names to be resolved with the IP address of my container
host.

I then used [cfssl](https://github.com/cloudflare/cfssl) to generate a CA and
then a pair of certificates and keys for `registry.kube.lan` and `portus.kube.lan`.

Finally I configured HAProxy to:

  * Listen on port 80 and 443.
  * Automatically redirect traffic from port 80 to port 443.
  * Perform TLS termination for registry and Portus.
  * Load balance requests against the right unix socket using
    the [Server Name Indication (SNI)](https://www.haproxy.com/blog/enhanced-ssl-load-balancing-with-server-name-indication-sni-tls-extension/).

By having dedicated FQDN for the registry and Portus and by using HAProxy's SNI
based load balancing, we can leave the registry listening on a standard port
(`443`) instead of using a different one (eg: `5000`). In my opinion that's a big
win, based on my personal experience having the registry listen on a non standard
port makes things more confusing both for the operators and the end users.

Once I was over with these steps I was able to log into `https://portus.kube.lan`
and perform the usual setup wizard of Portus.

### Mirroring images

We now have to mirror images from multiple registries into the local one, but
how can we do that?

Sometimes ago I stumbled over [this tool](https://github.com/ivanilves/lstags),
which can be used to copy images from multiple registries into a single one.
While doing that it can change the namespace of the image to put it all the
images coming from a certain registry into a specific namespace.

I wanted to use this tool, but I realized it relies on the docker open-source
engine to perform the pull and push operations. That's a blocking issue for me
because I wanted to run the mirroring tool into a container without doing nasty
tricks like mounting the docker socket of the host into a container.

Basically I wanted the mirroring tool to **not** rely on the docker open source
engine.

At SUSE we are already using and contributing to
[skopeo](https://github.com/projectatomic/skopeo), an amazing tool
that allows interactions with container images and container registries without
requiring any docker daemon.

The solution was clear: extend skopeo to provide mirroring capabilities.

I drafted a design proposal with my colleague [Marco Vedovati](https://github.com/marcov),
started coding and then ended up with [this pull request](https://github.com/projectatomic/skopeo/pull/524).

While working on that I also uncovered a [small glitch](https://github.com/containers/image/pull/480)
inside of the `containers/image` library used by skopeo.

Using a patched skopeo binary (which include both the patches above) I then
mirrored a bunch of images into my local registry:

```
$ skopeo sync --source docker://docker.io/busybox:musl --dest-creds="flavio:password" docker://registry.kube.lan
$ skopeo sync --source docker://quay.io/coreos/etcd --dest-creds="flavio:password" docker://registry.kube.lan
```

The first command mirrored only the `busybox:musl` container image from the
Docker Hub to my local registry, while the second command mirrored **all** the
`coreos/etcd` images from the quay.io registry to my local registry.

Since the local registry is protected by Portus I had to specify my credentials
while performing the sync operation.

Running multiple `sync` commands is not really practical, that's why we added
a `source-file` flag. That allows an operator to write a configuration file
indicating the images to mirror. More on that on a dedicated blog post.

At this point my local registry had the following images:

  * `docker.io/busybox:musl`
  * `quay.io/coreos/etcd:v3.1`
  * `quay.io/coreos/etcd:latest`
  * `quay.io/coreos/etcd:v3.3`
  * `quay.io/coreos/etcd:v3.3`
  *  ... more `quay.io/coreos/etcd` images ...

As you can see the namespace of the mirrored images is changed to
include the FQDN of the registry from which they have been downloaded.
This avoids clashes between the images and makes easier to track their
origin.

#### Mirroring on air-gapped environments

As I mentioned above I wanted to provide a solution that could be used also
to run mirrors inside of air-gapped environments.

The only tricky part for such a scenario is how to get the images from the
*upstream registries* into the local one.

This can be done in two steps by using the `skopeo sync` command.

We start by downloading the images on a machine that is connected to the internet.
But instead of storing the images into a local registry we put them on a local
directory:

```
$ skopeo sync --source docker://quay.io/coreos/etcd dir:/media/usb-disk/mirrored-images
```

This is going to copy all the versions of the `quay.io/coreos/etcd` image into
a local directory `/media/usb-disk/mirrored-images`.

Let's assume `/media/usb-disk` is the mount point of an external USB drive.
We can then unmount the USB drive, scan its contents with some tool, and
plug it into computer of the air-gapped network. From this computer we
can populate the local registry mirror by using the following command:

```
$ skopeo sync --source dir:/media/usb-disk/mirrored-images --dest-creds="username:password" docker://registry.secure.lan
```

This will automatically import all the images that have been previously downloaded
to the external USB drive.

### Pulling the images

Now that we have all our images mirrored it's time to start consuming them.

It might be tempting to just update all our `Dockerfile`(s), Kubernetes
manifests, [Helm charts](http://helm.sh/), automation scripts, ...
to reference the images from `registry.kube.lan/<upstream registry FQDN>/<image>:<tag>`.
This however would be tedious and unpractical.

As you might know the docker open source engine has a `--registry-mirror`.
Unfortunately the docker open source engine can only be configured to mirror the
Docker Hub, other external registries are not handled.

This annoying limitation lead me and [Valentin Rothberg](https://github.com/vrothberg)
to create [this pull request](https://github.com/moby/moby/pull/34319)
against the [Moby project](https://mobyproject.org/).

Valentin is also porting the patch against [libpod](https://github.com/projectatomic/libpod),
that will allow to have the same feature also inside of
[CRI-O](http://cri-o.io/) and `podman`.

During my experiments I figured some
[little bits](https://github.com/SUSE/docker-ce/commit/9129064d6d7e6647f27685f7f52d0645bdf2b4ca)
were missing from the original PR.

I built a docker engine with the [full patch](https://github.com/SUSE/docker-ce/tree/suse-v17.09.x-private-registry-with-prefix-and-auth)
applied and I created this `/etc/docker/daemon.json` configuration file:

```json
{
  "registries": [
    {
      "Prefix": "quay.io",
      "Mirrors": [
        {
          "URL": "https://registry.kube.lan/quay.io"
        }
      ]
    },
    {
      "Prefix": "docker.io",
      "Mirrors": [
        {
          "URL": "https://registry.kube.lan/docker.io"
        }
      ]
    }
  ]
}
```

Then, on this node, I was able to issue commands like:

```
$ docker pull quay.io/coreos/etcd:v3.1
```

That resulted in the image being downloaded from `registry.kube.lan/quay.io/coreos/etcd:v3.1`,
no communication was done against `quay.io`. Success!

## What about unpatched docker engines/other container engines?

Everything is working fine on nodes that are running this not-yet merged patch,
but what about vanilla versions of docker or other container engines?

I think I have a solution for them as well, I'm going to experiment a bit
with that during the next week and then provide an update.

## Show me the code!

This is a really long blog post. I'll create a new one with all the configuration
files and instructions of the steps I performed. Stay tuned!

In the meantime I would like to thank
[Marco Vedovati](https://github.com/marcov),
[Valentin Rothberg](https://github.com/vrothberg) for their help with skopeo and
the docker mirroring patch, plus [Miquel Sabaté Solà](https://github.com/mssola/)
for his help with Portus.
