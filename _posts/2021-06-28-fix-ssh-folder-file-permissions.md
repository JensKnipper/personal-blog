---
layout: post
title: Fixing permissions of files in ssh folder
author: jens_knipper
date: '2021-06-28 01:00:00'
description: I recently got an error on my work machine when copying ssh keys from Windows into WSL Linux. The error stated that the permissions were to open for my private key. 
categories: Linux, SSH
---
`Permissions 0644 for '/home/<user>/.ssh/id_rsa' are too open.`

Windows obviously does not set the proper access rights to the files when copying them.
Following an article from stackoverflow I simply set the access rights of all the files in the .ssh folder to `600`.
This solves the issue but is actually not better, because each file has it's own permission for a reason.
Some files are read only for security reasons, some are even changed by your ssh client and need a different permission due to that.

## Looking up correct ssh folder file permissions

So I settled to fix the files using my personal device.
First I had to look up the the files permissions. 
You can use the following commands to show the configuration of the ssh folder and it's contents.
{% highlight bash %}
    stat -c "%a %n" ~/.ssh
    stat -c "%a %n" ~/.ssh/*
{% endhighlight %}
That is how the results looked like on my work machine.
Be aware thet depending on your usage not all files may exist on your machine.
{% highlight bash %}
    700 ~/.ssh
    600 ~/.ssh/authorized_keys
    664 ~/.ssh/config
    400 ~/.ssh/id_rsa
    644 ~/.ssh/id_rsa.pub
    600 ~/.ssh/known_hosts
{% endhighlight %}
You might also want to take a look at the owner of the files with the following command.
{% highlight bash %}
    ls -ld ~/.ssh
    ls -l ~/.ssh
{% endhighlight %}
The username should be set as the owner and the group of the files.
In case it diverges we are going to fix it in the next step.

## Fixing the file permissions 

The following commands should set up everything as before.
{% highlight bash %}
    sudo chown -R $USER:$USER ~/.ssh
    sudo chmod 700 ~/.ssh
    sudo chmod 600 ~/.ssh/authorized_keys
    sudo chmod 400 ~/.ssh/id_rsa
    sudo chmod 644 ~/.ssh/id_rsa.pub
    sudo chmod 600 ~/.ssh/known_hosts
{% endhighlight %}
Make sure everything is working as before.
That's it. 
I fixed my messed up ssh folder configuration and I can be sure that security issues due to wrong configuration should not be a concern anymore.
