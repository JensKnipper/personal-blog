---
layout: post
title: Manually testing email notifications with GreenMail mock mail server
author: jens_knipper
date: '2021-02-16 01:00:00'
last_modified_at: '2021-07-01 01:00:00'
description: Are you sending mails featured with HTML and CSS and want see how they look before pushing your changes to production? You also do not want mails to leak from your local environment to your customer? A mock mail server like GreenMail might just be the tool you need.
categories: Java, Spring, GreenMail, Docker
---
Whenever developing an application I aim to get local and testing environment as close to production as possible without neglecting elementary things like privacy and security. This gives an advantage when implementing new features. You get the same user experience on your local device. This enables you to find troublesome or unclear sections.  
It also lowers the barrier for manual testing, which results in more and higher quality feedback from everyone (not only developers) involved in the project. 

For testing purposes you usually do not want to connect to production systems. That is why local or test environments often use local instances of additional applications like a database. This can also be applied to a mail server. When executing a local instance of a mail server you have to configure it properly to avoid leakage of data to your customers. A mock mail server like [GreenMail](https://greenmail-mail-test.github.io/greenmail/) is built to keep data on the same machine thus serves as a good option.

## GreenMail Docker configuration

To use GreenMail as a mail server you should use the standalone version. Running it in a docker container is the simplest approach.

{% highlight yaml %}
version: '3'

services:
  greenmail:
    image: "greenmail/standalone"
    environment:
      - JAVA_OPTS=-Dgreenmail.verbose
    ports:
      - 3025:3025 # SMTP
      - 3110:3110 # POP3
      - 3143:3143 # IMAP
      - 3465:3465 # SMTPS
      - 3993:3993 # IMAPS
      - 3995:3995 # POP3S
{% endhighlight %}

There are a few environment variables you can set to configure GreenMail. A good option is setting the logging to verbose. This way you can identify misconfigurations easily.  
All of the most common mail protocols are supported. The ports have an offset of `3000` compared to the default port of the corresponding protocol. That's it, you are now able to use GreenMail in your application. 

Simply set `localhost` as the host and specify the port you want to connect to in your local environment's application properties.

## GreenMail mail client connection

To test your mails locally you can either connect to a desktop client like Thunderbird or install a web client. You could also connect mobile mail clients by opening your ports to your local network.  
When connecting to Thunderbird, make sure to configure the connection manually:
![GreenMail Thunderbird settings](/assets/img/greenmail-thunderbird-settings.png)

A simple but popular web client is [Roundcube](https://roundcube.net/). In this example running it in a docker container is also the simplest approach.

{% highlight yaml %}
  roundcube:
    image: roundcube/roundcubemail
    depends_on:
      - greenmail
    ports:
      - 8000:80
      - 9000:9000
    environment:
      - ROUNDCUBEMAIL_DEFAULT_HOST=greenmail  # IMAP server - tls:// prefix for STARTTLS
      - ROUNDCUBEMAIL_DEFAULT_PORT=3143       # IMAP port
      - ROUNDCUBEMAIL_SMTP_SERVER=greenmail   # SMTP server - tls:// prefix for STARTTLS
      - ROUNDCUBEMAIL_SMTP_PORT=3025          # SMTP port
{% endhighlight %}

To make Roundcube use GreenMail you habe to specify certain environment variables for the Roundcube container.  
You can read mails using the IMAP or POP3 protocol. You just need to specify the url and the port. Writing mails can be done using the SMTP protocol where you also have to specify the url and the proper port.

GreenMail automatically accepts all incoming mails. If there is no corresponding address, one will automatically be created. The password is equal to the mail address.  

In our example we browse to `localhost:8000` to use the webclient. If you want to use another port you can simply change the port in the compose specification. The result should look like this:
![RoundCube login](/assets/img/greenmail-roundcube-login.png)

Because of this you should not make the GreenMail and Roundcube instances publicly available. If you still want to integrate it in your test or staging environment it is highly recommended to secure it with a reverse proxy like nginx or [Traefik](../basic-authentication-with-traefik).

## Further readings

You can check out the whole configuration on [GitHub](https://github.com/JensKnipper/greenmail-example/blob/main/docker-compose.yml). There is also a small example application where you can test the described behaviour in action.
If you want to know a little more, you can watch my [meetup talk about GreenMail](../openvalue-meetup-greenmail-talk) where I also explain how to test email notifications.