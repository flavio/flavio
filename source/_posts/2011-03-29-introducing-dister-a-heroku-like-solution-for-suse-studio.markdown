---
author: Flavio
date: '2011-03-29 15:39:21'
layout: post
slug: introducing-dister-a-heroku-like-solution-for-suse-studio
permalin: /:title.html
status: publish
title: Introducing dister, a Heroku like solution for SUSE Studio
redirect_from: /introducing-dister-a-heroku-like-solution-for-suse-studio/
comments: true
categories: [ruby, ruby on rails, suse studio]
---

[SUSE Studio](http://susestudio.com) is  an awesome tool, with a couple of
clicks you can create an openSUSE/SUSE based system and deploy to your hard
drive, an usb flash,  a live dvd, a VMware/VirtualBox/Xen server and even
Amazon EC2 cloud.

Suppose you want to create a tailored SUSE Studio appliance to run a Ruby on
Rails app, this is a list of things you have to take care of:

  * install all the gems required by the app (this can be a long list).
  * install and configure the database used by the app.
  * install and configure a webserver.
  * ensure all the required services are started at boot time.
You can save some time by cloning [this](http://susegallery.com/a/CZ0T0D
/rails-in-a-box) appliance shared on [SUSE Gallery](http://susegallery.com/),
but this is still going to be boring.

## Dister to the rescue!

Dister is a command line tool similar to the one used by
[Heroku](http://heroku.com) (one of the coolest ways to run your Rails app
into the cloud). Within a few steps you will be able to create a SUSE Studio
appliance running your rails application, download it and deploy wherever you
want.

Dister is named after SUSE Studio robot. It has been created by  [Dominik
Mayer](http://opensuse.blip.tv/file/4678185/) and me during the latest
[hackweek](http://en.opensuse.org/Portal:Hackweek).

## How it works

We are going to create a SUSE Studio appliance running a rails application
called _"SUSE history"_. The app uses [bundler](http://gembundler.com/) to
handle its dependencies. This is its Gemfile file:

    
    
    ﻿﻿source 'http://rubygems.org'
    gem 'rails', '3.0.5'
    gem 'pg'
    gem "flutie", "~> 1.1"
    

  
As you can see the app uses rails3, the
[flutie](https://github.com/thoughtbot/flutie) gem and PostgreSQL as database.

### Appliance creation

Move into the suse_history directory and execute the following command:

    
    
    dister create suse_history
    

{% img /images/introducing_dister/create.png %}

As you can see dister has
already done a couple of things for you:

  * created an appliance using latest version of openSUSE supported by SUSE Studio (you can use a different base system of course)
  * added the _devel:language:ruby:extensions_ repository to the appliance: this repo contains tons of ruby packages (like _mod_passenger_)
  * installed a couple of things: 
    * _devel_C_C++_ pattern: this will allow you to build native gems. 
    * _devel_ruby_ pattern: provides ruby, rubygems and a couple of development packages needed to build native gems.
    * _rubygem-bundler_: bundler is required by dister in order to handle the dependencies of your rails application.
    * _rubygem-passenger-apache2_: dister uses [passenger](http://www.modrails.com/) and apache2 to deploy your rails application.
    * _postgresql-server_: dister noticed suse_history uses PostgreSQL as database, hence it automatically installs it.
    * _rubygem-pg_: dister noticed suse_history uses PostgreSQL as database, hence it automatically installs the ruby's library forPostgreSQL.
  * uploaded a custom build script which ensures: 
    * mod_passenger module is loaded by Apache
    * both Apache and PostgreSQL are always started at boot time.
    * all dependencies are installed: this is done only during the first boot using bundler.
    * the database user required by your rails app is created. This is done only during the first boot using some SQL code.
    * the database used by the appliance is properly initialized (aka _rails db:create db:migrate_). This is done only during the first boot.
  

### Upload your code

It's time to upload suse_history code. This is done using the following
command:

    
    
    dister push
    

{% img /images/introducing_dister/push.png %}

As you can see dister packaged the
application source code and all its dependencies into a single archive. Then
uploaded the archive to SUSE Studio as an overlay file. Dister uploaded also
the configuration files required by Apache and by PostgreSQL setup.

### Build your appliance

It's build time!

    
    
    dister build
    

{% img /images/introducing_dister/build.png %}
The appliance has automatically being built
using the _raw disk_. You can use different formats of course.

### Testdrive

Testdrive is one of the coolest features of SUSE Studio. Unfortunately dister
doesn't support it yet. Just visit your appliance page and start testdrive
from your browser. Just enable testdrive networking and connect to your
appliance:

{% img /images/introducing_dister/testdrive.png %}

### Download

Your appliance is working flawlessly. Download it and deploy it wherever you
want.

    
    
    dister download
    

  

## Current status

As you can see dister handles pretty fine a simple Rails application, but
there's still room for improvements.

Here's a small list of the things on my TODO list:

  * The dependency management should install gems using rpm packages. This would make the installation of native gems easier, right now the user has to manually add all the development libraries required by the gem. Moreover it would reduce the size of the overlay files uploaded to SUSE Studio.
  * SUSE Studio Testdrive should be supported.
  * It should be possible to deploy the SUSE Studio directly to EC2.
  * Fix bugs!
  
Bugs and enhancements are going to be tracked
[here](https://github.com/flavio/dister/issues).

## Contribute

Dister code can be found here on [github](https://github.com/flavio/dister),
fork it and start contributing.

If you are a student you can work on dister during the next [Google Summer of 
code](http://en.opensuse.org/openSUSE:GSOC_2011_Ideas#Heroku_like_solution_for
_SUSE_Studio), apply now!

