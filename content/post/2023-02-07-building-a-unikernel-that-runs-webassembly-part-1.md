---
layout: post
title: "Building a unikernel that runs WebAssembly - part 1"
date: 2023-02-07T18:00:00+02:00
comments: true
categories:
- Unikernel
- WebAssembly
- Rust
---

[Hackweek 22](https://hackweek.opensuse.org/) took place last week. During
this week all the SUSE employees are free to hack on whatever they want. This one of
the perks of working at SUSE 😎.

This time my personal project has been about
[building a unikernel that runs WebAssembly](https://hackweek.opensuse.org/22/projects/build-a-unikernel-that-runs-webassembly).

> I wanted this blog post to contain all the details about this journey. However
> I realized this would have been too much for a single post. I hence decided to
> split everything into smaller chunks.
> I'll update this section to keep track of all the posts.
>
> In the meantime, you can find the code of the POC [here](https://github.com/flavio/hermit-wasm).


## Why

There are multiple reasons why I did that, but I don't want to repeat what I
wrote inside of the [project description](https://hackweek.opensuse.org/22/projects/build-a-unikernel-that-runs-webassembly).
Learning and fun goals aside, I think there's actually a good reason to mix
unikernels and WebAssembly.

From the application developer POV, porting/writing an application to the
unikernel is not an easy task. The application and all its dependencies
have to support the target unikernel. Some patching might be required inside
of the whole application stack to make it work.

From the unikernel maintainers POV, they have to invest quite some energies
to ensure any kind of application can run in a seamless way on top of their
platform. They don't know which kind of system primitives the user applications
will leverage, this makes everything harder.

On the other hand, when targeting a WebAssembly platform (think of
[Spin](https://github.com/fermyon/spin)
or [Spiderlightning](https://github.com/deislabs/spiderlightning)), the
application has a clear set of capabilities that have to be provided by
the WebAssembly runtime.

If you look at the Spiderlightning scenario, an application might be requiring
Key/Value store capabilities at runtime. However, how these capabilities are
implemented on the host side is not relevant to the application. That means
the same `.wasm` module can be run by a runtime that implements the K/V store
using Redis or using [Azure Cosmos DB](https://azure.microsoft.com/en-us/products/cosmos-db/).
That would be totally transparent to the end user application.

You might see where I'm going with all that...

If we write a unikernel application that runs WebAssembly modules and supports a
set of Spiderlightning APIs, then the same Spiderlightning application could be
run both on top of the regular `slight` runtime and of this unikernel.

All of that without any additional work from the application developer. The Wasm
module wouldn't even realize that.
The complexity would fall only on the unikernel developer who, whoever, would
have a clear set of functionalities to implement (as opposed to "let's try to
make any kind of application work").

## How

Sometimes ago I stumbled over the [RustyHermit](https://github.com/hermitcore/rusty-hermit)
project, this is a unikernel written in Rust.
I decided to use it as the foundation to write my unikernel application.

Building a RustyHermit application is pretty straightforward. Their documentation,
even though is a bit scattered, is good and their examples help a lot.

The cool thing is that RustyHermit is part of Rust nightly, which makes the whole
developer experience great. It feels like writing a regular Rust application.

Obviously you cannot expect all kind of Rust crates to just work with RustyHermit.
You will see how that influenced the development of the POC.

The next sections go over some of the major challenges I faced during the last week.
I'll share more details inside of the upcoming blog posts (see the disclaimer
section at the top of the page).

### The WebAssembly runtime

Unfortunately [Wasmtime](https://wasmtime.dev/), my favorite WebAssembly runtime,
does not build on top of RustyHermit. Many of its dependencies expect `libc`
or other low level libraries to be around.
The same applies to [wasmer](https://github.com/wasmerio/wasmer).

I thought about using something like [WebAssembly Micro Runtime (WAMR)](https://github.com/bytecodealliance/wasm-micro-runtime),
but I preferred to stick with something written in Rust and have the
"full RustyHermit experience".

After some searching I found [wasmi](https://crates.io/crates/wasmi) a pure Rust
WebAssembly runtime. This works fine on top of RustyHermit, plus its design
is inspired by the one of Wasmtime, which allowed me to reuse a lot of my previous
knowledge.

### WebAssembly Component Model

Spiderlightning leverages the [WebAssembly Component Model](https://github.com/webassembly/component-model)
proposal to offer capabilities to the WebAssembly guests and
to allow the host to consume capabilities offered by the WebAssembly guest.

The communication between the host and the guest happens using types defined
with the [Wasm Interface Type](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md).

To give some concrete examples, the demo I'm going to run leverages the
WebAssembly Component Model in these ways:

* The guest asks the host to start a HTTP server. When doing that, the guest
  informs the host about the HTTP routes that have to be registered, plus
  the names of its internal handlers (the functions that have to be executed).
  This is done by using the [`http-server`](https://github.com/deislabs/spiderlightning/blob/main/wit/http-server.wit)
  types. In this case it's the guest that leverages capabilities offered by the
  host.
* The host handles the incoming HTTP requests using the routing
  information provided by the guest. The http handlers mentioned before are
  functions exposes by the WebAssembly guest. The server is now consuming
  capabilities offered by the guest. The communication is done using the
  [`http-handler`](https://github.com/deislabs/spiderlightning/blob/main/wit/http-handler.wit)
  types.
* Some of the http handlers defined by the guest are also interacting with
  a Key/Value store. Also in this case the guest is leveraging a set of
  capabilities offered by the host. These are defined using the
  [`keyvalue`](https://github.com/deislabs/spiderlightning/blob/main/wit/keyvalue.wit)
  types.

As you can see there are many WIT types involved. For each one of them we
need code both inside of the guest (a SDK basically) and on the host (the
code that implements the guest SDK).
This code can be scaffolded by a cli tool called [`wit-bindgen`](https://github.com/bytecodealliance/wit-bindgen),
which generates host/guest code starting from a `.wit` file.

In this case I only had to implement the host side of these interfaces inside
of the unikernel.

The code generated by `wit-bindgen` is doing low level operations using the
WebAssembly runtime. The code to be scaffolded depends on the programming language
and on the WebAssembly runtime used on the host side.

Obviously the `wasmi` WebAssembly runtime was not supported by `wit-bindgen`,
hence I had to extend `wit-bindgen` to handle it. The code can be found inside of
[this fork](https://github.com/flavio/wit-bindgen/tree/wasmi), under the `wasmi`
branch.

With all of that in place, I scaffolded the host side of the Key/Value capability
and I made a simple implementation of the host traits. The host code was just
emitting some debug information.
I was then able run the vanilla [keyvalue-demo](https://github.com/deislabs/spiderlightning/tree/main/examples/keyvalue-demo)
from the Spiderlightning project. 🥳

## Summary

You made to the bottom of this long post, kudos! I think you deserve a prize for
that, so here we go...

This is a recording of the unikernel application running the Spiderlightning http-server
demo.

![A screencast of the unikernel application running the Spiderlightning http-server demo](/images/unikernel-webassembly/demo.gif "It's alive!")

I hope you enjoyed the reading.
Stay tuned for the next part of the journey. This will cover Rust async, Redis
and some weird errors.
