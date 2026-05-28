---
draft: true
title: "Demystifying Dependency Injection - Related Terms"
date: 2026-01-19T01:00:00+00:00
description: "There are a few terms tied to Dependency Injection where the relation or the definition is sometimes kinda vague. This is something I would like to clarify with this article."
cover:
    image: /img/dependency-injection.png
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
---

The two related concepts __Dependency Inversion Principle (DIP)__ and __Inversion of Control (IoC)__ will be explained in respect to how they are related to __Dependency Injection (DI)__

> This article is part of a series that goes in depth on Dependency Injection (DI). In this series, [we learn what DI actually is and why we would want to use it](https://jensknipper.de/blog/demystifying-dependency-injection-what-and-why/). Once you start digging into it, you will encounter quite a few related terms, which will be explained. There are also multiple ways to inject dependencies, with even more names and smaller variations. We will cover those and also explore the different ways dependencies can be wired and which frameworks can help us do that.

## Dependency Inversion Principle (DIP)

Whoever starts reading about Dependency Injection, is quickly confronted with the Dependency Inversion Principle, but that's also where the definitions start to get a little vague.  
So let's bring a little clarity into it.

The Dependency Inversion Principle is frequently used in combination with Dependency Injection. Often, you can read that it is the same, but actually these are two different things.  
Dependency Injection is just about the injection of the dependency, nothing more. When you combine it with the Dependency Inversion Principle, the client does not know which implementation of the dependency is used. This can be achieved e.g. by using an interface. Instead of depending of the actual implementation the client just expects the interface.

![Dependency Inversion Principle](/img/dependency-inversion-principle.png)

Using the DIP is great when you want to write a mock manually and pass it into the client. Another use case would be when you want to use a fake.
A fake is similar to a mock, but instead of returning somewhat static values, you imitate the behaviour. I did that in customer projects and replaced a key value store like [Redis](https://redis.io/) with a HashMap in my tests. It made the tests really nice to use, because you don't have to take care of the behaviuor anymore. It is even reusable, so you could theoretically use it in all your tests. 
But as always there is no light without shadow. Depending on the features of Redis that you use, your fake might get complicated really quickly. Think about features like key expiration and many more. You have to make sure, that your fake is correct, otherwise your tests might lie to you. And do you really want to write tests for your test utilities?

With mocking frameworks like [Mockito](https://site.mockito.org) you don't absolutely need the DIP to create a mock. So to keep things simple, you can just leave it out.

## Inversion of Control (IoC)

Inversion of Control is another thing you hear a lot when dealing with Dependency Injection. In contrast to procedural programming, there is no central entry point, instead the control flow is controlled by the framework. 
g

- instead the control flow is controlled by the framework
- Spring is actually a good example
  - things like RequestFilter or RestController implement it
  - for a RequestFilter all I got to do is annotate with @Component and implement the interface
  - RestController is even easier, I just have to annotate my class with @RestController
  - Spring takes care to call the classes when needed
- classes are decoupled from the rest of the application
- enables simple and isolated testing
- DI partially uses main idea behind the principle
  - control of the lifecycle of a dependency is inverted
- DI is not IoC, but half of it, the part about control flow is missing
- this is where a lot of sources get it wrong in my opinion - they say DI is IoC