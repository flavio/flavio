---
author: Flavio
date: '2007-08-10 15:00:21'
layout: post
slug: xesam-and-bathroom-hacking
status: publish
title: Xesam and bathroom hacking
comments: true
categories: [howto, strigi, xesam, KDE, life]
---

Yesterday morning I was quite arrived at work when Laura (my gf) called me.
Something went wrong in our bathroom and water was everywhere. She closed the
main water tap and I took the first train for home (yes, since I'm an outlier
I take the train two times per day). Once arrived at home I performed some
hacking on the guilty washing machine, checked some pipes and than took the
next train for office.

In the end yesterday I spent approximately **four hours** on the train. During
this elapse of time I started the Xesam User Language parser :)  During the
travels I:

  * refreshed my memories about [Flex](http://www.gnu.org/software/flex/), [Bison](http://www.gnu.org/software/bison/) and language parsers in general
  * wrote XesamUserLanguage's [BNF grammar](http://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form)
  * wrote Flex scanner
  * started Bison parser
  
Now, after fixing some build errors, I'll start writing Bison's grammar rules.
These rules will translate Xesam user language queries into Strigi::Query
objects.

I hope it will work (both bathroom and Xesam parser ;) )

