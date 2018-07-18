---
author: [Flavio]
date: '2009-07-28 09:47:05'
layout: post
slug: kaveau-easy-integrated-backup-solution-kde
status: publish
title: 'kaveau: easy and integrated backups solution for KDE'
redirect_from: /kaveau-easy-integrated-backup-solution-kde/
categories: [KDE]
comments: true
---

During the last week I had the possibility to work on anything I wanted,
Novell's hackweek is so cool :)

I decided to dedicate myself to an idea that has been obsessing me since a
long time. Last December my brand new hard disk suddenly died, making
impossible to recover anything. Fortunately I had just synchronized the most
important documents between my workstation and my laptop, so I didn't lose
anything important. This incident make me realize that I should perform
backups regularly and I immediately started looking for a good solution.

Personally I think that doing backups is pretty boring so I wanted something
damned easy to setup and use. Something that once configured runs in the
background and does the dirty job without bothering you. Let's face the truth:
I wanted Apple's Time Machine for KDE.

After some searches I realized that nothing was fitting my requirements and I
decided to create something new: _kaveau_.

### What is kaveau?

Kaveau is a backup program that makes backup easy for the average user.

As you will see while coding/planning kaveau I made some assumptions and so
only few things are configurable. I really think that sometimes [_"less is
more"_](http://en.wikipedia.org/wiki/Minimalism).

### What does kaveau?

Current features:

  * it performs backups to an external storage device: I don't think it will ever store backup data on a remote host. If you want to do that just use some good project like [Backup pc](http://backuppc.sourceforge.net/).
  * it backs up the complete home directory of the user: storage is cheap and average users (like me) keep everything inside their home directory ( but it's possible to exclude some directories from the backup).
  * it performs incremental backups.
  * the backup data are neither compressed nor stored in fancy formats: in this way you can plug your external disk into another machine and access your data without additional work.
  * backups are performed automatically every hour (of course only if your external disk is plugged).
  * notification messages are shown if your backup is older that a week.
More enhancements are coming...

### What technologies does it use?

Backups are performed using [rdiff-backup](http://rdiff-backup.nongnu.org/)
because it's damned easy to use, well tested (it's used also in production
environments) and packaged by all distributions.

The awesome [solid](http://solid.kde.org/) library is used for interacting
with the external disks is super easy.

### Status of kaveau

I have been working on kaveau just for five days, so **there's still a lot of
work to do**.

A screenshot tour will give you the right idea of its status.

#### First run - external storage device attached
![sf1](/images/kaveau/sf1.png)

####Backup wizard - page 1
![sf3](/images/kaveau/sf3.png)

####Backup wizard - page 2

![sf4](/images/kaveau/sf4.png)

####Backup wizard - final page

![sf5](/images/kaveau/sf5.png)

####Backup operation in progress

![sf6](/images/kaveau/sf6.png)

####Backup completed

![sf7](/images/kaveau/sf7.png)

Right now the code is available on
[this](http://github.com/flavio/kaveau/tree/master) git repository but I don't
recommend you to try it (unless you want to find and fix bugs ;) ).

I would really appreciate:

  * feedback about the user interface (right now it looks too much like Time Machine).
  * icons: it would be great to have a desktop icon and some system tray icons (contact me for more details).
  * new code, bug fixes, code reviews, hints,...

