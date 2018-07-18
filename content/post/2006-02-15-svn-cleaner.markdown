---
author: [Flavio]
date: '2006-02-15 00:17:23'
layout: post
slug: svn-cleaner
status: publish
title: Svn cleaner
redirect_from: /svn-cleaner/
comments: true
categories: [bash]
---

This simple program removes recursively all .svn directories.

**Requirements:** In order to run _remove-svn_ requires python.

**Synopsis:** _remove-svn_ syntax is: `remove-svn dir` in this way remove-svn will recursively remove all _.svn_ directories found under dir.  
An example: `remove-svn `pwd``

**UPDATE** A faster way for removing _.svn_ file through this simple bash command: `find ./ -name *svn* | xargs rm -rf`

The old script has been removed.

