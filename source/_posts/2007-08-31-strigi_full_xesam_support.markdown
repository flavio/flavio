---
author: Flavio
date: '2007-08-31 17:30:52'
layout: post
slug: strigi_full_xesam_support
status: publish
title: Strigi gains full Xesam queries support
redirect_from: /strigi_full_xesam_support/
comments: true
categories: [c++, KDE, strigi, xesam]
---

As I said in
[this](http://www.flavio.castelli.name/strigi_xesam_query_language_support)
previous post, Strigi's Xesam support was half-done since [XesamUserSearchLanguage](http://wiki.freedesktop.org/wiki/XesamUserSearchLanguage) wasn’t yet
handled. Well, this is no longer true... ;) In these weeks I’ve been working
on _XesamUserSearchLanguage_ support. Ehm... to be honest, I’ve been
_**fighting**_ with [Bison](http://www.gnu.org/software/bison/).

But in the end I tamed the beast and now
[Xesam](http://wiki.freedesktop.org/wiki/XesamAbout) support in Strigi is
full.

IMHO _XesamUserSearchLanguage_ can be considered more important than
[XesamQueryLanguage](http://wiki.freedesktop.org/wiki/XesamQueryLanguage)
since common users will write queries in this way.

As reported on the project page:
{% blockquote %}
It is _[XesamUserSearchLanguage]_ designed as an extended synthesis of
Apple's spotlight and Google's search languages.
{% endblockquote %}

These are some possible queries (examples taken from freedesktop site):

  * `_type:music hendrix_` will return all music items related to hendrix
  * `_type:image size>=1mb tag:flower africa_` will return all pictures displaying a flower greater than 1 Mb and related with africa
  

### Technical aspects

The Xesam's UserSearchLanguage query --> Strigi::Query object conversion is
made using a hand-written scanner and a C++ parser created by Bison.

You don't have to worry if you don't have Bison installed on your system since
all parser generated code is already put into svn. In these days, as soon as
I'll have some spare time (when?!), I'll write another post about open-source
scanner and parser generators.

By now I would like to thank [Andreas Pakulat](http://apaku.wordpress.com)
(developer of [KDevelop](http://www.kdevelop.org/)) for his help with parser
generators.

