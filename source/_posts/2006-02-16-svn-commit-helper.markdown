---
author: Flavio
date: '2006-02-16 12:51:41'
layout: post
slug: svn-commit-helper
status: publish
title: Svn commit helper
redirect_from: /svn-commit-helper/
comments: true
categories: pyton
---

Svn-commit is a command line utility for making rapid commit with subversion.
Suppose you're working on your local copy of a subversion project. If you
forget to run commands like svn del file or svn add file each time you add or
remove a file, when you'll try to commit your working copy you'll obtain
something like this:

  * _? file_: for each file/directory that isn't under the revision system
  * _! file_: for each file/directory you've removed without the command svn del
svn-commit will prevent these errors because it will tell svn to:

  * add all your unversioned files to the repository
  * delete all the files you've removed from your working directory (be careful !!)

### Requirements:

svn-commit requires:

  * python
  * svn client utilities

### Synopsis:

svn-commit syntax: `svn-commit directory` A simple example: `svn commit `pwd``

### Code

{% codeblock [svn-commit.py] [lang:python ] %}
#! /usr/bin/python

# svn-commit
# Created by Flavio Castelli <flavio.castelli@gmail.com>
# svn-commit follows GPL v2

import os,popen2
from sys import argv

if __name__ == "__main__":

  dir=argv[1];
  pipe = os.popen ('svn status %s'%(dir)) #executes shell command

  result = [x.strip() for x in pipe.readlines() ]
  pipe.close()

  to_add=[]
  to_remove=[]

  for x in result:
    if x.find('?')!=-1:
      to_add+=[x[x.find('/'):len(x)]]
    elif x.find('!')!=-1:
      to_remove+=[x[x.find('/'):len(x)]]

  print "To add\n";
  for x in to_add:
    print ("Command svn add %s"%(x))
    pipe= os.popen ("svn add \"%s\""%(x))
    result = [x.strip() for x in pipe.readlines() ]
    for y in result:
      print y
    pipe.close()
  print "To remove\n";
  for x in to_remove:
    print ("Command svn delete \"%s\""%(x))
    pipe= os.popen ("svn delete \"%s\""%(x))
    result = [x.strip() for x in pipe.readlines() ]
    for y in result:
      print y
    pipe.close()

{% endcodeblock %}
