---
author: [Flavio]
date: '2008-01-18 15:31:23'
layout: post
slug: my_current_tasks
status: publish
title: My current tasks
redirect_from: /my_current_tasks/
comments: true
categories: [KDE, strigi, xesam]
---

Usually I write blog posts announcing **what I have done**, but this time
[it](http://pollycoke.net/2008/01/14/strigi-ha-un-piano-di-sviluppo-
finalmente/) [is](http://www.kdedevelopers.org/node/3204)
[useless](http://strigi.sourceforge.net/?q=strigi_metting_summary). So I'm
going to blog about **what I'm going to do**.  After latest Strigi irc
meeting, I came out with this task:

> KDE integration: Flavio will coordinate the definition of interfaces over
which KDE will handle searching and metadata. He can ask Aaron, Evgeny and Jos
for help with the interface design. The interface will cover:

>

>   * Querying via Xesam

>   * Configuration of the Strigi daemon

>   * Indexing and deindexing of data by passing it to the daemon (allowing
for indexing for more than just files)

>   * Controlling the daemon (starting, stopping, pausing)

Once this interface will be ready, it will be easy to integrate Strigi
functionalities inside KDE programs. This mean (just reporting the most
relevant cases) that it will be possible to create a Strigi krunner, have
metadata extraction inside Dolphin and Konqueror, interact with Akonadi...

Talking about Xesam, right in these days I got a mail from [Fabrice Colin](http://developer.berlios.de/blog/authors/6825-Fabrice-Colin), author of
[Pinot](http://pinot.berlios.de/). Recently Fabrice made some improvements on
Pinot's _XesamQueryLanguage_ parser (which is also [used by Strigi](http://www.flavio.castelli.name/strigi_xesam_query_language_support)).
We're now figuring out how to share our code in a more convenient way. Maybe
we'll use [svn external](http://svnbook.red-bean.com/nightly/en/svn.advanced.externals.html)...

