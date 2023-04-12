---
layout: post
title: Recording and Slides of My Talk "Local testing of AWS serverless lambda functions" at an OpenValue Meetup
image: /assets/img/openvalue-lambda-talk.jpg
author: jens_knipper
date: '2022-11-10 01:00:00'
description: Last week I hold my talk "Local testing of AWS serverless lambda functions" at a virtual meetup of OpenValue. In this post, you can find the slides and the recording of my talk.
categories: Java, GreenMail, Talk
---
I really enjoyed the evening and the opportunity to speak at the [meetup](https://www.meetup.com/openvalue-dusseldorf/events/289041593/) from [OpenValue](https://www.openvalue.eu), thank you very much. Special thanks to [Sebastian Konieczek](https://twitter.com/the_real_sko) for organizing the meetups.  

## Abstract

There are a lot of resources and best practices about testing of microservices. 
Serverless functions like AWS Lambdas are still mostly tested manually after a deployment, especially concerning logic provided by AWS. Though it is possible to start the application locally and even test it. But how does it work for a (relatively) closed system like the AWS cloud? 
Through smart usage of LocalStack, TestContainers and the AWS SDK it is possible to automatically execute component tests and uncover possible errors in the usage of the lambda function. 

## Recording

<center>
    <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/5fPfKW-X5gA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

## Slides

<center>
    <iframe src="https://slides.com/jensknipper/testing-aws-lambdas/embed?style=light" width="576" height="420" title="Local Testing of AWS Serverless Lambda Functions" scrolling="no" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
</center>

## Code

The code examples shown in the talk are available on [GitHub](https://github.com/JensKnipper/testing-aws-lambdas).