---
author: Flavio
date: '2006-05-05 12:07:21'
layout: post
slug: ibook-mouse-emulation
status: publish
title: iBook mouse emulation
comments: true
categories: howto
---

In this really small guide you'll discover howto enable mouse button emulation
on a iBook G4 running linux

### Target:

I just wanted to enable mouse button emulation on my iBook using the following
shortcuts:

    
    fn + ctrl = middle mouse button
    fn + alt = left mouse button

### Requisites:

You've to enable `CONFIG_MAC_EMUMOUSEBTN` in your kernel.

### Commands:

In order to enable this shortcut at every boot add the following lines to your
`/etc/sysctl.conf`:

    
    dev.mac_hid.mouse_button_emulation = 1
    dev.mac_hid.mouse_button2_keycode = 97
    dev.mac_hid.mouse_button3_keycode = 100

Simple, isn't it?

