---
layout: post
title: "Tip of the Week: Prettier - An Opinionated Code Formatter"
comments: true
categories: 
- TipOW
tags: 
date: 2018-06-04
completedDate: 2018-06-04 16:09:06 +1000
keywords: 
description: Format your code fast, easy and consistent.
primaryImage: prettier.png
---

Code Formatting is an essential aspect of writing code, and I did write about this a while back on [introducing code formatting into a large code base](https://rahulpnath.com/blog/introducing-code-formatting-into-a-large-codebase/). It's not about what all rules you and your team use, it's about sticking with the conventions and using them consistently. Code Formatting rules are best when applied automatically, and the developer does not need to do anything in particular about it.

[Prettier](https://prettier.io/) is an opinionated code formatter, which supports multiple languages and editors and easy to get started. Getting set up is as easy as just installing the [prettier package using yarn/npm](https://prettier.io/docs/en/install.html). There are multiple points at which you can integrate Prettier - [in your editor](https://prettier.io/docs/en/editors.html), [pre-commit hook](https://prettier.io/docs/en/precommit.html) or [CI environments](https://prettier.io/docs/en/cli.html#list-different).

<div style="text-align: center;">
<iframe width="560" height="315" src="https://www.youtube.com/embed/zgWBAKZvdFQ" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>

Most of the IDE's have plugins for Prettier which makes it easy to get it into the code right from the beginning. You might need to update your IDE settings to run prettier when you save a file. For VS Code I have to set editor.formatOnSave to true to turn on this behaviour.

As the title says, Prettier is opinionated, which is useful in many ways and removes much time wasted on unnecessary discussions. However, it does provide some [configuration options](https://prettier.io/docs/en/configuration.html). Check if it provides enough for you to call a meeting to decide on one of them :).

Write prettier code!
