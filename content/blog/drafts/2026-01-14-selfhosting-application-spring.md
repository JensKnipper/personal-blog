---
draft: true
title: "I wrote a self-hosting application in Java and Spring Boot"
date: 2026-01-19T01:00:00+00:00
description:
---

I am running a lot of applications at home. Before, I was self-hosting my applications behind Traefik reverse proxy and defined the redirects in there.

My Docker Compose file got longer and longer to the point where it was barely readable at all. Also, the process of editing it was cumbersome: SSHing into the machine, editing the file with Vim and restarting the service.  
I also tried out different URL shorteners, but they were either difficult to set up or where doing so many more things. Sometimes they did not even provide the things that I needed.

I wanted to have something simpler, with a Web UI. So I just wrote one myself.  
This is article is about how I implemented it and which decisions and experiences I made in the process.

## What I implemented

re:Director lets you create redirects through a simple web interface. All you have to do is define which url should be redirected to which target. Just make sure the that the actual domain points to re:Director.
It's an open-source and self-hostable alternative to many SaaS solutions out there.

## Tech stack

The tech stack represents what I am most comfortable with. I worked on it in my free time, so I wanted to be fast and not turn it into a time sink.
I did deviate from the default and chose a few technologies new to me. Some for personal reasons, some because writing self-hosting applications is a little different to regular business applications. Let me explain my reasons here:

### Backend

TODO restructure stuff here
* Java 25
* Spring Boot

Though I like Kotlin, the latest features in Java are super nice to work with and give me less reason to switch. Because I mostly use Java in my day job I also chose it here.
I do like Quarkus and it's developer experience. But I am just not that familiar with it to be similarly productive as with Spring Boot.

### Frontend

* JTE
* Pico CSS

I am most comfortable in the backend, though I do know my way around the modern frontend frameworks. I usually prefer Svelte, but this project was going to start small and most important also stay small. Essentially it is just a simple CRUD app around the redirect part.
That's way I wanted to keep the frontend simple and defaulted to JTE. I initially started with Thymeleaf, but quickly had more and more issues with its model binding. Just by switching to JTE all those issues were resolved and the templates also gained type safety as a boon.
I really love Pico CSS. You essentially write plain HTML, add Pico CSS and boom, you're done. You get a relative nice frontend without the CSS class mess Bootstrap or Tailwind require.

https://jte.gg/

https://www.classlesscss.com/
https://github.com/dbohdan/classless-css
https://picocss.com/

### Database

* jOOQ
* Liquibase
* SQLite

This is a combination that is not that common in the Spring ecosystem.
The thing is that I really don't like JPA and Hibernate. The abstraction is just too far away from the database and I always feel like doing things twice: once in the entities and once in the Liquibase scripts.
With jOOQ I don't have that feeling anymore. The DB is the single source of truth and the DAOs will be generated from it. The DSL is super close to SQL, so I don't have to know SQL AND the framework.
I use Liquibase to manage changes in the DB schema. I am also comfortable with Flyway and don't really have a strong opinion for or against one or the other.
Using SQLite was a strict requirement for me, because of the self-hosting part. When self-hosting I want a simple application I can run in a docker container, preferably without an extra database container running. SQLite is just perfect for that. I can run it in in-memory mode for testing and don't have to rely on TestContainers. I can run it in file mode everywhere else and the user can create a volume to persist the file outside the docker containers lifecycle.

### Build

* Maven
* Jib

I never got warm with gradle (and also groovy). Breaking changes in different versions, every project feels different due to the scripting, ... I just think Maven is the better alternative, because it brings less complexity.
I know that Spring brings its own mechanism for building docker images, but I am using Jib here. The pros: it does not need docker for building, it's superfast, and you can create amd and arm images on the same machine. Super comfortable to keep the build simple.

## Summary

TODO write how it went
You can checkout the project and the source code yourself.

* Website: [https://re-director.github.io/](https://re-director.github.io/)
* Sources: [https://github.com/re-Director/re-director](https://github.com/re-Director/re-director)