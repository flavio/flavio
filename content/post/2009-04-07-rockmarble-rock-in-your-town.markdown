---
author: [Flavio]
date: '2009-04-07 17:40:45'
layout: post
slug: rockmarble-rock-in-your-town
status: publish
title: 'rockmarble: see who is going to rock in your town'
redirect_from: /rockmarble-rock-in-your-town/
comments: true
categories: [c++, qt, qjson, last.fm, KDE, marble]
---

During the last weekend I hacked a bit on rockmarble and I added a new
feature: retrieve all the events happening in a certain city.

As usual data is provided by last.fm, which should return also the events
_"near" _the specified city (don't ask me to define a value for _near_ :) ).

I have created new openSUSE packages, this time everything should work fine.
Just make sure to remove all [qjson](http://qjson.sourceforge.net/) packages
before installing this release (in fact all the previous problems were due to
packaging errors of qjson, now I have created new packages called libqjson0).

Packages are available for openSUSE 11.0, 11.1 and Factory (both for i586 and
x86_64 architectures).

One last news: rockmarble has also a new
[site](http://rockmarble.sourceforge.net), something slightly better than
github wiki ;)

[![](/images/rockmarble/rockmarble_1click.png)](http://cloud.github.com/downloads/flavio
/rockmarble/rockmarble.ymp)

