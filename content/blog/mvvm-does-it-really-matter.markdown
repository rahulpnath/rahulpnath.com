---
author: rahulpnath
comments: true
date: 2013-04-08 05:12:07+00:00
layout: post
slug: mvvm-does-it-really-matter
title: MVVM – Does it really matter?
wordpress_id: 484
categories:
- .Net
- Thoughts
- Windows 8
- Windows Phone
- WP7
- WPF
tags:
- MVVM
- WIndows 8
- Windows 8 Series
- windows phone 7
- Windows Phone 8
- Windows Phone Series
- WPF
---

MVVM (Model-View-ViewModel), is a popular architectural pattern since WPF/Silverlight. Separation of concerns(UI/code), testability etc are some of the key things that motivates one to go via the MVVM route. There are innumerous articles out there, just like this [one](http://msdn.microsoft.com/en-in/library/hh848246.aspx), that gets into the details of how and why one should use MVVM.

With Windows phone also embracing xaml and silverlight, any one who knew silverlight turned a phone developer overnight. MVVM did find its way into this space too. But most of the phone app developers, unlike those who developed for enterprise. would have never cared for MVVM , as they rarely would have written test cases for their apps, nor were they actually concerned on the UI/code separation. Since most of the apps were just out of a hobby, the only idea was just to have it up and available in the store as fast as possible. I might not be fully correct here, but I do know at least a dozen people,including me, who did this, so am good enough to put out that statement

With windows 8 too taking the store way and having the same development platform of silverlight/xaml, don’t be surprised MVVM  showed up there too. Now anyone who had an app on the phone, had to do a lot of copy pasting over the code to have the same application available in both the stores. This gives MVVM a totally new dimension for motivation that was not spoken  about earlier – **_Reusability. _**

Having an application for phone and windows 8 app store with the minimum amount of rework is best possible by using MVVM and also a couple of other techniques. There is a detailed [article](http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj681693(v=vs.105).aspx) on msdn on how how to maximize code reuse between Windows Phone 8 and Windows 8.

MVVM does really matter now, if we do not want to end up copy pasting code from phone app to the windows 8 store app. Also fixing and adding in new features would become more easier with following MVVM

[MVVM Toolkit](http://nuget.org/packages/Portable.MvvmLightLibs/) is a very popular helper library for implementing MVVM pattern, as is available on nuget as a PCL(Portable class library)

**MVVM_, It really does matter !!!_**
