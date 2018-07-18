---
layout: post
title: "Putting openSUSE Docker images on a diet"
date: 2015-07-24 14:21:34 +0200
comments: true
categories:
  - docker
  - opensuse
---

In case you missed the openSUSE images for Docker got suddenly smaller.

During the last week I worked together with [Marcus Schäfer](https://plus.google.com/118327827420943568236/about)
(the author of KIWI) to reduce their size.

We fixed some obvious mistakes (like avoiding to install man pages and
documentation), but we also removed some useless packages.

These are the results of our work:

  * openSUSE 13.2 image: from 254M down to 82M
  * openSUSE Tumbleweed image: from 267M down to 87M

Just to make some comparisons, the Ubuntu image is around 188M while the
Fedora one is about 186M. We cannot obviously compete with images like busybox or
Alpine, but the situation definitely improved!




Needless to say, the new images are already on the DockerHub.

Have fun!
