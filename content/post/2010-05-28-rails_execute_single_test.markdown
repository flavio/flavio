---
author: [Flavio]
date: '2010-05-28 15:57:57'
layout: post
slug: rails_execute_single_test
status: publish
title: How to run a single rails unit test
redirect_from: /rails_execute_single_test/
comments: true
categories: [ruby, ruby on rails, TDD, howto]
---

This post explains how to execute a single unit test (or even a single test
method) instead of running the complete unit test suite.

In order to run the unit tests of your rails application, basically you have
these official possibilities:

  * `rake test`: runs all unit, functional and integration tests.
  * `rake test:units`: runs all the unit tests.
  * `rake test:functionals`: runs all the functional tests.
  * `rake test:integration`: runs all the integration tests.
Each one of these commands requires some time and they are not the best
solution while developing a new feature or fixing a bug. In this circumstance
we just want to have a quick feedback from the unit test of the code we are
editing.

Waiting for all the unit/functional tests to complete decreases our
productivity, what we need is to execute just a single unit test. Fortunately
there are different solutions for this problem, let's go through them.

## The easy approach: use your favorite IDE

Most of the IDE supporting ruby allow you to run a single unit test. If you
are using Netbeans running a single unit test is really easy:

  * make sure the editor if showing the file you want to test or the file containing its unit tests
  * Hit _Ctrl+Shift+F6_ or click on the following menu entry: _Debug->Debug Test File_
Two new windows will be opened: one will contain the output produced by your
unit test, the other one will show the results of the unit test.

As you will notice the summary window contains also some useful information
like the:

  * hyper links to the exact location of the code that produced the error/failure.
  * execution time required by each one of the test methods.
As you will experience it will be like "compiling" your ruby code.

## From the console

If you are not using Netbeans you can always rely on some command line tools.

### No additional tools

These "tricks" don't require additional gems, hence they will work out of the
box.

The first solution is to call this rake task:

    
    rake test TEST=path_to_test_file

So the final command should look like

    
    rake test TEST=test/unit/invitation_test.rb

Unfortunately on my machine this command repeats the same test three times, I
hope you won't have the same weird behavior also on your systems...

Alternatively you can use the following command:

    
    ruby -I"lib:test" path_to_test_file"

It's even possible to **call a specific test method of your testcase**:

    
    ruby -I"lib:test" path_to_test_file -n name_of_the_method"

So calling:

    
    ruby -I"lib:test" test/unit/invitation_test.rb - test_should_create_invitation

will execute **only** _InvitationTest::test_should_create_invitation_.

It's also possible to **execute only the test methods matching a regular
expression**. Look at this example:

    
    ruby -I"lib:test" test/unit/invitation_test.rb -n /.*between.*/

This command will execute only the test methods matching the _/.*between.*/_
regexp.

### Using the single_test gem

If you want to avoid the awful syntax showed in the previous paragraph there's
a gem that can help you, it's called
[single_test](http://github.com/grosser/single_test).

The github page contains a nice documentation, but let's go through the most
common use cases.

You can install the gem as a rails plugin:

    
    script/plugin install git://github.com/grosser/single_test.git

single_test will add new rake tasks to your rails project, but **won't**
override the original ones.

Suppose we want to execute the unit test of _user.rb_, just type the following
command:

    
    rake test:user

If you want to execute the functional test of _User_ just call:

    
    rake test:user_c

Appending _"_c"_ to the class name will automatically execute its functional
test (if it exists).

It's still possible to **execute a specif test method**:

    
    rake test:user_c:_test_name_

So calling:

    
    rake test:user_c:test_update_user

Will execute the _test_update_user_ method written inside of
_test/functional/user_controller_test.rb_.

It's still possible to use regexp:

    
    rake test:invitation:.*between.*

This syntax is equivalent to `ruby -I"lib:test" test/unit/invitation_test.rb
-n /.*between.*/`.

## Possible issues

When a single unit test is run all the usual database initialization tasks are
not performed. If your code is relying on newly created migrations you will
surely have lots of errors. This is happening because the new migrations have
not been applied to the test database.

In order to fix these errors just execute:

    
    rake db:test:prepare

before running your unit test.

