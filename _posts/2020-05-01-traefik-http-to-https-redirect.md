---
layout: post
title: HTTP to HTTPS redirects with Traefik
author: jens_knipper
date: '2020-05-01 12:00:00'
description: This article covers the basic examples how to redirect HTTP-requests to HTTPS. This can be achieved per domain, for a single application only or globally for all containers.
categories: traefik, docker, docker-compose, http, https, ssl
---
Setting up SSL-Encryption with Traefik is incredibly easy due to the included ACME resolver. There is no reason not to use it, as search engines usually rank pages without HTTPS lower than pages which implement the protocol. Also security of yourself and your users should **always** be a critical concern.   
No redirect to HTTPS may cause users to use your unencrypted site  thus exposing them to a great number of potential vulnerabilities by sending their private data unencrypted across the internet.  
Even your SEO rank might suffer due to search engines thinking that your content is duplicated as it is served under two different urls - HTTP and HTTPS. Same applies to www redirects which will be handled in another article.  
Save yourself the trouble and enforce HTTPS. Here is how to.  

## Global HTTP to HTTPS redirect
The easiest and probably most used variant is to redirect to HTTPS globally. Once configured all your running services will use it. 

### Traefik configuration
The configuration of Traefik is displayed in the following code block. Due to the configuration in the `ports` section, the ports `80` and `443` of the Traefik Container are mapped to the port of the machine running the container. The ports are each mapped to specific entrypoints. `443` to the `websecure` and `80` to the `web` entrypoint. The `web` entrypoint additionally has a redirection to the `websecure` entrypoint. The scheme is defined as `https`, a predefined scheme by Traefik which turns HTTP into HTTPS.  
This results in all traffic coming in at port `80` being redirected to port `443` - the default HTTPS port.

{% highlight yaml %}
  traefik:
    image: traefik:v2.2
    command:
      - "--providers.docker"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entrypoint.to=websecure"
      - "--entrypoints.web.http.redirections.entrypoint.scheme=https"
      - "--entrypoints.websecure.address=:443"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
{% endhighlight %}

### Container configuration
If we would now add a container we could see the redirect in action. We can just use a simple whoami service which displays information about the user accessing it.  
As all the traffic is redirected to `websecure`, there is no need to setup a router for the `web` entrypoint.  
TLS has to be set to true to enable transport encryption.  
Type in the urls `http://whoami.localhost` and `https://whoami.localhost` and you will see that you allways end up accessing the service via HTTPS.

{% highlight yaml %}
  whoami:
    image: containous/whoami
    labels:
      - "traefik.http.routers.whoami.entrypoints=websecure"
      - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
      - "traefik.http.routers.whoami.tls=true"
{% endhighlight %}

The full example code can also be found on [Github](https://github.com/JensKnipper/traefik-examples/blob/master/redirects/http-to-https-redirect/http-redirect-global/docker-compose.yml).  
All the code may be executed locally by simply running the command `docker-compose up`.   

## Per Domain HTTP to HTTPS redirect
The Traefik configuration becomes a little simpler as there is no need to redirect anymore. At the same time configuration of containers redirecting to HTTPS becomes a little more complex.

### Traefik configuration
The ports and entrypoints are defined the same way as in the global example. Only the redirect is left out.  
All the services can now be accessed via HTTP and HTTPS.

{% highlight yaml %}
  traefik:
    image: traefik:v2.2
    command:
      - "--providers.docker"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
{% endhighlight %}

### Container configuration
Because of all the services possibly being accessable via HTTP and/or HTTPS we have to create routers for the specific entrypoints. The `web` entrypoint additionally gets a middleware `whoami-https`. The middleware is a redirection to HTTPS using the same scheme as in the global example.  
As all the traffic is now redirected to HTTPS an entrypoint for it has to be defined. This can be done using the same `labels` as in the global example.  
Type in the urls `http://whoami.localhost` and `https://whoami.localhost` and you will also see that you allways end up accessing the service via HTTPS.

{% highlight yaml %}
  whoami:
    image: containous/whoami
    labels:
      - "traefik.http.routers.whoami-http.entrypoints=web"
      - "traefik.http.routers.whoami-http.rule=Host(`whoami.localhost`)"
      - "traefik.http.routers.whoami-http.middlewares=whoami-https"
      - "traefik.http.middlewares.whoami-https.redirectscheme.scheme=https"
      - "traefik.http.routers.whoami.entrypoints=websecure"
      - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
      - "traefik.http.routers.whoami.tls=true"
{% endhighlight %}

For testing purposes we can add another container which only listens to the HTTP port.  
Type in the url `http://whoami.localhost`. It should not redirect to HTTPS. Accessing in the url `https://whoami.localhost` will cause a 404 error, because no entrypoint is defined and the page cannot be found.

{% highlight yaml %}
  whoami2:
    image: containous/whoami
    labels:
      - "traefik.http.routers.whoami2.entrypoints=web"
      - "traefik.http.routers.whoami2.rule=Host(`whoami2.localhost`)"
{% endhighlight %}
The full example code can also be found on [Github](https://github.com/JensKnipper/traefik-examples/blob/master/redirects/http-to-https-redirect/http-redirect-per-domain/docker-compose.yml).  
All the code may be executed locally by simply running the command `docker-compose up`.  

## Conclusion
Not only setting up HTTPS, but also the redirection from HTTP to HTTPS is easily done using Traefik. Keeping in mind the disadvantages of not using it there is no reason not to do it. 