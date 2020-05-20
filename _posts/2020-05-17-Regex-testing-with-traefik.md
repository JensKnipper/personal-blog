---
layout: post
title: Testing Regular Expressions in Traefik
author: jens_knipper
date: '2020-05-17 20:00:00'
last_modified_at: '2020-05-21 20:00:00'
description: Testing regular expressions for application configuration is always a pain as every programming language has a slightly different implementation. This results in expressions giving different results for different applications.
categories: Traefik, Regex, Regular Expression, Go
---
Sometimes it is not possible or just too much effort to make the Traefik configuration work locally. Testing on a production environment can be dangerous thus should not be an option. In these cases testing small parts of the configuration is essential. Regular expressions are one example where you can save a lot of time when testing outside of traefik.  
The Reverse Proxy [Traefik](https://traefik.io) is written in Go. That is why Traefik uses the [regex implementation of Go](https://golang.org/pkg/regexp/syntax/). Be aware that Go also introduces a slightly different syntax for its regular expressions.  
Now that we know which expressions to test it becomes easy to find a tester for Go regex. I personally prefer [regex101.com](https://regex101.com/). The site lets you chose between different regex implementations. Make sure to pick the Golang flavor. It also makes it easy to test your possible input and displays information about matches, which comes in handy when testing redirects. 