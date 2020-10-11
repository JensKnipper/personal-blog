---
layout: post
title: Generating Enums in Java with JCodeModel
author: jens_knipper
date: '2020-10-11 01:00:00'
description: This article will show you how to generate your own enum classes with jCodeModel. A framework to generate java classes.
categories: Java, jCodeModel
---
I have recently been in the need to generate my own Java classes from csv input.
Obviously JCodeModel is your first choice, but it lacks some examples. 
There is a nice [article by Kevin Sookocheff](https://sookocheff.com/post/java/generating-java-with-jcodemodel/) which covers the basic stuff, but it is missing some more in depth examples.
In case you are in need to generate Enums - like me - the next following lines might be interesting for you.

## Getting started

To get started you have to create an instance of JCodeModel. You will need this object to generate your classes.

{% highlight java %}
    JCodeModel codeModel = new JCodeModel();
{% endhighlight %}

## Creating a class

To create a class you need a package where the class is located. Package names can be specified as usual. 
JCodeModel will automcatically create the corresponding structure during code generation.

{% highlight java %}
    JPackage jPackage = codeModel._package("de.jensknipper.jcodemodel");
    JDefinedClass jClass = jPackage._enum("ExampleEnumName");
{% endhighlight %}

In case you do not want to declare a package, you can also generate an enum using the root code model.

{% highlight java %}
    JDefinedClass jClass = codeModel._class("ExampleEnumName", EClassType.ENUM);
{% endhighlight %}

## Adding a field

To create a field in your class you have to specify a modifier, the variable type and the name. 
Save the field into a local variable. 
You are going to need it in the getter method.

{% highlight java %}
    JFieldVar field = jClass.field(JMod.PRIVATE + JMod.FINAL, String.class, "exampleField");
{% endhighlight %}

## Adding a getter method

Adding a method works similar to adding a field. 
To make this method a getter method let the method body return the previously specified field.

{% highlight java %}
    JMethod getterMethod = jClass.method(JMod.PUBLIC, field.type(), "getExampleField");
    getterMethod.body()._return(field);
{% endhighlight %}

## Building a constructor

Enum constructors are private by default which makes it needless to declare a modifier at all. 
Constructors are methods with no return value and a predefined name, which makes the declaration of the method quite simple.
Add a parameter and in the body of the method assign the parameter to the field.
This way you can add multiple parameters to your constructor.

{% highlight java %}
    JMethod constructor = jClass.constructor(JMod.NONE);
    JVar constructorParam = constructor.param(String.class, field.name());
    constructor.body().assign(JExpr.refthis(field), JExpr.ref(constructorParam));
{% endhighlight %}

## Creating enum instances

Because an enum without instances is just useless we will now get into it.
To add an instance simply use the _enumConstant_ method in your previously specified class. 
Be aware that this calls the constructor we declared earlier.
To make the generated code compile we have to add an argument to the constructor call.
If you declared multiple parameters make sure to add all of them **in the correct order**.

{% highlight java %}
    JEnumConstant jEnum = jClass.enumConstant("ENUM_1");
    jEnum.arg((JExpr.lit("enumFieldValue")));
{% endhighlight %}

## Building the file

Simply define an output folder and let the model genearate your code.
All the classes, folders and packages you declared earlier will be generated for you.

{% highlight java %}
    File file = new File("src/main/java");
    codeModel.build(file);
{% endhighlight %}

## Output

JCodeModel will generate the following ready to use Enum for you.


{% highlight java %}
package de.jensknipper.jcodemodel;

public enum ExampleEnumName {
    ENUM_1("enumFieldValue");
    private final String exampleField;

    ExampleEnumName(String exampleField) {
        this.exampleField = exampleField;
    }

    public String getExampleField() {
        return exampleField;
    }
}
{% endhighlight %}

## Conclusion

JCodeModel is a nice framework, but lacks some solid documentation. 
I hope this article will help you to generate your own Java classes without crawling through the framework's JavaDoc or codebase.
You can check out he whole example code on [Github](https://gist.github.com/JensKnipper/2a267325faad1e3fa4f490d36cb2330a).

