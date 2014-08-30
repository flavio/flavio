---
author: Flavio
date: '2006-05-12 18:09:06'
layout: post
slug: ibook-x-keyboard-mapping
status: publish
title: iBook X keyboard mapping
redirect_from: /ibook-x-keyboard-mapping/
categories: howto
comments: true
---

How to have a comfortable keyboard layout under X. A guide destinated to
italian iBook linux users

### Target

have the following keymap: `apple keyboard = alt gr`

### Instructions

You've to edit your `/etc/X11/xkb/keycodes/xfree86` file. Remember to make a
backup copy of this file before editing. These're the easy steps:

  * under X run a program like xev to find the exact keycode of your apple key
  * change the  code with the new one
  * comment all the old command referring the new keycode 
  
Restart X and keep your finger crossed ;)
