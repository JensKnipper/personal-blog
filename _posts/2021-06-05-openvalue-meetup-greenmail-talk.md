---
layout: post
title: Recording and Slides of My Talk "Testing email with GreenMail, a mock mail server" at an OpenValue Meetup
image: /assets/img/openvalue-greenmail-talk.jpg
author: jens_knipper
date: '2021-06-05 01:00:00'
description: Last week I hold my talk "Testing email with GreenMail, a mock mail server" at a virtual meetup of OpenValue. In this post, you can find the slides and the recording of my talk.
categories: Java, GreenMail, Talk
---
I really enjoyed the evening and the opportunity to speak at the [meetup](https://www.meetup.com/OpenValue/events/278102326/) from [OpenValue](https://www.openvalue.eu), thank you very much. Special thanks to [Bert Jan Schrijver](https://twitter.com/bjschrijver) for organizing the meetups.  
Unfortunately the audio quality is not as good as I hoped it would be.

## Abstract

Sending emails or receiving and processing emails is something that a lot of applications feature to their users. But how do you know that the code doing this actually works?
Even though email integrations are widespread functionalities, only few projects have a clear process of how to test them. Why? Because testing is sometimes challenging and people are afraid of accidentally leaking emails to real mail servers. Often these processes also do not offer a great developer experience.
GreenMail is an open source, intuitive and easy-to-use suite of mock email servers for testing purposes, which aims to solve these problems. In this presentation you will learn how to leverage GreenMail to manually and integration test your email functionality while at the same time offering a great development experience.

## Recording

<center>
    <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/p6gNiJryjvo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

## Slides

<center>
    <iframe src="https://slides.com/jensknipper/greenmail/embed" width="576" height="420" scrolling="no" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
</center>

## Code

The code examples shown in the talk are available on [GitHub](https://github.com/JensKnipper/greenmail-example).