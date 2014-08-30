---
author: Flavio
date: '2009-12-04 01:07:48'
layout: post
slug: qjson-qobject-serialization-deserialization
status: publish
title: 'QJson: from QObject to JSON and vice-versa'
redirect_from: /qjson-qobject-serialization-deserialization/
comments: true
categories: [qt, qjson]
---

Some days ago I introduced the possibility to serialize a QObject instance to
JSON. Today I'm going to show you the opposite operation: initializing a
QObject using a JSON object.

I refactored a bit my latest changes: I created a new class called
QObjectHelper that provides the methods required to convert a QObject instance
to a QVariantMap and vice-versa.

This class can be used in conjunction with the Serializer and Parser classes
to serialize and deserialize QObject instances to and from JSON.

Let me show a quick example, suppose the declaration of Person class looks
like this:

{% codeblock [class definition] [lang:cpp ] %}
class Person : public QObject
{
  Q_OBJECT
  Q_PROPERTY(QString name READ name WRITE setName)
  Q_PROPERTY(int phoneNumber READ phoneNumber WRITE setPhoneNumber)
  Q_PROPERTY(Gender gender READ gender WRITE setGender)
  Q_PROPERTY(QDate dob READ dob WRITE setDob)
  Q_ENUMS(Gender)

  public:
    Person(QObject* parent = 0);
    ~Person();
    QString name() const;
    void setName(const QString& name);
    int phoneNumber() const;
    void setPhoneNumber(const int  phoneNumber);
    enum Gender {Male, Female};
    void setGender(Gender gender);
    Gender gender() const;
    QDate dob() const;
    void setDob(const QDate& dob);
  private:
    QString m_name;
    int m_phoneNumber;
    Gender m_gender;
    QDate m_dob;
};
{% endcodeblock %}

### From QObject to JSON

The following code will serialize an instance of Person to JSON :

{% codeblock [From QObject to JSON] [lang:cpp ] %}
Person person;
person.setName("Flavio");
person.setPhoneNumber(123456);
person.setGender(Person::Male);
person.setDob(QDate(1982, 7, 12));
QVariantMap variant = QObjectHelper::qobject2qvariant(&person;);
Serializer serializer;
qDebug() << serializer.serialize( variant);
{% endcodeblock %}

The generated output will be:

{% codeblock [JSON data] [lang:json ] %}
{ "dob" : "1982-07-12", "gender" : 0, "name" : "Flavio", "phoneNumber" : 123456 }
{% endcodeblock %}

### From JSON to QObject

Suppose you have the following JSON data stored into a QString:

{% codeblock [JSON data] [lang:json ] %}
{ "dob" : "1982-07-12", "gender" : 0, "name" : "Flavio", "phoneNumber" : 123456 }
{% endcodeblock %}

The following code will initialize an already allocated instance of Person
using the JSON values:

    
{% codeblock [From JSON to QObject] [lang:cpp ] %}
Parser parser;
QVariant variant = parser.parse(json);  
Person person;
QObjectHelper::qvariant2qobject(variant.toMap(), &person;);
{% endcodeblock %}

### A new release

These changes have been included inside the new release of QJson: 0.7.0.

Packages for openSUSE are building right now.

