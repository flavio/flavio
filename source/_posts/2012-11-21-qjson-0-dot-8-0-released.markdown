---
author: Flavio
layout: post
title: "QJson 0.8.0 released"
date: 2012-11-21 22:03
comments: true
categories: [qjson, qt, kde]
---

Almost three years passed since latest release of QJson.
A lot of stuff happened in my life and QJson definitely paid for that.
I have to admit I'm a bit ashamed.

So here we go, QJson 0.8.0 is out!

## What changed

A lot of bugs has been smashed during this time, this new release will fix
issues like [this](https://bugs.gentoo.org/show_bug.cgi?id=428256) one and
[this](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=687537) in a nicer way.

QJson's API is still backward compatible, while the ABI changed.

## Symbian support

Some work has also been done to get QJson work on the Symbian platform. The
development happened a long time before [Symbian was declared dead](http://www.wired.com/business/2011/02/nokia-burning-platform/all/).

Currently I do not offer any kind of support for the Symbian platform because
IMHO Symbian development is a mess and given the current situation in the mobile
world I don't see any point in investing more efforts on that.

Obviously Symbian patches and documentation are still accepted, as long as they
don't cause issues to the other target platforms.

## QMake support

QJson always used cmake as build system but since some Windows developers
had problems with it I decided to add some `.pro` files. That proved to be a
bad choice for me since I had to support two build systems. I prefer to invest
my time fixing bugs in the code and adding interesting features rather then
triaging qmake issues on Windows. Hence I decided to remove them from git.

If you are a nostalgic you can still grab these files from git. They
have been removed with commit [66d10c44dd3b21](https://github.com/flavio/qjson/commit/66d10c44dd3b21d693933f320c32b3c9a175a693).

## Relocating to Github

I decided to move QJson's code from Gitorious to [Github](https://github.com/flavio/qjson).
[Github's issue sysyem](https://github.com/flavio/qjson/issues) is going to replace Sourceforge's bug tracking system.

I currently use Github a lot, both for personal projects and for work, and I simply love it.
I think it offers the best tools in the market and that's really important to me.

[QJson's website](http://qjson.sourceforge.net) and mailing lists are still going to be hosted on Sourceforge.

<hr>
<br>
I think that's all from now. If you want more details about the changes introduced
take a look at the [changelog](https://github.com/flavio/qjson/blob/master/ChangeLog)
or checkout [QJson's website](http://qjson.sourceforge.net).

