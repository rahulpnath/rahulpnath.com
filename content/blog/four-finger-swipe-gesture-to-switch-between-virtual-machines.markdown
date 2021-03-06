---
layout: post
title: "Tip of the Week: Four Finger Swipe Gesture to Switch Between Virtual Machines"
comments: true
categories: 
- TipOW
tags: 
date: 2018-01-31
completedDate: 2018-01-31 05:57:40 +1000
keywords: 
description: Easily switch between VM's using the Windows Virtual Desktop feature.
primaryImage: virtual_desktop.png
---

Of late I have been working for multiple clients at the same time. Different clients have different development environments, which has forced me into using Virual Machines (VM's) for my day to day work. I will cover my actual setup and new way of working using VM's in a different post. 

When working on VM's I often have to switch to the host machine for email, chat and a few other programs that I have just on my host machine. Minimizing the VM host is time consuming and context breaking if you are working off a single screen. On a multi monitor setup you can always have VM on one screen and host on the other. This can still get tricky if you have more than one VM's connected.

<div style="text-align: center;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/fcbqOe8LnhM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>


The [Virtual Desktops](https://blogs.windows.com/windowsexperience/2015/04/16/virtual-desktops-in-windows-10-the-power-of-windowsmultiplied/) feature in Windows 10 is of great help in this scenario. We can move between desktops using keyboard shortcuts (Ctrl + Win + Right/Left Arrow). But with the VM running on separate Virtual Desktop any key presses gets picked up by the VM operating system and not by the host. This means that you cannot use the keyboard shortcuts to switch host desktops from inside a VM. However you can move between desktops using the Four Finger swipe gesture on your touchpad (if that is supported). These swipe gestures are picked up only by the host machine OS, unlike the keyboard shortcuts. So even when you are inside a VM, doing the four finger swipe gesture tells the host OS to switch desktops. This allows you to easily navigate between VM's running on different Virtual Desktops.

Hope this helps!