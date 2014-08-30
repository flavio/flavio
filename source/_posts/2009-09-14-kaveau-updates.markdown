---
author: Flavio
date: '2009-09-14 23:38:54'
layout: post
slug: kaveau-updates
status: publish
title: kaveau updates
redirect_from: /kaveau-updates/
comments: true
categories: KDE
---

Some weeks have passed since the announcement of kaveau. I'm really proud and
happy about this project because I received a lot of positive feedback
messages and it has been chosen as one of the best Hackweek's projects.

In the meantime I kept working on kaveau, so let me show you what has changed:

  * [rdiff-backup](http://rdiff-backup.nongnu.org/) has been replaced by [rsync](http://www.samba.org/rsync/).
  * the setup wizard has been improved according to the feedback messages I received.
  * old backups are now automatically removed.
  * the code has been refactored a lot.

### The switch to rsync

Previously kaveau used rdiff-backup as backup back-end. rdiff-backup is a
great program but unfortunately it relies on the outdated
[librsync](http://librsync.sourceforge.net/) library. The latest release of
librsync is dated 2004. It has a couple of serious bugs still open and, while
rsync has reached version three, this library is still stuck at version one.

These are the reasons of the switch from rdiff-backup to rsync. This choice
breaks the compatibility with the previous backups but it introduces a lot of
advantages. One of the most important improvements brought by the adoption of
rsync is an easier restore procedure: now **all** the backups can be accessed
using a standard file manager, while previously rdiff-backup was needed to
access the old backups.

#### Backup directory structure

On the backup device everything is saved under the
_kaveau_/_hostname_/_username_ path.

The directory will have a similar structure:

    
    drwxr-xr-x 3 flavio users 4096 2009-09-12 18:50 2009-09-12T18:50:19
    drwxr-xr-x 3 flavio users 4096 2009-09-14 23:07 2009-09-14T23:07:46
    drwxr-xr-x 3 flavio users 4096 2009-09-14 23:30 2009-09-14T23:30:36
    lrwxrwxrwx 1 flavio users   19 2009-09-14 23:30 current -> 2009-09-14T23:30:36

As you can see there's one directory per backup, plus a symlink called
_current_ pointing to the latest backup.

### Old backup deletion

Nowadays big external storage devices are pretty cheap, but it's always good
to save some disk space. Now kaveau keeps:

  * hourly backups for the past 24 hours.
  * daily backups for the past month.
  * weekly backups until the external disk is full.
Thanks to hard links' magic, old backups can be deleted without causing
damages to the other ones.

### Plans for the future

Before starting to work on the restore user interface I will spend some time
figuring out how to add support for network devices.

A lot of users requested this feature, hence I want to make them happy :) .

I'm planning to use [avahi](http://avahi.org/) to discover network shares
(nfs, samba) or network machines running ssh and use them as backup devices.
Honestly, I want to achieve something similar to
[Apple's time capsule](http://www.apple.com/timecapsule/).

As usual, feedback messages are really appreciated.

