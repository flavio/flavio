---
author: [Flavio]
date: '2008-11-04 13:37:07'
layout: post
slug: qjson_qt_json_library
status: publish
title: 'QJson: a Qt-based library for mapping JSON data to QVariant objects'
redirect_from: /qjson_qt_json_library/
wordpress_id: '98'
categories: [qt, json, qjson]
comments: true
---

In order to realize a project of mine I started looking for a Qt library for
mapping JSON data to Qt objects.

I came over a couple of solutions but none of them made me happy. So in the
last weekend I wrote my own library : **QJson** The library is based on Qt
toolkit and converts JSON data to _QVariant_ instances. JSON arrays will be
mapped to _QVariantList_ instances, while JSON's objects will be mapped to
_QVariantMap_. The JSON parser is generated with Bison, while the scanner has
been coded by me.

### Usage

Converting JSON's data to _QVariant_ instance is really simple:

{% codeblock [] [lang:cpp ] %}
// create a JSonDriver instance
JSonDriver driver;
bool ok;
// json is a QString containing the data to convert
QVariant result = driver.parse (json, &ok);
{% endcodeblock %}

Suppose you're going to convert this JSON data:

{% codeblock [JSON data] [lang:json ] %}
{ "encoding" : "UTF-8", "plug-ins" : [ "python", "c++", "ruby" ],
  "indent" : { "length" : 3, "use_space" : true } }
{% endcodeblock %}

The following code would convert the JSON data and parse it:

{% codeblock [] [lang:cpp ] %}
JSonDriver driver;
bool ok;
QVariantMap result = driver.parse (json, &ok).toMap();
if (!ok) {
  qFatal("An error occured during parsing");
  exit (1);
}
qDebug() << "encoding:" << result["encoding"].toString();
qDebug() << "plugins:";
foreach (QVariant plugin, result["plug-ins"].toList()) {
  qDebug() << "\t-" << plugin.toString();
}
QVariantMap nestedMap = result["indent"].toMap();
qDebug() << "length:" << nestedMap["length"].toInt();
qDebug() << "use_space:" << nestedMap["use_space"].toBool();
{% endcodeblock %}

The output would be:

    encoding: "UTF-8" plugins: - "python" - "c++" - "ruby" length: 3 use_space: true

### Requirements

QJson requires:

  * cmake
  * Qt

### Obtain the source

Actually QJson code is hosted on KDE subversion repository. You can download
it using a svn client:

> svn co svn://anonsvn.kde.org/home/kde/trunk/playground/libs/qjson

For more informations visit [QJson site](http://qjson.sourceforge.net)

