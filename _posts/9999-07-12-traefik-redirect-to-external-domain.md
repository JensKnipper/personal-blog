---
layout: post
title: Redirecting to external domain with traefik
image: /assets/img/traefik_dashboard.png
author: jens_knipper
date: '9999-07-12 01:00:00'
description: 
categories: traefik, docker, docker-compose, redirect
---


## Traefik configuration



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

## Further readings

As always the full source code is available on [GitHub](https://github.com/JensKnipper/traefik-examples/blob/master/redirects/redirect-to-external-url/docker-compose.yml).