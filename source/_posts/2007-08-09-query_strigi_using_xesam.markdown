---
author: Flavio
date: '2007-08-09 13:10:30'
layout: post
slug: query_strigi_using_xesam
status: publish
title: How to have some fun with Strigi and Xesam queries
comments: true
categories: [howto, strigi, xesam, KDE]
---

Last day just after I hit the _"submit"_ button a doubt came into my mind:
"did I say everything ?" Well, the answer is **"No!"** In fact I forgot to
tell you one of the most important things: _how to have some fun with Strigi
and Xesam!_ Actually the only way to perform _XesamQueryLanguage_ queries with
Strigi is through the _strigicmd_ program.

_Strigicmd_ is a command-line tool shipped with Strigi. It can perform
different actions like:

  * create Strigi indexes
  * remove items from index
  * list all files contained into an index
  * retrieve informations associated to an indexed file
  * update the contents of your index
  * query the index
  * **perform a query using XesamQueryLanguage**

So, if you want to try the new Xesam support you've just to use _strigicmd_
with the _xesamquery_ option. The command syntax is: `strigicmd xesamquery -t
backend -d indexdir [-u xesam_user_language_file] [-q
xesam_query_language_file]` As you can expect you've to save your Xesam query
to file and point _strigicmd_ to it.

This is a really small step-by-step guide:

  * Create a new Strigi index (in this case I'll index all irc logs): `strigicmd create -t clucene -d temp/ logs/`
  * Create a simple file containing your Xesam query. You can find some example query on [Xesam site](http://www.freedesktop.org/wiki/XesamQueryLanguage) or inside strigi tarball (complete path: _strigi/src/streamanalyzer/xesam/testqueries/_). This is a stupid and easy query: 

    {% codeblock [query] [lang:xml ] %}
    <request>
    <query>
    <or>
    <equals>
    <string casesensitive="true">Oever</string>
    </equals>
    <contains>
    <string casesensitive="false">jos</string>
    </contains>
    </or>
    </query>
    </request>
    {% endcodeblock %}

  * Perform the search, just type: 
    
    strigicmd xesamquery -t clucene -d temp/ -q ~/irc_oever.xml

  * **Enjoy the search results ;)**

Remember that _XesamUserLanguage_ query language isn't yet supported.

