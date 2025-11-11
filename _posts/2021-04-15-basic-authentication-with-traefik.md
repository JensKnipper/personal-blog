---
layout: post
title: Adding basic authentication to secure a service with Traefik
author: jens_knipper
date: '2021-04-15 01:00:00'
last_modified_at: '2025-11-11 01:00:00'
description: Adding authentication to a service that does not support it by default can be done easily by using Traefik. This way you gain an additional layer of security and you can leverage other features of Traefik like domain names. 
categories: traefik, docker, docker-compose, authentication
---
In this article I will show you how to secure a service in Traefik reverse proxy using basic authentication.
You could use the authentication for example to secure your [Traefik dashboard](../exposing-traefik-dashboard/).
The example can be executed locally which allows simple adjustment to your own needs.
Technologies used are only docker and docker-compose.
For the purpose of simpler declaration I will not make use of configuration files, but only use docker labels.

## Traefik configuration

A minimalistic configuration of Traefik can be seen in the code block below. 
Only port `80` is exposed end assigned to the `web` entrypoint.
All incoming traffic is now routed through this specific entrypoint.

{% highlight yaml %}
  traefik:
    image: traefik:v3.5
    command:
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
{% endhighlight %}


## Container configuration

To keep it simple we use the `whoami` image from _containous_, the company behind Traefik.
We set it up to hook to the just specified entrypoint and tell it to listen to the domain `whoami.localhost`.
This way we can easily access it locally.

To secure this service you have to add a middleware called `auth`. 
With another label we can add basic authentication and specify user credentials. 
Username and password are separated by a simple colon. 
You can add multiple users by separating them via semicolon.

For password creation have a look at the next chapter.

{% highlight yaml %}
  whoami:
    image: traefik/whoami
    labels:
      - "traefik.http.routers.whoami.entrypoints=web"
      - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
      - "traefik.http.routers.whoami.middlewares=auth"
      - "traefik.http.middlewares.auth.basicAuth.users=test:$$2y$$12$$ci.4U63YX83CwkyUrjqxAucnmi2xXOIlEF6T/KdP9824f1Rf1iyNG"
{% endhighlight %}

This example resolves to the credentials with both username and password `test`.

If you now start the services with `docker-compose up` and navigate to `whoami.localhost` you will be prompted to type in the credentials.
After successfull login you should be able to access the container's content.

### Password hashing

Traefik supports different hash algorithms to secure your services. 
Most of these hash algorithms like MD5 or SHA-1 are considered unsafe and not recommended for production use.
That is why you should stick to [bcrypt](https://auth0.com/blog/hashing-in-action-understanding-bcrypt/).

To create a hashed passphrase you can make use of an [online generator](https://bcrypt-generator.com/).
Twelve is the default number of rounds for bcrypt, though Traefik is able to handle any number of rounds. 
A higher number increases the complexity of encryption which results in a safer hash, but may impact your latency.
You can try some numbers to find your sweet spot or simply stick to the default.
Make sure to replace all single dollar chars in your hashed password with double ones for escaping.

## Further readings

As always you can check the full source code on [GitHub](https://github.com/JensKnipper/traefik-examples/blob/master/authentication/basic-authentication/docker-compose.yml).
