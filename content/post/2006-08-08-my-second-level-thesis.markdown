---
author: [Flavio]
date: '2006-08-08 19:57:32'
layout: post
slug: my-second-level-thesis
status: publish
title: My second level thesis
redirect_from: /my-second-level-thesis/
comments: true
categories: [c++, KDE, strigi]
---

Months passed in silence and now you publish two news in a couple of minutes!?
yep, tonight I just want to go over internet and write something here... Wink
It's time to talk about my second level thesis...

### Kat

A couple of months passed since I started seeking something interesting to
challenge with. I found [kat](http://kat.mandriva.com/), an open-source
information retrieval program for KDE. If you don't know what's an information
retrieval program you've just to think about a local google. It's simply a
program that let you search through your local files like google. Other
information retrieval programs are [beagle](http://beagle-
project.org/Main_Page) and [google desktop](http://desktop.google.com/). I
started to study kat's code and I also discovered that its mantainer is a nice
italian guy. Unfortunately kat have some ugly problems and Roberto (the
manteiner) can't fix them because now he's really busy. I was going to
investigate over these problems for fixing them when I discovered a similar
project: **strigi**.

### Strigi

strigi is a really young project created by Jos van den Oever. It's written in
C++ using STL and other external libraries. It runs as a daemon listening over
a socket. In this way you've just to write your custom gui using your favorite
language, nice isn't it? I've contacted Jos and I began to send patches and
add new features to strigi, committing them straight into kde subversion
repository (cool, I'm a bit excited about it :) ). Recently I added the
support for the linux kernel inotify interface, an essential component for
strigi.

I really prefer strigi over kat because:

  * it works
  * it can be run under different window manager
  * in the near future it'll run also under different OS (windows by now, maybe I'll port it to macos)
  * it's highly under development
  
You can find more informations about strigi [here](http://strigi.sf.net).

