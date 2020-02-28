---
layout: post
title: "Semantic versioning and containers"
date: 2020-02-27T21:48:16+02:00
comments: true
categories:
- docker
- containers
- kubernetes
---

Developers are used to express the dependencies of their programs using
[semantic versioning](https://semver.org/) constraints.

For example a Node.js application relying on left-pad could force only certain
versions of this library to be used by specifying a constraint like
`>= 1.1.0 < 1.2.0`. This would force npm to install the latest version of the
library that satisfies the constraint.

How does that translates to containers?

Imagine the following scenario: a developer deploys a containerized application
that requires a Redi database. The developer deploys the latest version
of the redis container (eg: `redis:4.0.5`), ensures his application works fine
and then moves to do other things.

After some weeks a security issue/bug is found inside of Redis and a new patched
release takes place. Suddenly the deployed container is outdated. How can the
developer be aware a new v4 release of Redis is available? Wouldn't be even
better to have some automated tool taking care of this upgrade?

After some more weeks a new minor release of Redis is released (eg: `4.1.0`).
Is it safe to automatically update to a new minor release of Redis, is the
developer application going to work as expected?

Some container images have special tags like `v4` or `v4.1` and the developer
could just leverage them to kinda pinpoint the redis container to a more
delimited set of versions. However using these tags reduces reproducibility
and debuggability.

Let's imagine the redis image being deployed is `redis:v4.1` and everything is
working as expected. Assume after some time the developer (or some automated tool)
pulls a new version of the `redis:v4.1` image and suddenly the application
has some issues. How can the developer understand what really changed?
Wouldn't it be great to be able to say something like "everything worked fine
with `redis:4.1.0` but it broke when I upgraded to `redis:4.1.9`"?


There are some tools that can be used to find and automatically update old
container images: [Watchtower](https://containrrr.github.io/watchtower/)
and [ouroboros](https://github.com/pyouroboros/ouroboros). However none of them
allows the flexibility I was looking for (in terms of checks), plus they are
both tailored to work only against docker.

Because of that, during the 2020 edition of
[SUSE Hackweek](https://hackweek.suse.com/), I spent some time working on
a different solution to this use case.

## Introducing fresh-container

[fresh-container](https://github.com/flavio/fresh-container) is a tool that
can be used to see if a container can be updated to a more recent release.

fresh-container is different compared to Watchtower and ouroboros because it
relies on [semantic versioning](https://semver.org) to process container
image tags.

Semantic versioning is used to express the version constraints a container
version must satisfy. This gives more flexibility, for example take a look
at the following scenarios:

  * I'm fine with any release of Redis that is part of the v4 code stream: `>= 4.0.0 < 5.0.0`
  * I'm fine only with patch releases of Redis that belong to the 4.1 code stream: `>= 4.1.0 < 4.2.0`
  * I'm don't want any release of Redis after v6: `< 6.0.0`

### CLI mode

`fresh-container` can be run as a standalone program:

```shell
$ fresh-container check --constraint ">= 1.9.0 < 1.10.0" nginx:1.9.0

The 'docker.io/library/nginx' container image can be upgraded from the '1.9.0' tag to the '1.9.15' one and still satisfy the '>= 1.9.0 < 1.10.0' constraint.
```

Behind the scenes fresh-container will query the container registry hosting
the image to gather the list of all the available tags. The tags that do not
respect semantic versioning will be ignored and finally the tool will
evaluate the constraint provided by the user.

It can also generate computer parsable output by producing a JSON response:

```shell
$ fresh-container check -o json --constraint ">= 1.9.0 < 1.10.0" nginx:1.9.0

{
  "image": "docker.io/library/nginx",
  "constraint": ">= 1.9.0 < 1.10.0",
  "current_version": "1.9.0",
  "next_version": "1.9.15",
  "stale": true
}
```

### Server mode

Querying the remote container registries to fetch all the available tags of a
container image is an expensive operation. That gets even worse when multiple
containers have to be inspected on a regular basis.

The fresh-container binary can operate in a server mode to alleviate this issue:

```shell
$ fresh-container server
```

This will start a web server offering a simple REST API that can be used to
perform queries. The remote tags of the container images are cached inside of
an in-memory database to speed up constraint resolution.

It's possible to run `fresh-container check` against a `fresh-container` server
to perform faster queries by using the `--server <http://fresh-container-server>`
flag.

## Kubernetes integration

fresh-container is a tool built to serve one specific use case: your provide some
data as input and, as output, it will tell you if the container image can be
updated to a more recent version.

It's main goal is to be leveraged by other tools to build something bigger like
[fresh-container-operator](https://github.com/flavio/fresh-container-operator).

This is a kubernetes operator that, once deployed inside of a kubernetes cluster,
will look at all the kubernetes deployments running inside of it and finds the
ones having stale containers.

The operator can also automatically update these outdated deployments to use
the latest version of the container images that satisfy their requirements.

### Usage

How does it work? First of all you have to enrich your deployment definition
by adding some ad-hoc annotations.

For each container image used by the deployment you have to specify the semantic
versioning constraint that has to be used to evaluate their "freshness".

Take a look at the following example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  annotations:
    fresh-container.autopilot: "false"
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        fresh-container.constraint/nginx: ">= 1.9.0 < 1.10.0"
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.0
        ports:
        - containerPort: 80
```

In this case the operator will look at the version of the nginx container
in use and evaluate it against the `>= 1.9.0 < 1.10.0` constraint.

**Note well:** deployments that do not have any
`fresh-container.constraint/<container name>` will be ignored by the operator.

### Find stale deployments

The operator adds the special label `fresh-container.hasOutdatedContainers=true`
to all the deployments that have one or more stale containers inside of them.

This allows quick searches against all the deployments:

```shell
$ kubectl get deployments --all-namespaces -l fresh-container.hasOutdatedContainers=true
NAMESPACE   NAME               READY   UP-TO-DATE   AVAILABLE   AGE
default     nginx-deployment   1/1     1            1           19m
```

### Why is a deployment stale?

The details about the stale containers are added by the operator as annotations
of the deployment:

```shell
kubectl describe deployments.apps nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 27 Feb 2020 10:32:55 +0100
Labels:                 fresh-container.hasOutdatedContainers=true
Annotations:            deployment.kubernetes.io/revision: 1
                        fresh-container.autopilot: false
                        fresh-container.lastChecked: 2020-02-27T09:45:07Z
                        fresh-container.nextTag/nginx: 1.9.15
```

For each stale container the operator adds an annotation with
`fresh-container.nextTag/<container name>` as key and the tag of the most recent
container that satisfies the constraint as value.

In the example above you can see that the nginx container inside of the
deployment can be updated to the `1.9.15` tag while still satisfying the
`>= 1.9.0 < 1.10.0` constraint.

## Automatic upgrades

The next step is to allow fresh-container-operator to update all the
deployments that have stale containers.

This is not done by default, but can be enable on a per-deployment basis
by adding the `fresh-container.autopilot=true` annotation inside of the
deployment metadata.

## What comes next

As I stated in the beginning I created these projects during the 2020 edition of
[SUSE Hackweek](https://hackweek.suse.com/). They are early prototypes that
need more love.

I would be happy to hear what you think about them. Feel free to leave a comment
below or open an issue on their GitHub projects:

  * [fresh-container](https://github.com/flavio/fresh-container)
  * [fresh-container-operator](https://github.com/flavio/fresh-containeri-operator)
