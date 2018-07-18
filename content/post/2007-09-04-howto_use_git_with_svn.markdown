---
author: [Flavio]
date: '2007-09-04 12:22:15'
layout: post
slug: howto_use_git_with_svn
status: publish
title: Howto use Git and svn together
redirect_from: /howto_use_git_with_svn/
categories: [howto, KDE, git, svn]
comments: true
---

In these days I've heard lot of rumors around [Git](http://git.or.cz/). After
reading some manual/tutorial/guide I discovered that it can be really useful,
especially if you spend lot of time coding off-line (that's my situation).

This is a really small howto that describes how to work on a project versioned
with svn (maybe taken from KDE repository ;) ) using git.

### What're the advantages?

Since Git is a distributed revision control system (while svn is a centralized
one) you can perform commits, brances, merges,... on your local working dir
without being connected to internet.

Next time you'll be online, you will be able to _"push"_ your changes back to
the central svn server.

### Steps to follow:

You've to:

  1. install _git_ and _git-svn_
  2. create the working dir: `mkdir strigi`
  3. init your git working dir: `cd strigi && git-svn init https://svn.kde.org/home/kde/trunk/kdesupport/strigi` _git-svn init_ command is followed by the address of the svn repository (in this case we point to strigi's repository)
  4. Find a commit regarding the project (you can get it from [cia version control](http://cia.vc/)). **Warning:** the command _git-log_ will show project's history starting from this revision.
  5. Perform the command `git-svn fetch -rREVISION` Where _REVISION_ is the number obtained before.
  6. Update your working dir: `git-svn rebase`
Now you'll be able to work on your project using git as revision control
system.

To **keep update** your working copy just perform:

`git-svn rebase`

You can **commit your changes ** to the svn server using the command:

`git-svn dcommit`

In this way each commit made with git will be _"transformed"_ into a svn one.

### Solve git-svn rebase problems

While adding new cool features to your program, you may experiment some
problem when synchronizing with the main development tree. In fact you have to
commit all local modifications (using the `git-commit` command) before
invoking `git-svn rebase`.

Sometimes it isn't reasonable since your changes are not yet ready to be
committed (you haven't finished/tested/improved your work). But don't worry,
git has a native solution also for this problem, just follow these steps:

  1. put aside your changes using the command: `git-stash`
  2. update your working copy using: `git-svn rebase` as usual
  3. take back your changes typing: `git-stash apply`
  4. clear _"the stash"_ typing: `git-stash clear`
After the first step all your uncommitted changes will disappear from the
working copy, so you'll be able to perform the rebase command without
problems.

For further informations read `git-stash` man page.

That's all.

A special mention goes to Thiago Macieira for his help.

