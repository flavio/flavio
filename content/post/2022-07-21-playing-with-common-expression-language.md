---
layout: post
title: "Playing with Common Expression Language"
date: 2022-07-21T19:00:00+02:00
comments: true
categories:
- kubernetes
- WebAssembly
- CPP
- Common expression language
---

[Common Expression Language (CEL)](https://github.com/google/cel-spec) is
an expression language created by Google. It allows to define constraints
that can be used to validate input data.

This language is being used by some open source projects and products, like:

* [Google Cloud Certificate Authority Service](https://cloud.google.com/certificate-authority-service/docs/using-cel)
* [Envoy](https://www.envoyproxy.io/docs/envoy/latest/xds/type/matcher/v3/cel.proto)
* There's even a [Kubernetes Enhancement Proposal](https://github.com/kubernetes/enhancements/blob/master/keps/sig-api-machinery/2876-crd-validation-expression-language/README.md)
  that would use CEL to validate Kubernetes' CRDs.

I've been looking at CEL since some time, wondering how hard it would be to
find a way to write
[Kubewarden](https://kubewarden.io)
validation policies using this expression language.

Some weeks ago [SUSE Hackweek 21](https://hackweek.opensuse.org/) took place,
which gave me some time to play with this idea.

This blog post describes the first step of this journey. Two other blog posts
will follow.

## Picking a CEL runtime

Currently the only mature implementations of the CEL language are written in
[Go](https://github.com/google/cel-go) and
[C++](https://github.com/google/cel-cpp).

Kubewarden policies are implemented using [WebAssembly](https://webassembly.org/)
modules.

The official Go compiler isn't yet capable of producing WebAssembly modules that can
be run outside of the browser. [TinyGo](https://tinygo.org/), an alternative
implementation of the Go compiler,
can produce WebAssembly modules targeting the WASI interface. Unfortunately
TinyGo doesn't yet support the whole Go standard library. Hence it cannot be
used to compile [cel-go](https://github.com/google/cel-go).

Because of that, I was left with no other choice than to use the
[cel-cpp](https://github.com/google/cel-cpp) runtime.

C and C++ can be compiled to WebAssembly, so I thought everything would have
been fine.

> Spoiler alert: this didn't turn out to be *"fine"*, but that's for another
> blog post.

## CEL and protobuf

CEL is built on top of [protocol buffer](https://en.wikipedia.org/wiki/Protocol_Buffers)
types.
That means CEL expects the input data (the one to be validated by the constraint)
to be described using a protocol buffer type. In the context of Kubewarden
this is a problem.

Some Kubewarden policies focus on a specific Kubernetes resource; for example,
all the ones implementing Pod Security Policies are inspecting only `Pod` resources.
Others, like the ones looking at `labels` or `annotations` attributes, are instead
evaluating any kind of Kubernetes resource.

Forcing a Kubewarden policy author to provide a protocol buffer definition of
the object to be evaluated would be painful.
Luckily, CEL evaluation libraries are also capable of working against free-form
JSON objects.

## The grand picture

The long term goal is to have a CEL evaluator program compiled into a
WebAssembly module.

At runtime, the CEL evaluator WebAssembly module would be instantiated and would
receive as input three objects:

  * The validation logic: a CEL constraint
  * Policy settings *(optional)*: these would provide a way to tune the
    constraint. They would be delivered as a JSON object
  * The actual object to evaluate: this would be a JSON object

Having set the goals, the first step is to write a C++ program that takes as
input a CEL constraint and applies that against a JSON object provided by the
user.

There's going to be no WebAssembly today.

## Taking a look at the code

In this section I'll go through the critical parts of the code. I'll do that
to help other people who might want to make a similar use of cel-cpp.

There's basically zero documentation about how to use the cel-cpp library. I had to learn how to
use it by looking at the excellent test suite. Moreover, the topic of validating
a JSON object (instead of a protocol buffer type) isn't covered by the tests.
I just found some tips inside of the GitHub issues and then I had to connect the
dots by looking at the protocol buffer documentation and other pieces of cel-cpp.

> **TL;DR** The code of this POC can be found inside of
> [this repository](https://github.com/flavio/cel-evaluator-poc).


### Parse the CEL constraint

The program receives a string containing the CEL constraint and has to
use it to create a `CelExpression` object.

This is pretty straightforward, and is done inside of [these lines]
(https://github.com/flavio/cel-evaluator-poc/blob/0cb59fb7d1efd9ef67b2e943309a21c744bcd479/main/evaluate.cc#L55-L86)
of the `evaluate.cc` file.

As you will notice, cel-cpp makes use of the [Abseil](https://abseil.io/)
library. A lot of cel-cpp APIs are returning [absl::StatusOr<T>](https://abseil.io/docs/cpp/guides/status)
objects. That leads to use a specific pattern, something like:

```cpp
// invoke API
auto parse_status = cel_parser::Parse(constraint);
if (!parse_status.ok())
{
  // handle error
  std::string errorMsg = absl::StrFormat(
      "Cannot parse CEL constraint: %s",
      parse_status.status().ToString());
  return EvaluationResult(errorMsg);
}

// Obtain the actual result
auto parsed_expr = parse_status.value();
```

### Handle the JSON input

cel-cpp expects the data to be validated to be loaded into a `CelValue`
object.

As I said before, we want the final program to read a generic JSON object as
input data. Because of that, we need to perform a series of transformations.

First of all, we need to convert the JSON data into a `protobuf::Value` object.
This can be done using the `protobuf::util::JsonStringToMessage`
[function]
(https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.util.json_util#JsonStringToMessage.details).
This is done by [these lines](https://github.com/flavio/cel-evaluator-poc/blob/0cb59fb7d1efd9ef67b2e943309a21c744bcd479/main/evaluate.cc#L92-L101)
of code.

Next, we have to convert the `protobuf::Value` object into a `CelValue` one.
The cel-cpp library doesn't offer any helper. As a matter of fact, one of
the oldest [open issue](https://github.com/google/cel-cpp/issues/24) of cel-cpp
is exactly about that.

This last conversion is done using a series of helper functions I wrote inside
of the `proto_to_cel.cc` [file](https://github.com/flavio/cel-evaluator-poc/blob/0cb59fb7d1efd9ef67b2e943309a21c744bcd479/main/proto_to_cel.cc).
The code relies on the introspection capabilities of `protobuf::Value` to
build the correct `CelValue`.

### Evaluate the constraint

Once the CEL expression object has been created, and the JSON data has been
converted into a `CelValue, there's only one last thing to do: evaluate
the constraint against the input.

First of all we have to create a CEL `Activation` object and insert the
`CelValue` holding the input data into it. This takes [just few lines of code]
(https://github.com/flavio/cel-evaluator-poc/blob/0cb59fb7d1efd9ef67b2e943309a21c744bcd479/main/evaluate.cc#L105-L117).

Finally, we can use the `Evaluate` method of the `CelExpression` instance
and look at its result. This is done by [these lines of code]
(https://github.com/flavio/cel-evaluator-poc/blob/0cb59fb7d1efd9ef67b2e943309a21c744bcd479/main/evaluate.cc#L119-L142),
which include the usual pattern that handles `absl::StatusOr<T>` objects.

The actual result of the evaluation is going to be a `CelValue` that holds
a boolean type inside of itself.

## Building

This project uses the Bazel build system. I never used Bazel before, which
proved to be another interesting learning experience.

A recent C++ compiler is required by cel-cpp. You can use either gcc (version 9+) or
clang (version 10+).
Personally, I've been using clag 13.

Building the code can be done in this way:

```console
CC=clang bazel build //main:evaluator
```

The final binary can be found under `bazel-bin/main/evaluator`.

## Usage

The program loads a JSON object called `request` which is then embedded
into a bigger JSON object.

This is the input received by the CEL constraint:

```json
{
  "request": < JSON_OBJECT_PROVIDED_BY_THE_USER >
}
```

> The idea is to later add another top level key called `settings`. This one would
> be used by the user to tune the behavior of the constraint.

Because of that, the CEL constraint must access the request values by
going through the `request.` key.

This is easier to explain by using a concrete example:

```console
./bazel-bin/main/evaluator \
  --constraint 'request.path == "v1"' \
  --request '{ "path": "v1", "token": "admin" }'
```

The CEL constraint is satisfied because the `path` key of the request
is equal to `v1`.

On the other hand, this evaluation fails because the constraint is
not satisfied:

```console
$ ./bazel-bin/main/evaluator \
  --constraint 'request.path == "v1"' \
  --request '{ "path": "v2", "token": "admin" }'
The constraint has not been satisfied
```

The constraint can be loaded from file. Create a file
named `constraint.cel` with the following contents:

```cel
!(request.ip in ["10.0.1.4", "10.0.1.5", "10.0.1.6"]) &&
  ((request.path.startsWith("v1") && request.token in ["v1", "v2", "admin"]) ||
  (request.path.startsWith("v2") && request.token in ["v2", "admin"]) ||
  (request.path.startsWith("/admin") && request.token == "admin" &&
  request.ip in ["10.0.1.1",  "10.0.1.2", "10.0.1.3"]))
```

Then create a file named `request.json` with the following contents:

```json
{
  "ip": "10.0.1.4",
  "path": "v1",
  "token": "admin",
}
```

Then run the following command:

```console
./bazel-bin/main/evaluator \
  --constraint_file constraint.cel \
  --request_file request.json
```

This time the constraint will not be satisfied.

> **Note:** I find the `_` symbols inside of the flags a bit weird. But this is
> what is done by the [Abseil flags](https://abseil.io/docs/cpp/guides/flags)
> library that I experimented with. 🤷

Let's evaluate a different kind of request:

```console
./bazel-bin/main/evaluator \
  --constraint_file constraint.cel \
  --request '{"ip": "10.0.1.1", "path": "/admin", "token": "admin"}'
```

This time the constraint will be satisfied.

## Summary

This has been a stimulating challenge.

### Getting back to C++

I didn't write big chunks of C++ code since a long time!
Actually, I never had a chance to look at the latest C++ standards. I
gotta say, lots of things changed for the better, but I still prefer to pick
other programming languages 😅

### Building the universe with Bazel

I had prior experience with `autoconf` & friends, `qmake` and `cmake`, but I
never used Bazel before.
As a newcomer, I found the documentation of Bazel quite good. I appreciated
how easy it is to consume libraries that are using Bazel. I also like how
Bazel can solve the problem of downloading dependencies, something
you had to solve on your own with `cmake` and similar tools.

The concept of building inside of a sandbox, with all the dependencies vendored,
is interesting but can be kinda scary. Try building this
project and you will see that Bazel seems to be downloading the whole universe.
I'm not kidding, I've spotted a Java runtime, a Go compiler plus a lot of other
C++ libraries.

Bazel `build` command gives a nice progress bar. However, the number of tasks to
be done keeps growing during the build process. It kinda reminded me of the old
Windows progress bar!

I gotta say, I regularly have this feeling of "building the universe" with Rust, but
Bazel took that to the next level! 🤯

### Code spelunking

Finally, I had to do a lot of spelunking inside of different C++ code bases:
envoy, protobuf's c++ implementation, cel-cpp and Abseil to name a few.
This kind of activity can be a bit exhausting, but it's also a great way to
learn from the others.

## What's next?

Well, in a couple of weeks I'll blog about my next step of this
journey: building C++ code to standalone WebAssembly!

Now I need to take some deserved vacation time 😊!

⛰️ 🚶👋
