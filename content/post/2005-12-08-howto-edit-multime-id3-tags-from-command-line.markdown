---
author: [Flavio]
date: '2005-12-08 18:45:39'
layout: post
slug: howto-edit-multime-id3-tags-from-command-line
status: publish
title: Howto edit multime id3 tags from command line
redirect_from: /howto-edit-multime-id3-tags-from-command-line/
categories: [bash]
comments: true
---

### Goal

id3medit is a simple script for tagging all mp3/ogg files present in a
directory.

### Requirements:

id3medit relies on id3v2, a command-line tool for editing id3v2 tags file
names must be in format: _'## - trackname.ext'_. Where _##_ is track's number,
and _ext_ is file's extension (mp3 or ogg in case insensitive format)

### Synopsis:

id3medit syntax is: `id3medit artist album year(*) genre(*)` Where _*_ denotes
optional arguments You can obtain genre identification number in this way:
`id3v2 -L | grep -i genre`

### Example

    id3v2 -L | grep -i rock
     
       1: Classic Rock
      17: Rock
      40: Alt. Rock
      47: Instrum. Rock
      56: Southern Rock
      78: Rock & Roll
      79: Hard Rock
      81: Folk/Rock
      91: Gothic Rock
      92: Progress. Rock
      93: Psychadel. Rock
      94: Symphonic Rock 
      95: Slow Rock
     121: Punk Rock
     141: Christian Rock
    
    

  

### Code

{% gist 2469919 %}
