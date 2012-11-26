---
author: Flavio
date: '2009-04-02 08:51:43'
layout: post
slug: rockmarble-follow-artists-tours-with-marble
status: publish
title: 'rockmarble: how to follow your favourite artists tour with Marble'
comments: true
categories: [c++, qt, qjson, last.fm, KDE, marble]
---

During the last weekend I wanted to have some fun with
[QJson](http://qjson.sourceforge.net/). So I came out with this idea: retrieve
from [last.fm](http://www.last.fm/home) the tour dates of my favourite artists
and display the locations using [Marble](http://edu.kde.org/marble/).

After some hacking I created this small application: _rockmarble_...

{% img /images/rockmarble/flow.png %}

If you have a last.fm account rockmarble will import your favourite artist
list. Otherwise you can add one artists at a time.

  
The tour location will be displayed inside Marble, using
[openstreetmap](http://www.openstreetmap.org/).

## Requirements

In order to build/run it you will need:

  * [Qt4](http://www.qtsoftware.com/downloads)
  * [marble](http://edu.kde.org/marble/)
  * [QJson](http://qjson.sourceforge.net/)

## Installation

You can grab the source code of rockmarble
[here](http://github.com/flavio/rockmarble/tree/master).

If you are an openSUSE user you can use 1click install:

[![](/images/rockmarble/rockmarble_1click.png)](http://cloud.github.com/downloads/flavio
/rockmarble/rockmarble.ymp)

## Issues

### Geolocalization

The geolocalization data are given by last.fm, so if you discover that
Metallica are going to give a concert in the middle of the Pacific Ocean
please don’t bother me :)

### Special names

It seems that QJson doesn’t handle properly special characters. Maybe you will
some artist with a blank name. I’m going to fix this issue asap.

## More details

Visit [rockmarble website](http://rockmarble.sourceforge.net)

## Future

Who wants to integrate it into amarok's context view? ;)

