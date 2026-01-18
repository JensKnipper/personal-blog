---
layout: post
title: Using Maven on Windows with the Linux subsystem
author: jens_knipper
date: '2020-07-03 01:00:00'
last_modified_at: '2025-01-23 01:00:00'
description: Installing and configuring Maven on Windows does not have to be a pain anymore. Thanks to the WSL it becomes almost as straightforward as using Linux. Although depending on your use case there might be some restrictions.
categories: Windows, Linux, WSL, Maven, IntelliJ
---
> **_NOTE:_** The information on this article is mostly outdated, because there have been a lot of changes in the WSL. I would recommend to use the [WSL GUI](https://github.com/microsoft/wslg) now and install all my programming related things in the WSL. Starting any application from the WSL with a GUI will now open a window in the Windows desktop.

Because I am a Linux user by choice, being forced to use Windows never feels quite right. Especially when installing and configuring your development setup, Windows really feels cumbersome.

The [Windows Subsystem for Linux](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux) does a lot of things right. Using it in combination with the new [Terminal](https://github.com/microsoft/terminal) for Windows you can get a really neat setup that comes close to using just Linux itself.

## Painpoints of the Windows Subsystem for Linux
One thing that sometimes feels a little odd is the integration of Windows and the WSL. The path variable of Windows is integrated into the WSL and you can call all your installed Linux applications in the Windows shell by prepending `wsl`. You can even access all your Windows files as the file system is mounted into the subsystem.

Given these features, there still are some restrictions when trying to access the other way around. It is not always that easy to use applications installed in the subsystem for usage in Windows.  
One big concerns are applications that rely on other applications. For example an installed version of Maven in the Linux subsystem cannot easily be used by IntelliJ or any other IDE in Windows. In this case you have to rely on the bundled Maven version IntelliJ is shipping. This results in two installed versions on your computer, which both use different repositories with probably mostly the same content.

## Shareing Maven configuration between WSL and Windows
A workaround can be to share the configuration between WSL and Windows by linking the Linux Maven folder to the one from Windows. This can be simply done with the following command:  
`ln -s /mnt/c/Users/<username>/.m2 ~/.m2`  
Prepend `wsl` to the command when using it in a Windows cmd or Powershell.

This Workaround can also be applied to other applications like git, ssh or whatever you like, to share the configuration between the two systems. It works on the first version of WSL and also on WSL2.

## Avoid prepending the WSL-Keyword by using dosKey
To avoid prepending `wsl` all the time you can use the [dosKey](https://4sysops.com/archives/using-doskey-aliases/) command. This way it feels even more like calling the commands natively.  
For Maven the dosKey may look like this:  
`doskey mvn=mvn wsl $*`  
Be aware that the command is not boot safe and out of the box it is only working in cmd, not in Powershell. Though all these points can be fixed.

## Going full Linux with X-Server
Probably the most sustainable, but also elaborate workaround is [installing an X-Server and accessing it from Windows](https://github.com/lackovic/notes/tree/master/Windows/Windows%20Subsystem%20for%20Linux#run-a-linux-gui-application-in-wsl-2). It is actually the one I can recommend most. There may also be a follow up article covering this topic.

## WSLg
Probably the best approach is using the new [WSL GUI](https://github.com/microsoft/wslg). 
It uses X11 and RDP in the background and enables using your fully fledged IDE in your WSL.
The integration into the desktop is seamless and easy to use.
The only downside is that it is only supported for Windows 11.

## Conclusion
You can really ease your development processon Windows by using WSL. In most cases simple workarounds can be applied to make the integration of the two operating systems more seamless. When using a more complex development setup the effort of installing an X-Server may be worth it to get the full Linux experience. For Windows 11 users using WSLg is probably the easiest and best integrated solution.