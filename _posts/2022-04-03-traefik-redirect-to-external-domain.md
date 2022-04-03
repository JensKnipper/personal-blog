---
layout: post
title: Redirecting to an external domain with traefik
author: jens_knipper
date: '2022-04-03 01:00:00'
description: Redirecting to another domain is a regular use case I need a lot. Luckily Traefik makes it easy to implement it.
categories: traefik, docker, docker-compose, redirect
---

In this article I will show you how to use redirects to an external domain in Traefik.
The example can be executed locally which allows simple adjustment to your own needs.
Technologies used are only docker and docker-compose.
For the purpose of simpler declaration I will not make use of configuration files, but only use docker labels.

## Traefik configuration

A minimalistic configuration of Traefik can be seen in the code block below. 
Only port `80` is exposed end assigned to the `web` entrypoint.
All incoming traffic is now routed through this specific entrypoint.

The configuration of the redirect is in the labels section of the Traefik container. 
No further containers are needed.

In the labels section a router listening to the URL `redirect.localhost` in the web entrypoint is defined.
A middleware of the type `redirectregex` is dealing with all incoming requests.  
A regex is defined to filter the incoming requests.
The regex `(.)*` will allow all incoming requests.
None will be filtered out.  
The replacement is the URL where we want to redirect to.
In this case it is set to my website.

For testing purposes I would recommend to set the redirect to temporarily (HTTP Code 302) by setting the attribute `permanent` to `false`.
This avoids a lot of pain while testing, because the browser will not cache the redirect.
Depending on your use case you might want to set it to `permanent` (HTTP Code 301) as soon as you are done.

{% highlight yaml %}
  traefik:
    image: traefik:v2.2
    command:
      - "--providers.docker"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    labels:
      - "traefik.http.routers.redirect-router.entrypoints=web"
      - "traefik.http.routers.redirect-router.rule=Host(`redirect.localhost`)"
      - "traefik.http.routers.redirect-router.middlewares=redirect-regex"
      - "traefik.http.middlewares.redirect-regex.redirectregex.regex=(.)*"
      - "traefik.http.middlewares.redirect-regex.redirectregex.replacement=https://jensknipper.de"
      - "traefik.http.middlewares.redirect-regex.redirectregex.permanent=false"
{% endhighlight %}

You should be aware that multiple levels of redirects do not work.
If you also want to do a [global https](../traefik-http-to-https-redirect) and www redirect for your domain you are not able to do both. For that I would recommend using the following [docker container](https://hub.docker.com/r/schmunk42/nginx-redirect) and keep the global redirect.
Make sure to add `https://` or only the curent path will be replaced and you would end up at `redirect.localhost/jensknipper.de`.

If you now start the service with `docker-compose up` and navigate to `redirect.localhost` you should be redirected to my website.

## Further readings

As always the full source code is available on [GitHub](https://github.com/JensKnipper/traefik-examples/blob/master/redirects/redirect-to-external-url/docker-compose.yml).