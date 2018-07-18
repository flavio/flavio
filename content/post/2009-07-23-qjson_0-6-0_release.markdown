---
author: [Flavio]
date: '2009-07-23 11:51:30'
layout: post
slug: qjson_0-6-0_release
status: publish
title: New QJson release
redirect_from: /qjson_0-6-0_release/
comments: true
categories: [KDE, qt, qjson]
---

Gran Canaria Desktop Summit has been great and really productive. I had the
pleasure to meet people interested in QJson, chat with them but also hack with
them.

In fact we hacked a lot, doing lots of changes to QJson:

  * the API has been cleaned, now it can be considered stable
  * unicode support has been completely rewritten
  * it's now possible to convert QVariant objects into JSON ones
  
So it's with a great pleasure that I announce the release of QJson 0.6.0.

Beware, since the API has been changed your application will probably break.
I'm really sorry about that, but I guarantee it won't happen in the future (as
I said both API and ABI interfaces can now be considered stable).

[QJson web site](http://qjson.sourceforge.net) has been updated, reflecting
all the changes made to the library. openSUSE packages has been moved from my
home repo to [KDE:Qt](http://download.opensuse.org/repositories/KDE:/Qt/) one.

One last note, if you have problems with QJson please contact me using the
qjson-devel mailing list. You can subscribe
[here](https://sourceforge.net/mailarchive/forum.php?forum_name=qjson-devel).

