---
title: "Generating generic fields in JCodeModel"
date: 2020-10-14T01:00:00+00:00
description: "Another article about JCodeModel. This time we will make a short excursion about how to generate generic fields in JCodeModel."
tags: ["Java", "jCodeModel"]
---
I recently wrote an article about [generating enums with JCodeModel](../generating-enums-in-java-with-jcodemodel/).
Due to the lack of documentation of JCodeModel using it can become very time consuming. 
I had to put in a lot of time to research a lot of basic stuff. 
This article will show you how to declare and assign generic fields.

## Getting started

As before to get started you have to create an instance of JCodeModel. You will need this object to generate your classes.

``` java
    JCodeModel codeModel = new JCodeModel();
```

## Creating a class

To create a class you need a package where the class is located. Package names can be specified as usual. 
JCodeModel will automcatically create the corresponding structure during code generation.

``` java
    JPackage jPackage = codeModel._package("de.jensknipper.jcodemodel");
    JDefinedClass jClass = jPackage._class("ExampleClassName");
```

In case you do not want to declare a package, you can also generate a class using the root code model.

``` java
    JDefinedClass jClass = codeModel._class("ExampleClassName");
```

## Declaring the field type

Building a class to declare fields is straightforward and can be done by referencing it with an instance of _JCodeModel_. 
Declaring generic types can be done by using the _narrow_ method. 
You can even create nested generics like `Map<String, List<String>>` by nesting the _narrow_ method.

``` java
    AbstractJClass classType = codeModel
        .ref(Map.class)
        .narrow(
            codeModel.ref(String.class),
            codeModel.ref(List.class).narrow(String.class));
```


## Using generic wildcards

When using generics you soon want to get into using wildcards.
To create a generic type simply reference a class using the JCodeModel instance and call the _wildcard_ method. 
You can specify the wildcard type with the _EWildcardBoundMode_ enum.

``` java
    JTypeWildcard wildcard = codeModel.ref(Comparable.class).wildcard(EWildcardBoundMode.EXTENDS);
    AbstractJClass classType = codeModel
        .ref(Map.class)
            .narrow(
                codeModel.ref(String.class),
                codeModel.ref(List.class).narrow(wildcard));
```

## Creating and setting the field

Starting with Java 7 there is no need to declare an instances generic type when you already declared it in the fields definition. 
Just let the angle brackets stay empty.
To achieve it in JCodeModel put an empty list into the _narrow_ method.

``` java
    JFieldVar field = jClass.field(JMod.PRIVATE + JMod.STATIC + JMod.FINAL, classType, "exampleField");

    JInvocation newMap = codeModel.ref(HashMap.class).narrow(Collections.emptyList())._new();
    field.init(newMap);
```

## Building the file

Simply define an output folder and let the model genearate your code.
All the classes, folders and packages you declared earlier will be generated for you.

``` java
    File file = new File("src/main/java");
    codeModel.build(file);
```

## Output

JCodeModel will generate the following ready to use class for you.

``` java
package de.jensknipper.jcodemodel;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class ExampleClassName {
    private static final Map<String, List<? extends Comparable>> exampleField = new HashMap<>();
}
```

## Conclusion

JCodeModel is a nice framework, but lacks some solid documentation. 
I hope this article will help you to generate your own Java classes without crawling through the framework's JavaDoc or codebase.
You can check out he whole example code on [Github](https://gist.github.com/JensKnipper/9b2591d8871d605121f81aa6dd3135ad).

