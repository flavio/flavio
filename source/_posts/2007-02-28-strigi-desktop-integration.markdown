---
author: Flavio
date: '2007-02-28 19:26:43'
layout: post
slug: strigi-desktop-integration
status: publish
title: Strigi Desktop integration
categories: [KDE, strigi]
comments: true
---

### Short abstract:

Strigi desktop integration: how to access Strigi features.

Strigi, the fastest and smallest desktop search engine, provides fast searches
and good metadata extraction capabilities. It will be used by the next KDE,
but it can be easily integrated in other programs.

### Long abstract:

Modern humans are using more and more data every day. Keeping data organized
is becoming insufficient, so finding and filtering documents have become key
tasks in modern operating systems. The traditional unix tools find, grep and
locate do no longer suffice for a number of reasons: The amounts of data have
grown more than data access speeds, many document formats are too complex to
handle by simple tools and relations between different types of data are
becoming more important for a convenient user experience. Strigi introduces a
new way of looking at metadata and file formats that enables the creation of
very efficient tools for improving the way users handle their data. It does so
by using simple C++ code with very few dependencies.

Strigi's features can be easily integrated by other programs using two
different interfaces: socket and DBus. The last one can be considered the best
choice since there are lot of DBus bindings that make possible to interact
with Strigi using different programming languages. DBus inter-process
communication system will be used by KDE4 and is actually used by Gnome.
Moreover, DBus bindings exists for lot of programming languages. Thanks to
this Strigi can be easily integrated in different window manager and inside
programs written in languages different from C++.

KDE developers can also take advantage from Strigi's JStreams: a set of
classes for fast access to archive files. In fact Strigi defines a dedicated
KIO Slave for JStreams, this can be easily used by other programs, making
archives access faster.

This presentation will show how the current Strigi clients interact with the
daemon, how to communicate with Strigi and, most important of all, how other
projects can benefit too accessing Strigi.

<div style="width:425px" id="__ss_12640141"><strong style="display:block;margin:12px 0 4px"><a href="http://www.slideshare.net/fcastelli/strigi-desktopintegration" title="Strigi desktop-integration">Strigi desktop-integration</a></strong><object id="__sse12640141" width="425" height="355"><param name="movie" value="http://static.slidesharecdn.com/swf/ssplayer2.swf?doc=strigi-desktop-integration-120422101942-phpapp01&stripped_title=strigi-desktopintegration&userName=fcastelli" /><param name="allowFullScreen" value="true"/><param name="allowScriptAccess" value="always"/><param name="wmode" value="transparent"/><embed name="__sse12640141" src="http://static.slidesharecdn.com/swf/ssplayer2.swf?doc=strigi-desktop-integration-120422101942-phpapp01&stripped_title=strigi-desktopintegration&userName=fcastelli" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" wmode="transparent" width="425" height="355"></embed></object><div style="padding:5px 0 12px">View more <a href="http://www.slideshare.net/">presentations</a> from <a href="http://www.slideshare.net/fcastelli">Flavio Castelli</a>.</div></div>
