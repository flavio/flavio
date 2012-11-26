---
author: Flavio
date: '2008-10-03 11:28:28'
layout: post
slug: ruby-downloader-for-jamendo
status: publish
title: Ruby downloader for Jamendo
categories: [ruby, jamendo]
comments: true
---

Also this year I'll attend the Linux day (a day dedicated to Gnu/Linux  and
FLOSS that occurs every year in Italy) organized by my [LUG](http://bglug.it).
Guess what I'll be talking about... ;)

While organizing the event somebody proposed to setup a local server with some
music released under CC license. He suggested to download some albums from
[Jamendo](http://www.jamendo.com) (due to network issues we won't be able to
provide direct access to the website).

Since nobody wanted to download the albums by hand, last night I wrote a small
ruby program that does the dirty job.

### Requirements:

Ruby and json gem have to be installed on you machine.

### Usage:

Help:[](http://www.flavio.castelli.name/wp-
content/uploads/2008/10/jamendo_downloader.rb)

   ./jamendo_downloader.rb --help

Download the top 10 rock albums:

  ./jamendo_downloader.rb -g rock -t 10

### Have fun

I think there's nothing more to say... enjoy it!

{% gist 2437530 jamendo_downloader.rb %}
