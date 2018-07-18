---
author: [Flavio]
date: '2006-02-16 12:42:50'
layout: post
slug: regexp-with-boost
status: publish
title: Regexp with boost
redirect_from: /regexp-with-boost/
comments: true
categories: [c++]
---

How can you add regular expressions to C++?
Here you're three small examples.

## Pattern matching
In this example you’ll find how you can match a regexp in a string.

{% codeblock [pattern matching] [lang:c++ ] %}
// Created by Flavio Castelli <flavio.castelli_AT_gmail.com>
// distrubuted under GPL v2 license

#include <boost/regex.hpp>
#include <string>

int main()
{
  boost::regex pattern ("bg|olug",boost::regex_constants::icase|boost::regex_constants::perl);
  std::string stringa ("Searching for BsLug");

  if (boost::regex_search (stringa, pattern, boost::regex_constants::format_perl))
    printf ("found\n");
  else
    printf("not found\n");
    
  return 0;
}
{% endcodeblock %}

## Substitutions
In this example you’ll find how you can replace a string matching a pattern.

{% codeblock [substitutions] [lang:c++ ] %}
// Created by Flavio Castelli <flavio.castelli@gmail.com>
// distrubuted under GPL v2 license

#include <boost/regex.hpp>
#include <string>

int main()
{
  boost::regex pattern ("b.lug",boost::regex_constants::icase|boost::regex_constants::perl);
  std::string stringa ("Searching for bolug");
  std::string replace ("BgLug");
  std::string newString;

  newString = boost::regex_replace (stringa, pattern, replace);

  printf("The new string is: |%s|\n",newString.c_str());

  return 0;
}
{% endcodeblock %}

## Split
In this example you’ll find how you tokenize a string with a pattern.

{% codeblock [split] [lang:c++ ] %}
// Created by Flavio Castelli <flavio.castelli@gmail.com>
// distrubuted under GPL v2 license

#include <boost/regex.hpp>
#include <string>

int main()
{
  boost::regex pattern ("\\D",boost::regex_constants::icase|boost::regex_constants::perl);
  
  std::string stringa ("26/11/2005 17:30");
  std::string temp;
  
  boost::sregex_token_iterator i(stringa.begin(), stringa.end(), pattern, -1);
  boost::sregex_token_iterator j;

  unsigned int counter = 0;
  
  while(i != j)
  {
    temp = *i;
    printf ("token %i = |%s|\n", ++counter, temp.c_str());
    i++;
  }

  return 0;
}
{% endcodeblock %}

## Requirements
In order to build this examples you’ll need:

  * a c++ compiler (like g++)
  * [boost regexp library](http://www.boost.org/libs/regex/doc/index.html)
