---
author: Flavio
date: '2007-07-10 22:50:34'
layout: post
slug: lots-of-improvements-into-strigi-inotify-support
status: publish
title: Lots of improvements into Strigi inotify support
categories: [c++, strigi, KDE]
comments: true
---

Some days ago I committed lots of changes regarding inotify support. Goodies
and improvements have been introduced...  I had this code laying on my laptop
for several weeks because I wasn't able to fully test it. An annoying bug
afflicting the _update index_ operations was blocking me. But some days ago
[Jos](http://www.vandenoever.info) fixed it, so I didn't have any excuse :)

These are the major changes introduced:

  1. event caching: using a small cache it's now possible to _simplify multiple events_ and _prevent high cpu usage_ when lot of changes occur on the same files
  2. interruption handling during the re-index operations: changing the directories to watch during an indexing operation will break the previous job and start a new one
  3. other small changes for improving cpu utilization
  
Actually I'm very happy of the first point, but I think the second one can
still be improved...

