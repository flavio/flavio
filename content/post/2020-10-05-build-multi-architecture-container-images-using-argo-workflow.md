---
layout: post
title: "Build multi-architecture container images using argo workflow"
date: 2020-10-05T18:30:00+02:00
comments: true
categories:
- argo
- argo workflow
- ARM
- buildah
- containers
- kubernetes
- multi-architecture container
---

> **Note well:** this blog post is part of a series, checkout the previous episode about
[running containerized buildah on top of Kubernetes](/2020/09/16/build-multi-architecture-container-images-using-kubernetes/).

# Quick recap
I have a small Kubernetes cluster running at home that is made of ARM64 and x86_64
nodes. I want to build multi-architecture images so that I can run them
everywhere on the cluster, regardless of the node architecture.
My plan is to leverage the same cluster to build these container images. That
leads to a "[Inception](https://www.imdb.com/title/tt1375666/)-style" scenario:
building container images from within a container itself.

To achieve that I decided to rely on [buildah](https://github.com/containers/buildah/)
to build the container images. I've shown how run buildah in a containerized
fashion **without** using a privileged container and with a tailor-made AppArmor
profile to secure it.

The previous blog post also showed the definition of Kubernetes PODs that would
build the actual images.

# Today's goals
What I'm going to show today is how to automate the whole building process.

Given the references to the Git repository that provides a container image
definition, I want to automate these steps:

  1. Build the container image on a ARM64 node, push the image to a container
     registry.
  1. Build the container image on a x86_64 node, push the image to a container
     registry.
  1. Create a multi-architecture container image manifest, push it to a container
     registry.

Steps #1 and #2 can be done in parallel, while step #3 needs to wait for the
previous ones to complete.

This kind of automation can be done using some pipeline solution.

# Kubernetes native pipeline solutions

There are many Continuous Integration and Continuous Delivery solutions that
are available for Kubernetes. If you love to seek enlightenment by staring in
front of beautiful logos, checkout [this](https://landscape.cncf.io/category=continuous-integration-delivery&format=card-mode&grouping=category)
portion of the [CNCF landscape](https://landscape.cncf.io/) dedicated to
CI and CD solutions. 🤯

After some research I came up with two potential candidates:
[Argo](https://argoproj.github.io/) and [Tekton](https://argoproj.github.io/).

Both are valid projects with active communities. However I decided to settle
on Argo. The main reason that led to this decision was the lack of ARM64 support
from Tekton.

Interestingly enough, both Tekton and [kaniko](https://github.com/GoogleContainerTools/kaniko)
(which I discussed in the previous blog post of this series) use the same
mechanism to build themselves, a mechanism that can produce
only x86_64 container images and is not so easy to extend.

Argo is an umbrella of different projects, each one of them tackling specific
problems like:

  * [Workflows and pipelines](https://argoproj.github.io/projects/argo)
  * [Continuous delivery](https://argoproj.github.io/projects/argo-cd)
  * [Event handling](https://argoproj.github.io/projects/argo-events)
  * [Enriched Kubernetes Deployment resources](https://argoproj.github.io/argo-rollouts)

The projects above are just the mature ones, many others can be found
under the [Argo project labs](https://github.com/argoproj-labs) GitHub organization.
These projects are not yet considered production ready, but are super interesting.

My favourite ones are:

  * [Notifications for Argo CD](https://github.com/argoproj-labs/argocd-notifications)
  * [Automatic updater of deployed images](https://argocd-image-updater.readthedocs.io/en/stable/)

The majority of these projects don't have ARM64 container images yet, but work
is being done and this work is significantly simpler compared to the one of
porting Tekton. Most important of all: the core projects I need have already
been ported.

# Creating pipelines using Argo Workflow

A pipeline can be created inside Argo by defining a [Workflow resource](https://argoproj.github.io/argo/fields/#workflow).

Copying from the [core concepts](https://argoproj.github.io/argo/core-concepts/)
documentation page of Argo Workflow, these are the elements I'm going to use:

  * *Workflow*: a Kubernetes resource defining the execution of one or more `template`.
  * *Template*: a `step`, `steps` or `dag`.
  * *Step*: a single step of a workflow, typically runs a container based on inputs and capture the outputs.
  * *Steps*: a list of steps.
  * *Directed Acyclic Graph (DAG)*: a set of steps (nodes) and the dependencies (edges) between them.

Spoiler alert, I'm going to create multiple Argo Templates, each one of them focusing on
one specific part of the problem. Then I'll use a DAG to explicit the dependencies
between all these Templates. Finally, I'll define an Argo Workflow to "wrap"
all these objects.

I could show you the final result right away, but you would probably be
overwhelmed by it. I'll instead go step-by-step as I did. I'll start with
a small subset of the problem and then I'll keep building on top of it.

## Porting our build POD to an Argo Workflow

By the end of the previous blog post, I was able to build a container
image by using the following Kubernetes POD definition:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: builder
  annotations:
    container.apparmor.security.beta.kubernetes.io/main: localhost/containerized_buildah
spec:
  nodeSelector:
    kubernetes.io/arch: "amd64"
  containers:
  - name: main
    image: registry.opensuse.org/home/flavio_castelli/containers/containers/buildahimage:latest
    command: ["/bin/sh"]
    args: ["-c", "cd code; cd $(readlink checkout); buildah bud -t guestbook ."]
    volumeMounts:
      - name: code
        mountPath: /code
    resources:
      limits:
        github.com/fuse: 1
  initContainers:
  - name: git-sync
    image: k8s.gcr.io/git-sync/git-sync:v3.1.7
    args: [
      "--one-time",
      "--depth", "1",
      "--dest", "checkout",
      "--repo", "https://github.com/flavio/guestbook-go.git",
      "--branch", "master"]
    volumeMounts:
      - name: code
        mountPath: /tmp/git
  volumes:
  - name: code
    emptyDir:
      medium: Memory
```

These are the key points of this POD:

  * It uses an [Init Container](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
    to retrieve the source code of the container image from a Git repository.
  * A Kubernetes Volume is used to share the source code of the container image to be built
    between the Init Container and the main one.
  * The Git repository details, the image name and other references are all hard-coded.
  * The POD just builds the container image, there's no push action at the end of it.
  * The POD is forcefully scheduled on a x86_64 node; hence this will produce
    only x86_64 container images.
  * The POD requires a Fuse resource, this is required to allow buildah to use
    the performant overlay graph driver.
  * The POD uses a specific AppArmor profile, not the default one provided by
    the container engine.

Starting from something like Argo's ["Hello world Workflow"](https://argoproj.github.io/argo/examples/#hello-world),
we can transpose the POD defined above to something like that:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: simple-build-
spec:
  entry point: buildah
  templates:
  - name: buildah
    metadata:
      annotations:
        container.apparmor.security.beta.kubernetes.io/main: localhost/containerized_buildah
    nodeSelector:
      kubernetes.io/arch: "amd64"
    container:
      image: registry.opensuse.org/home/flavio_castelli/containers/containers/buildahimage:latest
      command: ["/bin/sh"]
      args: ["-c", "cd code; cd $(readlink checkout); buildah bud -t guestbook ."]
      volumeMounts:
        - name: code
          mountPath: /code
      resources:
        limits:
          github.com/fuse: 1
    initContainers:
    - name: git-sync
      image: k8s.gcr.io/git-sync/git-sync:v3.1.7
      args: [
        "--one-time",
        "--depth", "1",
        "--dest", "checkout",
        "--repo", "https://github.com/flavio/guestbook-go.git",
        "--branch", "master"]
      volumeMounts:
        - name: code
          mountPath: /tmp/git
    volumes:
    - name: code
      emptyDir:
        medium: Memory
```

As you can see the POD definition has been transformed into a [Template](https://argoproj.github.io/argo/fields/#template)
object. The contents of the POD `spec` section have been basically copied and pasted under the Template.
The POD annotations have been moved straight under the `template.metadata` section.

I have to admit this was pretty confusing to me in the beginning, but everything became clear once I
started to look at the [field documentation](https://argoproj.github.io/argo/fields/) of the Argo resources.

The workflow can be submitted using the `argo` cli tool:

```bash
$ argo submit workflow-simple-build.yaml
Name:                simple-build-qk4t4
Namespace:           argo
ServiceAccount:      default
Status:              Pending
Created:             Wed Sep 30 15:45:20 +0200 (now)
```

This will be visible also from the Argo Workflow UI:

![Image of Argo Workflow UI](/images/build-multi-arch-containers-post2/argo-workflow-build1.png)

## Refactoring the Argo Workflow

The previous Workflow definition can be cleaned up a bit, leading to
the following YAML file:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: simple-build-
spec:
  entry point: buildah
  templates:
  - name: buildah
    inputs:
      parameters:
      - name: arch
      - name: repository
      - name: branch
      - name: image_name
      - name: image_tag
    metadata:
      annotations:
        container.apparmor.security.beta.kubernetes.io/main: localhost/containerized_buildah
    nodeSelector:
      kubernetes.io/arch: "amd64"
    script:
      image: registry.opensuse.org/home/flavio_castelli/containers/containers/buildahimage:latest
      command: [bash]
      source: |
        set -xe
        cd /code/
        # needed to workaround protected_symlink - we can't just cd into /code/checkout
        cd $(readlink checkout)
        buildah bud -t {{inputs.parameters.image_name}}:{{inputs.parameters.image_tag}}-{{inputs.parameters.arch}} .
        buildah push --cert-dir /certs {{inputs.parameters.image_name}}:{{inputs.parameters.image_tag}}-{{inputs.parameters.arch}}
        echo Image built and pushed to remote registry
      volumeMounts:
        - name: code
          mountPath: /code
        - name: certs
          mountPath: /certs
          readOnly: true
      resources:
        limits:
          github.com/fuse: 1
    initContainers:
    - name: git-sync
      image: k8s.gcr.io/git-sync/git-sync:v3.1.7
      args: [
        "--one-time",
        "--depth", "1",
        "--dest", "checkout",
        "--repo", "{{inputs.parameters.repository}}",
        "--branch", "{{inputs.parameters.branch}}"]
      volumeMounts:
        - name: code
          mountPath: /tmp/git
    volumes:
    - name: code
      emptyDir:
        medium: Memory
    - name: certs
      secret:
        secretName: registry-cert
```

Compared to the previous definition, this one doesn't have any hard-coded
value inside of it. The details of the Git repository, the image name, the container registry,... all
of that is now passed dynamically to the template by using the `input.parameters` map.

The `main` container has also been rewritten to use an Argo Workflow specific field: `script.source`. This
is really handy because it provides a nice way to write a bash script to be executed inside the
container.

The `source` script has been also extended to perform a `push` operation at the
end of the build process.
As you can see the architecture of the image is appended to the tag of the image.
This is a common pattern used when building multi-architecture container images.


One final note about the `push` operation. The destination registry is secured
using a self-signed certificate. Because of that either the CA that signed the
certificate or the registry's certificate have to be provided to buildah.
This can be done by using the `--cert-dir` flag and by placing the certificates
to be loaded under the specified path.
Note well, the certificate files must have the `.crt` file extension otherwise
they won't be handled.

I "loaded" the certificate into Kubernetes by using a Kubernetes secret
like this one:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: registry-cert
  namespace: argo
type: Opaque
data:
  ca.crt: `base64 -w 0 actualcert.crt`
```

As you can see the `main` container is now mounting the contents of the `registry-cert`
Kubernetes Secret under `/certs`.

This time, when submitting the workflow, we must specify its parameters:

```bash
$ argo submit workflow-simple-build-2.yaml \
    -p arch=amd64 \
    -p repository=https://github.com/flavio/guestbook-go.git \
    -p branch=master \
    -p image_name=registry-testing.svc.lan/guestbook-go \
    -p image_tag=0.0.1
Name:                simple-build-npqdw
Namespace:           argo
ServiceAccount:      default
Status:              Pending
Created:             Wed Sep 30 15:52:06 +0200 (now)
Parameters:
  arch:              {1 0 amd64}
  repository:        {1 0 https://github.com/flavio/guestbook-go.git}
  branch:            {1 0 master}
  image_name:        {1 0 registry-testing.svc.lan/guestbook-go}
  image_tag:         {1 0 0.0.1}
```

## Building on multiple architectures

The Workflow object defined so far is still hard-coded to be scheduled only
on x86_64 nodes (see the `nodeSelector` constraint).

I could create a new Workflow definition by copying one shown before and then
change the `nodeSelector` constraint to reference the
ARM64 architecture. However, this would violate the
[DRY principle](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

Instead, I will abstract the Workflow definition by leveraging a feature of
Argo Workflow called
[loops](https://argoproj.github.io/argo/examples/#loops).
I will define a parameter for the target architecture and then I will iterate
over two possible values: `amd64` and `arm64`.

This is the resulting Workflow definition:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: simple-build-
spec:
  entry point: build-images-arch-loop
  templates:
  - name: build-images-arch-loop
    inputs:
      parameters:
      - name: repository
      - name: branch
      - name: image_name
      - name: image_tag
    steps:
    - - name: build-image
        template: buildah
        arguments:
          parameters:
          - name: arch
            value: "{{item.arch}}"
          - name: repository
            value: "{{inputs.parameters.repository}}"
          - name: branch
            value: "{{inputs.parameters.branch}}"
          - name: image_name
            value: "{{inputs.parameters.image_name}}"
          - name: image_tag
            value: "{{inputs.parameters.image_tag}}"
        withItems:
          - { arch: 'amd64' }
          - { arch: 'arm64' }
  - name: buildah
    inputs:
      parameters:
      - name: arch
      - name: repository
      - name: branch
      - name: image_name
      - name: image_tag
    metadata:
      annotations:
        container.apparmor.security.beta.kubernetes.io/main: localhost/containerized_buildah
    nodeSelector:
      kubernetes.io/arch: "{{inputs.parameters.arch}}"
    script:
      image: registry.opensuse.org/home/flavio_castelli/containers/containers/buildahimage:latest
      command: [bash]
      source: |
        set -xe
        cd /code/
        # needed to workaround protected_symlink - we can't just cd into /code/checkout
        cd $(readlink checkout)
        buildah bud -t {{inputs.parameters.image_name}}:{{inputs.parameters.image_tag}}-{{inputs.parameters.arch}} .
        buildah push --cert-dir /certs {{inputs.parameters.image_name}}:{{inputs.parameters.image_tag}}-{{inputs.parameters.arch}}
        echo Image built and pushed to remote registry
      volumeMounts:
        - name: code
          mountPath: /code
        - name: certs
          mountPath: /certs
          readOnly: true
      resources:
        limits:
          github.com/fuse: 1
    initContainers:
    - name: git-sync
      image: k8s.gcr.io/git-sync/git-sync:v3.1.7
      args: [
        "--one-time",
        "--depth", "1",
        "--dest", "checkout",
        "--repo", "{{inputs.parameters.repository}}",
        "--branch", "{{inputs.parameters.branch}}"]
      volumeMounts:
        - name: code
          mountPath: /tmp/git
    volumes:
    - name: code
      emptyDir:
        medium: Memory
    - name: certs
      secret:
        secretName: registry-cert
```

The workflow definition grew a bit. I've added a new template called `build-images-arch-loop`, which is now
the entry point of the workflow. This template performs a loop over the
`[ { arch: 'amd64' }, { arch: 'arm64' } ] ` array, each time invoking the `buildah`
template with slightly different input parameters. The only parameter that changes
across the invocations is the `arch` one, which is used to define the
`nodeSelector` constraint.

Executing this workflow results in two steps being executed at the same time: one building the image on
a random x86_64 node, the other doing the same thing on a random ARM64 node.

This can be clearly seen from the Argo Workflow UI:

![Image of Argo Workflow UI](/images/build-multi-arch-containers-post2/argo-workflow-build3.png)

When the workflow execution is over, the registry will contain two different images:

  * `<image-name>:<image-tag>-amd64`
  * `<image-name>:<image-tag>-arm64`

Now there's just one last step to perform: create a multi-architecture container manifest referencing
these two images.

## Creating the image manifest

The [Image manifest Version 2, Schema 2](https://github.com/docker/distribution/blob/master/docs/spec/manifest-v2-2.md)
specification defines a new type of image manifest called *"Manifest list"*
(`application/vnd.docker.distribution.manifest.list.v2+json`).

Quoting the official specification:

> The manifest list is the "fat manifest" which points to specific image manifests for one or more platforms. Its use is optional, and relatively few images will use one of these manifests. A client will distinguish a manifest list from an image manifest based on the Content-Type returned in the HTTP response.

The creation of such a manifest is pretty easy and it can be done with docker, podman
and buildah in a similar way.

I will still use buildah to create the manifest and push it to the registry
where all the images are stored.

This is the Argo Template that takes care of that:

```yaml
 - name: create-manifest
    inputs:
      parameters:
      - name: image_name
      - name: image_tag
      - name: architectures
    metadata:
      annotations:
        container.apparmor.security.beta.kubernetes.io/main: localhost/containerized_buildah
    volumes:
    - name: certs
      secret:
        secretName: registry-cert
    script:
      image: registry.opensuse.org/home/flavio_castelli/containers/containers/buildahimage:latest
      command: [bash]
      source: |
        set -xe
        image_name="{{inputs.parameters.image_name}}"
        image_tag="{{inputs.parameters.image_tag}}"
        architectures="{{inputs.parameters.architectures}}"
        target="${image_name}:${image_tag}"
        architectures_list=($(echo $architectures | tr "," "\n"))
        buildah manifest create ${target}
        #Print the split string
        for arch in "${architectures_list[@]}"
        do
          arch_image="${image_name}:${image_tag}-${arch}"
          buildah pull --cert-dir /certs ${arch_image}
          buildah manifest add ${target} ${arch_image}
        done
        buildah manifest push --cert-dir /certs ${target} docker://${target}
        echo Manifest creation done
      volumeMounts:
        - name: certs
          mountPath: /certs
          readOnly: true
      resources:
        limits:
          github.com/fuse: 1
```

The template has an input parameter called `architectures`, this string is made
of the architectures names joined by a comma; e.g. `"amd64,arm64"`.

The `script` creates a manifest with the name of the image and then, iterating
over the architectures, it adds the architecture-specific images to it.
Once this is done the manifest is pushed to the container registry.

To make a simple example, assuming the following scenario:

  * We are building the `guestbook-go` application with release `v0.1.0`
  * We want to build the image for the x86_64 and the ARM64 architectures
  * We want to push the images to the `registry.svc.lan` registry

The Argo Template that creates the manifest will pull the following
images:

  * `registry.svc.lan/guestbook-go:v0.1.0-amd64`: the x86_64 image
  * `registry.svc.lan/guestbook-go:v0.1.0-arm64`: the ARM64 image

Finally, the Template will create and push a manifest named `registry.svc.lan/guestbook-go:v0.1.0`.
This image reference will always return the right container image to the node
requesting it.

Adding the container image to the manifest is done with the
`buildah manifest add` command. This command doesn't actually need to have
the container image available locally, it would be enough to reach out to
the registry hosting it to obtain the manifest digest.

In our case the images are stored on a registry secured with
a custom certificate. Unfortunately, the `manifest add` command
was lacking some flags (like the `cert` one); because of that I had
to introduce the workaround  of pre-pulling all the images referenced
by the manifest. This has the side effect of wasting some time, bandwidth and
disk space.

I've submitted patches both to [buildah](https://github.com/containers/buildah/pull/2593)
and to [podman](https://github.com/containers/podman/pull/7576) to enrich their
`manifest add` commands; both pull requests have been merged into the `master`
branches. The next release of buildah will ship with my patch and the
manifest creation Template  will be simpler and faster.

## Explicating dependencies between Argo templates

Argo allows to define a workflow sequence with clear dependencies
between each step. This is done by defining a [DAG](https://argoproj.github.io/argo/examples/#dag).

Our workflow will be made of one Argo Template of type DAG, that will have two tasks:

  1. Build the multi-architecture images. This is done with the Argo
     Workflow loop shown above.
  1. Create the manifest. This task depends on the successful completion of
     the previous one.

This is the Template definition:

```yaml
- name: full-process
  dag:
    tasks:
    - name: build-images
      template: build-images-arch-loop
      arguments:
        parameters:
        - name: repository
          value: "{{workflow.parameters.repository}}"
        - name: branch
          value: "{{workflow.parameters.branch}}"
        - name: image_name
          value: "{{workflow.parameters.image_name}}"
        - name: image_tag
          value: "{{workflow.parameters.image_tag}}"
    - name: create-multi-arch-manifest
      dependencies: [build-images]
      template: create-manifest
      arguments:
        parameters:
        - name: image_name
          value: "{{workflow.parameters.image_name}}"
        - name: image_tag
          value: "{{workflow.parameters.image_tag}}"
        - name: architectures
          value: "{{workflow.parameters.architectures_string}}"
```
As you can see the Template takes the usual series of parameters we've already defined,
and forwards them to the tasks.

This is the **full** definition of our Argo workflow, hold on... this is really long 🙀

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: build-multi-arch-image-
spec:
  ttlStrategy:
    secondsAfterCompletion: 60
  entry point: full-process
  arguments:
    parameters:
    - name: repository
      value: https://github.com/flavio/guestbook-go.git
    - name: branch
      value: master
    - name: image_name
      value: registry-testing.svc.lan/guestbook
    - name: image_tag
      value: 0.0.1
    - name: architectures_string
      value: "arm64,amd64"
  templates:
  - name: full-process
    dag:
      tasks:
      - name: build-images
        template: build-images-arch-loop
        arguments:
          parameters:
          - name: repository
            value: "{{workflow.parameters.repository}}"
          - name: branch
            value: "{{workflow.parameters.branch}}"
          - name: image_name
            value: "{{workflow.parameters.image_name}}"
          - name: image_tag
            value: "{{workflow.parameters.image_tag}}"
      - name: create-multi-arch-manifest
        dependencies: [build-images]
        template: create-manifest
        arguments:
          parameters:
          - name: image_name
            value: "{{workflow.parameters.image_name}}"
          - name: image_tag
            value: "{{workflow.parameters.image_tag}}"
          - name: architectures
            value: "{{workflow.parameters.architectures_string}}"
  - name: build-images-arch-loop
    inputs:
      parameters:
      - name: repository
      - name: branch
      - name: image_name
      - name: image_tag
    steps:
    - - name: build-image
        template: buildah
        arguments:
          parameters:
          - name: arch
            value: "{{item.arch}}"
          - name: repository
            value: "{{inputs.parameters.repository}}"
          - name: branch
            value: "{{inputs.parameters.branch}}"
          - name: image_name
            value: "{{inputs.parameters.image_name}}"
          - name: image_tag
            value: "{{inputs.parameters.image_tag}}"
        withItems:
          - { arch: 'amd64' }
          - { arch: 'arm64' }
  - name: buildah
    inputs:
      parameters:
      - name: arch
      - name: repository
      - name: branch
      - name: image_name
      - name: image_tag
    metadata:
      annotations:
        container.apparmor.security.beta.kubernetes.io/main: localhost/containerized_buildah
    nodeSelector:
      kubernetes.io/arch: "{{inputs.parameters.arch}}"
    volumes:
    - name: code
      emptyDir:
        medium: Memory
    - name: certs
      secret:
        secretName: registry-cert
    script:
      image: registry.opensuse.org/home/flavio_castelli/containers/containers/buildahimage:latest
      command: [bash]
      source: |
        set -xe
        cd /code/
        # needed to workaround protected_symlink - we can't just cd into /code/checkout
        cd $(readlink checkout)
        buildah bud -t {{inputs.parameters.image_name}}:{{inputs.parameters.image_tag}}-{{inputs.parameters.arch}} .
        buildah push --cert-dir /certs {{inputs.parameters.image_name}}:{{inputs.parameters.image_tag}}-{{inputs.parameters.arch}}
        echo Image built and pushed to remote registry
      volumeMounts:
        - name: code
          mountPath: /code
        - name: certs
          mountPath: /certs
          readOnly: true
      resources:
        limits:
          github.com/fuse: 1
    initContainers:
    - name: git-sync
      image: k8s.gcr.io/git-sync/git-sync:v3.1.7
      args: [
        "--one-time",
        "--depth", "1",
        "--dest", "checkout",
        "--repo", "{{inputs.parameters.repository}}",
        "--branch", "{{inputs.parameters.branch}}"]
      volumeMounts:
        - name: code
          mountPath: /tmp/git
  - name: create-manifest
    inputs:
      parameters:
      - name: image_name
      - name: image_tag
      - name: architectures
    metadata:
      annotations:
        container.apparmor.security.beta.kubernetes.io/main: localhost/containerized_buildah
    volumes:
    - name: certs
      secret:
        secretName: registry-cert
    script:
      image: registry.opensuse.org/home/flavio_castelli/containers/containers/buildahimage:latest
      command: [bash]
      source: |
        set -xe
        image_name="{{inputs.parameters.image_name}}"
        image_tag="{{inputs.parameters.image_tag}}"
        architectures="{{inputs.parameters.architectures}}"
        target="${image_name}:${image_tag}"
        architectures_list=($(echo $architectures | tr "," "\n"))
        buildah manifest create ${target}
        #Print the split string
        for arch in "${architectures_list[@]}"
        do
          arch_image="${image_name}:${image_tag}-${arch}"
          buildah pull --cert-dir /certs ${arch_image}
          buildah manifest add ${target} ${arch_image}
        done
        buildah manifest push --cert-dir /certs ${target} docker://${target}
        echo Manifest creation done
      volumeMounts:
        - name: certs
          mountPath: /certs
          readOnly: true
      resources:
        limits:
          github.com/fuse: 1
```

That's how life goes with Kubernetes, sometimes there's just a **lot** of YAML...

![Fortune teller and Kubernetes](/images/build-multi-arch-containers-post2/fortune-teller-yaml.jpg)

Now we can submit the workflow to Argo:

```bash
$ argo submit build-pipeline-final.yml
Name:                build-multi-arch-image-wndlr
Namespace:           argo
ServiceAccount:      default
Status:              Pending
Created:             Thu Oct 01 16:22:46 +0200 (now)
Parameters:
  repository:        {1 0 https://github.com/flavio/guestbook-go.git}
  branch:            {1 0 master}
  image_name:        {1 0 registry-testing.svc.lan/guestbook}
  image_tag:         {1 0 0.0.1}
  architectures_string: {1 0 arm64,amd64}
```

The visual representation of the workflow is pretty nice:

![Image of Argo Workflow UI](/images/build-multi-arch-containers-post2/argo-workflow-build4.png)

As you might have noticed, I didn't provide any parameter to `argo submit`; the
Argo Workflow now has default values for all the input parameters.

## Garbage collector

Something worth of note, Argo Workflow leaves behind all the containers it creates.
This is good to triage failures, but I don't want to clutter my cluster with all
these resources.

Argo provides [cost optimization](https://argoproj.github.io/argo/cost-optimisation/#limit-the-total-number-of-workflows-and-pods)
parameters to implement cleanup strategies. The one I've used above is the
[Workflow TTL Strategy](https://argoproj.github.io/argo/fields/#ttlstrategy).

You can see these lines at the top of the full Workflow definition:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: build-multi-arch-image-
spec:
  ttlStrategy:
    secondsAfterCompletion: 60
```

This triggers an automatic cleanup of all the PODs spawned by the
Workflow 60 seconds after its completion, be it successful or not.

# Summary

Today we have seen how to create a pipeline that builds container images
for multiple architectures on top an existing Kubernetes cluster.

Argo Workflow proved to be a good solution for this kind of automation. There's
quite some YAML involved with that, but I highly doubt over projects 
would have spared us from that.

What can we do next? Well, to me the answer is pretty clear. The definition of
the  container image is stored inside of a Git repository; hence I want to connect
my Argo Workflow to the events happening inside of the Git repository.

Stay tuned for more updates! In the meantime feedback is always welcome.

