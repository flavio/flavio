---
author: Flavio
date: '2008-01-10 15:49:17'
layout: post
slug: tyding_up_strigi_analyzers
status: publish
title: Tyding up Strigi analyzers
comments: true
categories: [c++, KDE, strigi]
---

As you may know, KDE4 will use Strigi for meta information extraction instead
of the old [KFilePlugin](http://api.kde.org/3.5-api/kdelibs-
apidocs/kio/kio/html/classKFilePlugin.html) classes.

Since Strigi's analyzer work in a different way, lot of code has to be ported.
Unfortunately, after a good start, some relevant analyzers were still missing.

But in the last weeks Strigi gained support of:

  * wave file
  * avi files
  * txt files
  * dds files
  * rgb files
  * sid files
  * ico files

I've also updated [this](http://wiki.kde.org/tiki-index.php?page=Porting+KFilePlugin+Progress) summary page. As you can see
there's still some work to do, but don't worry... I'll try to do the best ;)
