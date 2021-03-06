---
layout: post
title: "Organizing Tests into Test Suites for Visual Studio"
comments: true
categories:
- Testing
- Productivity 
tags: 
date: 2016-01-18 22:51:03 
keywords: 
description: 
---

While working with large code base, that has a lot of tests (unit, integration, acceptance etc), running all of them every time we make a small  change (if you are doing TDD or just using build for feedback) takes a lot of time. Organizing tests into different test suites, making it easier to run as required by the current context, is handy in such cases. 

There are multiple ways that we can do this within Visual Studio and below are some of the options available. I tend to use a mix of all these in my current project. This gives the flexibility to run only the new tests that I am writing while writing new code or set of related tests for the updates that I am making. Once done with the changes, I can run the full suite of unit tests, followed by the integration tests. This reduces the interruption duration while coding and has a direct impact on the overall productivity too. (If you think small interruptions does not matter much think twice!)

<img class="center" alt="Geek productivity" src="/images/geek_productivity.jpg" />

#### **Test Traits** ####

Traits are a good way to group tests together and to run them as different suites. It encompasses TestCategory, TestProperty, Priority and Owner. Using [TestCategory](https://msdn.microsoft.com/en-au/library/microsoft.visualstudio.testtools.unittesting.testcategoryattribute.aspx) attribute we can specify  the group of the test and the Visual Studio Test Explorer uses this value to group the tests and allows executing tests in specific groups.

<img class="center" alt="Visual Studio Test Traits" width="75%" src="/images/vs_testExplorer_traits.png" />

Limitation with the above approach is that it depends on developers to put these attributes on the test cases or class level and not leveraging any existing conventions that might be already in place. Having integration tests, unit tests, acceptance tests in different projects is a very common practice, with conventions like project names ending with '.UnitTests, .IntegrationTests, .AcceptanceTests' etc. 

#### **Build Tasks and Task Runner Explorer** ####

The [Task Runner Explorer](https://visualstudiogallery.msdn.microsoft.com/8e1b4368-4afb-467a-bc13-9650572db708) (TRE) provides custom task runner support to Visual Studio, allowing to run grunt/gulp task or target inside Visual Studio. Grunt/Gulp has packages for most of the unit testing frameworks, using which different build tasks can be created. To select the tests to execute different conventions can also be used. Below is an example of a gulp task to execute all the c# unit tests in the project.

``` javascript
var gulp = require('gulp');
var xunit = require('gulp-xunit-runner');
var xunitConsolePath = 'xunit.console.exe';
var unitTestsConvention = ['**/*.Tests.dll'];

gulp.task('c#UnitTests', function () {
    runTests(unitTestsConvention);
});

function runTests(dllPath) {
    return gulp.src(dllPath, { read: false })
        .pipe(xunit({
            executable: xunitConsolePath,
            options: { parallel: 'all' }
        }));
}
```
> *You need to install [Node Package Manager](https://www.npmjs.com/package/npm) and grunt/gulp npm packages for TRE.*
  
Similarly we can have multiple tasks to execute different groups of tests and it will be available in the TRE within Visual Studio as shown below. This approach gives the most flexibility, allowing tests be grouped any way and providing ability to execute tests across the stack of technologies.
<img class="center" alt="Visual Studio Task Runner Explorer" src="/images/vs_tre.png" />

#### **Tests Settings File** ####
Creating Test Playlist is an easy way to group tests into a playlist and executing them as  group. From the Test Explorer, select the tests to be grouped and on right-click, the option to create playlist is available. The saved playlists can be selected from the drop down menu on the top bar for later execution.   
   
<img class="center" alt="Visual Studio Test Playlist" src="/images/vs_testExplorer_playlist.png" />

This works well for short-lived groupings, when we are actively working on a part of the code and need to execute tests for that area. Every time a new test is added, we need to add it explicitly to the playlist if required. 

We have seen multiple ways of grouping tests into test suites, and each of them comes handy in different situations. For project wide convention tests, I tend to use build tasks that integrate with TRE as it is more flexible and extendable. Do you use any other ways to group your tests, drop in with a comment!