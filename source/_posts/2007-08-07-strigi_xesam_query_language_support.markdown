---
author: Flavio
date: '2007-08-07 14:43:27'
layout: post
slug: strigi_xesam_query_language_support
status: publish
title: Strigi gets XesamQueryLanguage queries support
redirect_from: /strigi_xesam_query_language_support/
comments: true
categories: [c++, strigi, KDE, xexam]
---

Since last Thursday Strigi gained _XesamQueryLanguage_ support. This means
that now is possible to process queries formulated using this syntax.

But why is this important?  If you aren't able to answer the previous question
probably you don't know what is Xesam. Here's a short definition taken from
the [official site](http://www.freedesktop.org/wiki/XesamAbout):  Xesam is an
umbrella project with the purpose of providing unified apis and specs for
desktop search- and metadata services.  Thanks to dbus and Xesam it will be
possible to access the informations indexed by Strigi (and all the desktop
searching programs supporting these technologies) in a standard and easier
way. Isn't it cool?

### Credits

I've to say a big _**"thank you"**_ to [Fabrice Colin](http://developer.berlios.de/blog/authors/6825-Fabrice-Colin) (author of
[pinot](http://pinot.berlios.de/)) because my Xesam code relies upon his work.

### Future tasks

My work isn't yet finished. Xesam defines two kind of queries:

  * Xesam user language queries
  * Xesam query language queries
Fabrice's code for _XesamUserLanguage_ queries uses [Spirit library](http://spirit.sourceforge.net/). Since we don't want to depend
against the boost library, I'll write a new parser for this language.

By now I'm thinking to accomplish this task using flex, but I'm just in a
preliminary state. Suggestions are welcome!

**P.S.** I'm really happy because this is my first post published on [PlanetKDE](http://www.planetkde.org). Hello to everybody!

