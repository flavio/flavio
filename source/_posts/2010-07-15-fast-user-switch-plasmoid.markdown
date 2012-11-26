---
author: Flavio
date: '2010-07-15 07:23:11'
layout: post
slug: fast-user-switch-plasmoid
status: publish
title: Fast user switch plasmoid
comments: true
categories: [KDE, plasma]
---

Last week my mother in law started to share her Linux laptop with my wife.
Suddenly my wife asked me how she could switch from one user session to
another. She was looking for something similar to [OS X fast user
switch](http://bit.ly/auzc56) feature but she couldn't find it. In fact there
wasn't a fast and easy way to switch between users' sessions with KDE,
until... now :)

Let me introduce my first plasmoid: the fast user switch plasmoid. It's a
simple icon in the panel that allows users to swich to another open session or
to open a new login page. Here you can see the mandatory screenshots.

{% img /images/fast_user_switch/fastuserswitch02.png %}
{% img /images/fast_user_switch/fastuserswitch01.png %}

You can find the source code
[here](http://gitorious.org/fast_user_switch#more). Binary packages for
openSUSE are already available on the [build
service](http://software.opensuse.org/search?q=plasmoid-
fastuserswitch&baseproject=ALL&lang=en&exclude_debug=true).

## One last thought about KDM

I think that KDM should allow to switch back to an already open session in a
more transparent way. Right now if an user has already one session open, he
goes back to the login screen and enters his credentials a **new ** session is
started. I think that most users would expect to be switched back to their
already running session. Starting a new session is just confusing for them.

