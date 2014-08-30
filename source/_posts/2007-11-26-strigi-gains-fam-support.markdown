---
author: Flavio
date: '2007-11-26 12:33:52'
layout: post
slug: strigi-gains-fam-support
status: publish
title: Strigi gains FAM support
redirect_from: /strigi-gains-fam-support/
comments: true
categories: [c++, KDE, strigi]
---

Last Monday I submitted lot of changes into Strigi's trunk. I've heavily
refactored some classes in order to obtain a more flexible file system
notification infrastructure. Thanks to this work now it will be easier to add
support for new file system notification facilities.  For example, in order to
add _File Alteration Monitor_ (aka _FAM_) support I had to write only 576 loc
(including license and documentation stuff).

So, by now, Strigi supports the following file system monitoring facilities:

  * polling: used when nothing else is available
  * inotify: available only on linux platform with kernel >= 2.6.13
  * FAM: available on most of the *nix systems, I recommend to use Gamin instead of FAM (most linux distributions already use it by default)

