---
layout: post
title: "Write kubectl plugins using WebAssembly and WASI"
date: 2022-04-26T21:00:00+02:00
comments: true
categories:
- kubernetes
- WebAssembly
- WASI
---

A long time passed since the last time I wrote something on this blog! 😅
I haven't been idle during this time, quite the opposite... I kept myself busy experimenting with [WebAssembly](https://webassembly.org/)
and Kubernetes.

You probably have already heard about WebAssembly, but there are high chances
that happened in the context of Web application development. There's however
a new emerging trend that consists of using WebAssembly outside of the browser.

WebAssembly has many interesting properties that make it great for writing
plugin systems or even distributing small computational units (think of FaaS).

WebAssembly is what is being used to power [Kubewarden](https://kubewarden.io),
a project I created almost two years ago at SUSE Rancher, with the help of
[Rafa](https://github.com/ereslibre) and other
[awesome folks](https://github.com/orgs/kubewarden/people).
This is where the majority of my "blogging energies" have been
[focused](https://www.kubewarden.io/blog/).

Now, let's go back to the main focus of today's blog entry: **write kubectl
plugins using WebAssembly**.

## The current state of things

As you all know, kubectl can be easily extended by
[writing external plugins](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/).

These plugins are executables named `kubectl-<name of the plugin>` that, once
put in your `$PATH` can be invoked via `kubectl <name of the plugin>`. This
is the same mechanism used to write `git` plugins.

These plugins can be managed via a tool called [Krew](https://krew.sigs.k8s.io/).

The `kubectl` tool is available for multiple operating systems and architectures,
which means these plugins must be available for many platforms.

## Can WebAssembly help here?

I think writing kubectl plugins using WebAssembly has the following advantages:

* Portability: you don't have to build your WebAssembly-powered plugin for all the
  possible operating systems and architectures the end users might want.
* Security: each WebAssembly module is executed inside of a dedicated sandbox. They
  cannot see other modules or processes running on the host. They also don't have access to the host resources (filesystem, devices, network).
Think of them as containers.
* Size: the majority of kubectl plugins are written using Go, which produces big
  binaries. The average size of a kubectl plugin is around 9 Mb. WebAssembly on
  the other hand, can produce plugins that are half the size.

Last but not least, this sounds like a fun experiment!

## Introducing `krew-wasm`

The idea about writing kubectl plugins with WebAssembly originated during a
brainstorming session I was doing with Rafa about our upcoming talk for
[WasmDay EU 2022](https://sched.co/zgbM). The idea kinda "infected" me, I had to
hack on it ASAP!!! This is how the [`krew-wasm`](https://github.com/flavio/krew-wasm)
project was created.

krew-wasm takes inspiration from [Krew](https://krew.sigs.k8s.io/),
but it **does not** aim to replace it. That's quite the opposite, it's a
complementary tool that can be used alongside with Krew.

The sole purpose of krew-wasm is to manage and execute kubectl plugins written using
[WebAssembly](https://webassembly.org/) and [WASI](https://wasi.dev/).

krew-wasm plugins are WebAssembly modules that are distributed using container
registries, the same infra used to host container images.

krew-wasm can download kubectl WebAssembly plugins from a container registry and
make them discoverable to kubectl.
This is achieved by creating a symbolic link for each managed plugin. This symbolic
link is named `kubectl-<name of the plugin>` but, instead of pointing to the
WebAssembly module, it points to the `krew-wasm` executable.

Once invoked, `krew-wasm` determines its usage mode which could either be a
"direct invocation" (when the user invokes the `krew-wasm` binary to manage plugins)
or it could be a "wrapper invocation" done via `kubectl`.

When invoked in "wrapper mode", krew-wasm takes care of loading the WebAssembly
plugin and invoking it. krew-wasm works as a WebAssembly host, and takes care of
setting up the WASI environment used by the plugin.

I'll leave the technical details out of this post, but if you want you can
find more on the [GitHub project](https://github.com/flavio/krew-wasm) page.

## Some examples

The POC would not be complete without some plugins to run. Guess what, you can find
a one right [here](https://github.com/flavio/kubectl-decoder)!

The `kubectl decoder` plugin dumps Kubernetes Secret objects to the standard output,
decoding all the data that is base64-encoded. On top of that, when a x509 certificate
is found inside of the Secret, a detailed output is shown rather then the not so helpful
PEM encoded representation of the certificate.

If you want to experiment with this idea, you can write your plugins using Rust and [this SDK](https://github.com/flavio/krew-wasm-plugin-sdk-rust).

## Summary

This has been a nice experiment. It proves the combination of WebAssembly and WASI
can be used to produce working kubectl plugins.

What's more interesting is the fact these technologies could be used to extend other Cloud Native projects. Did
someone say [helm](https://helm.sh/docs/topics/plugins/)? 😜

There are however some limitations, mostly caused by the freshness of WASI. These
are documented [here](https://github.com/flavio/krew-wasm#limitations=). However, I'm pretty sure things will definitely improve over the next months.
After all the WebAssembly ecosystem is moving at a fast pace!
