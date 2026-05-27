---
draft: true
title: "Test email sending in Java using GreenMail"
date: 2026-01-19T01:00:00+00:00
description: "Integration tests can be hard, when you do not know where to integrate to. For emails there is a great tool called GreenMail which makes it a lot easier."
tags: ["Java", "Mail", "SMTP"]
---
https://github.com/JensKnipper/greenmail-example/blob/main/src/test/java/de/jensknipper/greenmailexample/control/mail/send/MailSendClientTest.java

https://github.com/JensKnipper/greenmail-example/blob/main/src/main/java/de/jensknipper/greenmailexample/control/mail/send/MailSendClient.java

In this short article I will show how to write an integration test for a service that sends mails using SMTP. 
We will make use of a mock mail server called GreenMail to verify that the mail we send actually arrives.
