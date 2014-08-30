---
author: Flavio
date: '2009-11-30 20:20:58'
layout: post
slug: qjson-easier-serialization-of-qobject-instances-to-json
status: publish
title: 'QJson: easier serialization of QObject instances to JSON'
redirect_from: /qjson-easier-serialization-of-qobject-instances-to-json/
comments: true
categories: [qt, qjson]
---

I have just committed into trunk a couple of changes that make easier to
serialize a QObject instance to JSON.

This solution relies on the awesome [Qt's property system](http://doc.trolltech.com/latest/properties.html).

Suppose the declaration of Person class looks like this:

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

  

The following code will serialize an instance of Person to JSON:

{% codeblock [Serialize to JSON] [lang:cpp ] %}
    Person person;
    person.setName("Flavio");
    person.setPhoneNumber(123456);
    person.setGender(Person::Male);
    person.setDob(QDate(1982, 7, 12));  
    Serializer serializer;
    qDebug() << serializer.serialize( &person;);
{% endcodeblock %}

The generated output will be:
{% codeblock [JSON data] [lang:json ] %}
    { "dob" : "1982-07-12", "gender" : 0, "name" : "Flavio", "phoneNumber" : 123456 }
{% endcodeblock %}

I hope you will find this new feature useful. I'm also considering to create a
similar method inside the Parser class.

As usual suggestions are welcome.

