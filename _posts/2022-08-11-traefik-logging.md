---
layout: post
title: Enable logging in Traefik
author: jens_knipper
date: '2022-08-11 01:00:00'
description: In this article I will show you how to activate logging in Traefik. There are two kinds of logs in Traefik, the general log and the access log. We will have a look at both of them and take a deeper look into the possibilities they provide.
categories: traefik, docker, docker-compose, redirect
---

The examples can be executed locally which allows simple adjustment to your own needs.
Technologies used are only docker and docker-compose.
For the purpose of simpler declaration I will not make use of configuration files, but only use docker labels.

Let's get started with the deeper settings Traefik provides and then get into the general log and the access log.

## Settings for both logs

There are different settings both logs share.
Getting to know them is crucial when investigating problems with your settings. 
You might want to adjust them temporarily for debugging purposes.

### Log levels
As in many other programmes there are multiple log levels.
The log levels in order of severity are DEBUG, INFO, WARN, ERROR, FATAL and PANIC.
DEBUG is the lowest level and PANIC the highest.
Setting a higher level reduces the amount of messages you receive.

Traefik's default log level is ERROR.
It's a good level to start with, but when toubleshooting a problem you might need further information.
Temporarily choosing a less severe level is a good place to get started.

You can set the level in the command section using the following statement `log.level=DEBUG`.

### Log formatting

As a typical cloud native application Traefik allows logging in json format, which different log aggregators like Datadog, Graylog and others use.

The default option for logging is `common`.
Setting it to json requires to set a variable in the command section.
The statement to set it to json in the general log is `log.format=json`.
The statement for the access log is `accesslog.format=json` .

### Timezone and timestamps

Traefik logs timestamp in UTC time by default. 
To change this behaviour there are a few options.

You can set the timezone using the environment variable called `TZ`. 
An example can be found in the following code block.

{% highlight yaml %}
environment:
  - TZ=Europe/Berlin
{% endhighlight %}

A list of all timezones can be found [here](https://go.dev/src/time/zoneinfo_abbrs_windows.go
).

Another way of changing the timezone is by using a volume.
You simply mount the timezone and the localtime from the host machine into the container.

{% highlight yaml %}
volumes:
  - /etc/timezone:/etc/timezone:ro
  - /etc/localtime:/etc/localtime:ro
{% endhighlight %}

In case you are unsure how your host machine is configured you can check the localtime and timezone with the following bash command on your machine: `timedatectl`.

To get a list of all available timezones you can use the following command: `timedatectl list-timezones`. 
You can navigate and search the list with the common vim bindings.

To set your timezone the following statement from the codeblock may be used.

{% highlight bash %}
sudo timedatectl set-timezone <your_time_zone>
# e.g.
sudo timedatectl set-timezone Europe/Berlin
{% endhighlight %}

Afterwards the timezone from your host machine should match the timezone in your Traefik container.

## Logging

Let's get to the logging itself and have a look into the next code block how to set the log flie and how to define a log file.  
By defining the a file you are able to access the logs e.g. after recreating the containers. 
A volume has to be defined to persist the file.

{% highlight yaml %}
  traefik:
    image: "traefik:v2.2"
    command:
      - "--providers.docker"
      - "--log.level=DEBUG"
      - "--log.filePath=/logs/traefik.log" 
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./logs/:/logs/
{% endhighlight %}

If you now start the service with `docker-compose up` Traefik starts writing to the log on startup. 
You can see the entries by looking into the log using `less logs/traefik.log`.

The full source code is also available on [GitHub](https://github.com/JensKnipper/traefik-examples/blob/master/logging/enable-logging/docker-compose.yml).

## Activate access logging

Access logging is quite similar to regular logging. 
Next to defining the path to the file, you have to activate it.  
You also need a service, accessible through Traefik, to see this behaviour. 

{% highlight yaml %}
  traefik:
    image: "traefik:v2.2"
    command:
      - "--providers.docker"
      - "--entrypoints.web.address=:80"
      - "--accesslog=true"
      - "--accesslog.filePath=/logs/access.log"
    ports:
      - "80:80"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./logs/:/logs/

  whoami:
    image: containous/whoami
    labels:
      - "traefik.http.routers.whoami.entrypoints=web"
      - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
{% endhighlight %}

If you now start the service with `docker-compose up` and navigate to `whoami.localhost` this access should appear in the access log.  
You can see the entry looking into the log using `less logs/access.log`. 
It should look similiar to the following log entry:
```
<your-ip-address> - - [14/Aug/2022:00:00:00 +0000] "GET / HTTP/1.1" 200 691 "-" "-" 6 "whoami@docker" "http://<your-ip-address>:80" 1ms
```

The full source code is also available on [GitHub](https://github.com/JensKnipper/traefik-examples/blob/master/logging/enable-access-logging/docker-compose.yml).

### Further settings

Access logging has a few more options to choose from. 

#### Buffering size

You may set a bufferingsize for performance reasons. 
Traefik will keep a certain amount of log lines in memory before writing them to the log file. 
This way the amount of writes to the filesystem is reduced. 
The following statement does the magic: `--accesslog.bufferingsize=50`

#### Filtering 

Another way to optimize performance and save some data on your harddrive is filtering. 
An overview of the different filter methods can be seen in the following code block.

{% highlight yaml %}
--accesslog.filters.statuscodes=200,300-302
--accesslog.filters.retryattempts
--accesslog.filters.minduration=10ms
{% endhighlight %}

There are options to allow only certain statuscodes. 
It is possible to list them comma separated and also include ranges.  
The filter `retryattempts` only shows entries with at least one retry.  
To show only requests which take a certain time the `minduration` filter may be used.
Valid timeformats for the duration can be found [here](https://pkg.go.dev/time#ParseDuration).  
These filters provide a nice way to reduce log entries and to focus only on certain edge cases where the performance might be impaired.