---
author: rahulpnath
comments: true
date: 2010-01-27 15:57:00+00:00
layout: post
slug: the-building-block
title: The Building block
wordpress_id: 15
categories:
- WF
tags:
- WF
---

Since this comes under the 'WF' tag it would not be difficult for anyone to understand what I am talking about.  
Yes its 'Activity'.  
Activity is nothing but a piece of re-usable component performing a specified task.When I say re-usable it means across multiple workflows and thats where the catch is.Write Once ,Test Once and there you have something which works fine wherever put into.  
            WF ships with many activities,a list of which you will find [here](http://msdn.microsoft.com/en-us/library/ms733615.aspx).The real power of WF is not in the out-of-the-box activities it ships with..it lies within you..Custom Activities.  
  
My first question to such a statement would be,WHY?  
WF is nothing but organizing activities in a logical manner addressing the requirement at stake.So the more of custom built activities you have the more easier and faster it is to address your needs.It all becomes the drag'n'drop funda microsoft boasts about and the reason why more developer turn towards it.The same reason why a GUI developer would go for a custom control rather than tweaking existing controls in every page he needs a similar look'n'feel.  
  
Activities are of 2 type  
1. Basic/Simple Activity  
2. Composite Activity  
  
Put everything in the code activity(the best example for a simple activity) and get done with the work assigned would be another approach amongst us.A definite YES keeping in mind the short term task at hand.But a big NO on the long run.  
Code Activities ends within the current workflow.Re-usability across workflows is the key factor that is at stake in this approach,which should makes us think twice before using one.CodeActivities,mostly would be fully dependent on the instance properties of the workflow,which makes testing of something performed within the activity require the full workflow to be executed.  
On the other hand a Custom Activity has its own properties on which it depends,can be tested individually and above all can be re-used across workflows.  
  
So the next time you drag'n'drop a Code Activity from the toolbox think twice :)  
  
Will catch you on developing a custom activity soon :)  
[CodeProject](http://www.codeproject.com/script/Articles/BlogFeedList.aspx?amid=5842203)
