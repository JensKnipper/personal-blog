---
layout: post
title: Linux initial server setup
author: jens_knipper
date: '9999-10-18 01:00:00'
description: 
categories: Linux, Ubuntu, Server
---
Whenever I am setting up a Linux server I end up doing the same steps no matter how it is finally being used. 
Whether it ends up being a docker host, serves some static sites or an application there are some steps to secure the server which remain the same. 
I always keep a text file with some shell commands with no explanations shamelessly copy and pasted from different articles. 
The time has come to change this and document the setup properly.
  
The steps are needed to secure the server from the most common security threats.
Even though the example commands are intended for Ubuntu 20.04 they can be applied to other versions or even different distributions. 

## Secure SSH login

There are different approaches to this. 
Usually permitting password login and setting up SSH keys should be sufficient. 
For adding further security you can also create a new user and block root login. 
This way an attacker has to get your login name **and** your password before accessing the server.
Because the setup with another user is not really different I will not explain it any further.

### Setup SSH keys

At first we will take care of setting up SSH keys for your user.
This way you will be able to login without a password and block password login.

#### Create local RSA Key Pair

Your private key should be kept absolutely private and **never** leave your local device.
That is why it is recommended to generate the key locally and not on the server.
To generate a key locally simply type in the following command.

{% highlight bash %}
ssh-keygen -t rsa
{% endhighlight %}

You will be asked where to store the keys and whether you want to secure your freshly generated key with a password. 
Generating a password is optional and does not add additional security to the server itself.
But in case your private key gets in someone else's hands, it may buy you some time to create and upload a new pair of keys before your key gets corrupted.
To use the defaults simply press enter to confirm.

As soon as the generation is finished you can check the output with the following command (given that you dit not change the default location).

{% highlight bash %}
ls ~/.ssh/
{% endhighlight %}

There should be at least two files: `id_rsa` and `id_rsa.pub`.
The first key is your private key and should never leave your hands.
The second one with the _.pub_ ending is your public key, which we will transfer to your server in the next step.

#### Copy the public key to your server

TODO ssh command cat public key with set into authorized keys

### Do not allow password login
secure ssh
allow root login or use another user

### 
https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04

## Automatically install updates
unattended upgrades
to reboot or not?

## Install a firewall
intall ufw or not?
not if you want to install docker

## Install brute force intrusion prevention
install fail2ban




https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04
https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2
https://www.howtoforge.com/tutorial/how-to-setup-automatic-security-updates-on-ubuntu-1604/
https://www.digitalocean.com/community/tutorials/how-fail2ban-works-to-protect-services-on-a-linux-server


ssh-keygen
cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys

alternativ
ssh-copy-id username@remote_host

vi /etc/ssh/sshd_config

PermitRootLogin yes
PermitEmptyPasswords no
PasswordAuthentication no
UsePAM no

sudo systemctl reload sshd


sudo apt install unattended-upgrades
vi /etc/apt/apt.conf.d/50unattended-upgrades
"${distro_id}:${distro_codename}-updates";

vi /etc/apt/apt.conf.d/20auto-upgrades
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";

sudo apt install fail2ban

sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

vi /etc/fail2ban/jail.local
enabled	= true
filter	= sshd
maxretry = 4

sudo systemctl restart fail2ban.service


sudo apt install docker.io docker-compose
usermod -a -G docker $USER
systemctl restart docker

sudo apt install git