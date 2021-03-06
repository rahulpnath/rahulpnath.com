---
author: rahulpnath
comments: true
date: 2013-04-12 07:40:46+00:00
layout: post
slug: windows-phone-series-preloading-content
title: Windows Phone Series – Preloading Content
wordpress_id: 501
categories:
- .Net
- Windows Phone
- WP7
tags:
- preload data in apps
- t4 template windows phone
- t4 templates
- windows phone 7
- Windows Phone 8
- windows phone preload
- Windows Phone Series
- wp preloading
---

Mostly phone apps, connect to a service for the data and wrap them up to a cool UI for user consumption. But at times we would have apps that comes with a lot of preloaded content, with offline capability using [sqlite](http://visualstudiogallery.msdn.microsoft.com/cd120b42-30f4-446e-8287-45387a4f40b7).  Offline scenarios might either start of with preloaded content or get content on the apps first launch. This article is more for the scenario where we have preloaded content bundled with the app, and then on launch of the app, it would also get new content and updates to existing content. Complete knowledge of the pre-loaded content at the time of development would be rare. All you would know would be the metadata schema and how/where to look for the preloaded content.

We would not be creating a fully functional sample here, but would be touching on important aspects and the approaches that can be used to tackle similar scenarios and some code snippets

This [article](http://blogs.windows.com/windows_phone/b/wpdev/archive/2013/03/12/using-the-sqlite-database-engine-with-windows-phone-8-apps.aspx) here would help you out on setting up sqlite for windows phone apps. Mostly we would be storing metadata of files/media that gets preloaded in the sqlite and keep the original files/media(preloaded content) packaged along with the xap, ideally by setting the [Build Action to Content](http://msdn.microsoft.com/en-in/library/windowsphone/develop/ff967560\(v=vs.105\).aspx#BKMK_Media).  Now in cases where we are expecting to update the existing content and also get new content we would have to copy out the entire media/files into the IsolatedStorage, so that we can do any further updates or additions.

Assuming that we have a folder “MyPreloadedContent” as indicated in the image below, which would be where all our preloaded content is going to be . Most of the time with preloaded content, we would not know what exact data would be in there. It might contain files, images, folders etc. We would want an easy way to set Build Action to Content for all the files/folders that gets placed under the folder( even if it is done outside of Visual Studio).

![preloaded content Visual studio](/images/preloaded_content_Visual_studio.png)

For this we would need to tweek the project file, to tell it that whatever is under MyPreloadedContent should be treated as ‘Content’. Edit the csproj file from notepad or any other text editor that you use([Notepad++](http://notepad-plus-plus.org/) is my personal favorite). Scroll down to wherever the other Content files are specified, like “<Content Include="ApplicationIcon.png">” for example. Add in the below line to make all the content put into that folder to be treated as Content, and save the csproj file.

    
    <strong><span style="font-size:large;"><Content Include="MyPreloadedContent**" /></span></strong>


Now you could open that folder and put in some content into that. For now I put in some image files and also a sub folder  as below

![preloaded content explorer](/images/preloaded_content_explorer.png)

You would need to reload the projects/solution in visual studio to see that those files are automatically included into the solution. If not in a Visual studio,  msbuild would automatically include all the files as Content.

![preloaded content visual studio refresh](/images/preloaded_content_vs_refresh.png)

Now that we have all the files copied into that folder to be automatically included into the solution you need to now need to copy out all these files onto the IsolatedStorage when the app starts for the first time. You would want to do this, so that if there are any updates onto the files that you copied(which would be delivered to you via a web service in a real scenario), you can overwrite the files in the IsolatedStorage so that the new content would be taken then on.

Now comes the next challenge of getting all the files that are under MyPreloadedContent folder, so that you can copy them over to IsolatedStorage. Since this files would be copied at a later point of time, say at the time of packaging, we would not be able to know all the file names and directory structure prior.

We can use [T4 ( Text Templating Transformation Toolkit) templates](http://msdn.microsoft.com/en-us/library/vstudio/bb126445.aspx) to help us out here. Using T4 templates we can generate a class file that will have a property returning us all the file names in the MyPreloadedContent directory.

To create a T4 template, add a new item to the project **MyFiles.tt. **Select Ok if you get any warning message

![preloaded content t4 template](/images/preloaded_content_t4_template.jpg)

In a T4 template it would be a mix of text and code, that would be used to generate a new file that would be a class in our case which would expose a function to get all the file names under the folder . Below is the entire text/code that would go into the new file that we just created(**MyFiles.tt**)

``` csharp
    <#@ template debug="false" hostspecific="true" language="C#" #>
    <#@ output extension=".gen.cs" #>
    <#@ import namespace="System.IO"#>
    // <auto-generated />
    
    namespace PreloadedContent
    {
        public class MyFiles
        {
            private static string[] MyPreloadedContentFiles()
            {
                return new[] {
    <#
                DirectoryInfo directoryInfo = new DirectoryInfo(
                   Path.Combine(Path.GetDirectoryName(Host.TemplateFile),"MyPreloadedContent"));
            foreach(FileInfo file in directoryInfo.GetFiles("*.*", SearchOption.AllDirectories))
                {
                    if (!file.FullName.Contains(@"."))
                    {#>
                          "<#= file.FullName.Substring(
                          file.FullName.IndexOf("MyPreloadedContent")).Replace(@"", "/") #>",
    <#              }
                }
    #>
                            };
            }
        }
    }
```

It just says to read the directory MyPreloadedContent and iterate to get all the files in that and writes it out by trimming of the absolute path and putting in only the relative path. Save the MyFiles.tt, and in Visual Studio right click on it and say “Run Custom Tool”. This would generate the cs file with an extension “.gen.cs” as we have mentioned in “**<#@ output extension=".gen.cs" #>”.  **The generated class would look like below

``` csharp
    namespace PreloadedContent
    {
        public class MyFiles
        {
            private static string[] MyPreloadedContentFiles()
            {
                return new[] {
                               "MyPreloadedContent/picfinity login.png",
                               "MyPreloadedContent/Search.png",
                               "MyPreloadedContent/Share.png",
                               "MyPreloadedContent/upload.png",
                               "MyPreloadedContent/Profile/profile info.png",
                               "MyPreloadedContent/Profile/Profile.png",
                            };
            }
        }
    }

```
The above class has all the file paths to the content and you could iterate that to copy out the files into the IsolatedStorage.

There are a couple of ways, by which you can ensure that the template file is run before the actual code gets compiled. This is to make sure that this generated class is going to be updated with the latest files that would be copied into the folder at build time i.e your templates are transformed at build time. This [article](http://msdn.microsoft.com/en-us/library/ee847423.aspx) details out the methods to get that integrated into the build. On a build server where you dont have Visual Studio installed you would need to copy out these files mentioned [here](http://msdn.microsoft.com/en-us/library/ee847423.aspx#buildserver) explicitly

With that integrated you are all set to go to build your app with preloaded content. You would not need to know anything about the file names/ structure of the content and it would just work as long as the metadata that drives it correct.

Hope it helps!!
