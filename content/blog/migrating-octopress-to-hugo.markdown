---
title: "Migrating Octopress To Hugo"
comments: true
categories: 
- Blogging
tags: 
date: 2019-01-12
description: Migrated my blog again - Here's how I went about doing it.
primaryImage: bloggingSchedule_writedaily.jpg
cover: /images/bloggingSchedule_writedaily.jpg
---

I have been on [Octopress blogging platform for around 5 years](https://www.rahulpnath.com/blog/static-generator-is-all-a-blog-needs-moving-to-octopress/) and was fairly happy with it. I had [optimized Octopress workflow for new posts](https://www.rahulpnath.com/blog/optimizing-octopress-workflow-for-new-posts/), [set it up for continuous delivery](https://www.rahulpnath.com/blog/continuos-delivery-of-octopress-blog-using-travisci-and-docker/) and also [enabled scheduling posts in the future](https://www.rahulpnath.com/blog/automatic_deployment_of_future_posts_with_octopress/).

I have been wanting to migrate off Octopress since a while (reasons below) but have been putting it off since I did not want to go through [another migration pain](https://www.rahulpnath.com/blog/own-your-urls/). Now that it is all done, the migration was not as hard as I thought. In this post I will walk through the reasons of migrating away from Octopress, the actual migration steps involved and tweaking the default Hugo settings/theme and workflow to get what I wanted.

### Reasons To Migrate

- **No Longer Maintained:** [Octopress](http://octopress.org/) is no longer maintained by anyone and it's hard to keep up with all the dependent library updates and ruby version changes. I have my [builds breaking randomly](https://travis-ci.org/rahulpnath/rahulpnath.com-old/builds/470281590) for dependent package updates and it was not something I liked dealing with.

- **Terribly Slow:** To build by full site Octopress takes around 2 minutes. Since I have modified my workflow to [build only the draft posts when in local](https://www.rahulpnath.com/blog/optimizing-octopress-workflow-for-new-posts/) I usually don't have to wait that long, but still it's slow.

These two reasons were pressing enough to migrate off Octopress. Hugo was the natural choice for its speed and community and is the next highest rated after Jekyll(/Octopress). I also chose to migrate away from Azure Hosting and use Netlify to host this blog.

**Why Netlify?**
With Octopress I had my [build pipeline](https://www.rahulpnath.com/blog/continuos-delivery-of-octopress-blog-using-travisci-and-docker/) push the generated site contents back into GitHub (to a separate branch) and then have Azure deploy that branch automatically using Github trigger. If I were to remain on Azure, I would have to do almost the same. Netlify comes with a [hugo template](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/). The template is automatically detected when pointed to your repository and sets up all that is required to deploy the generated static content. All I had to update was to set the correct Build Environment Variable for HUGO_VERSION.

> [Netlify](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/) can host your Hugo site with CDN, continuous deployment, 1-click HTTPS, an admin GUI, and its own CLI.

I moved this site over to [HTTPS](https://www.rahulpnath.com/blog/ok-i-have-got-https-what-next/) a while back and had been using Cloudflare's Shared SSL. Moving over to the free [Let's Encrypt](https://letsencrypt.org/) certificate required [additional setup on Azure](https://letsencrypt.org/). Netlify takes out all this complexity and handles this all for you in the background. Once you set up a [custom domain](https://www.netlify.com/docs/custom-domains/), it's provisioned with a [Let's Encrypt](https://www.netlify.com/blog/2016/01/15/free-ssl-on-custom-domains/) certificate.


### Migration

The actual migration of the content was mostly related to moving all the files and fixing up some code blocks.

- **Move Files**
Moving files was easy, as both platforms support Markdown. Everything from your source folder maps into the content folder in Hugo. Pages that were in folders in Octopress are now Markdown files with the appropriate name in Hugo. The actual posts in Markdown had the date appended in them. Using a PowerShell script, I stripped off the first 11 characters (*YYYY-MM-DD-*) and moved them into the blog folder (since all my blog posts are under the /blog URL path). All my images live under the static folder.
![Octopress to Hugo - Files](/images/octopress_to_hugo_files.png)

- **Fixing Code Blocks**
Octopress supported adding a [custom title](http://octopress.org/docs/blogging/code/) on the code block which is not available out of the box in Hugo. You can use a [custom shortcode](https://github.com/parsiya/Hugo-Shortcodes/blob/master/README.MD#codecaption-codecaptionhtml) to set this up. However, I chose to remove them as there were only a few code blocks that had them. Using regex search in VSCode, it's easy to get rid of them at once. 

### Configuring Hugo

With all the content ported over all that was left was to select a theme a customize it, make sure all existing URLs work on the new site, configure search and a few other things.

After a bit of hunting around for [themes](https://themes.gohugo.io/), I decided to go with [Minimo](https://themes.gohugo.io/minimo/) for its simplicity and supporting most of the configurations of Hugo. All my theme overrides are in the [layouts folder](https://github.com/rahulpnath/rahulpnath.com/tree/master/layouts). I have added in support for showing a paged list of all my blog posts and a few layout changes for list views and headers. Added in a [google custom search engine support](https://cse.google.com/cse/all) as a [widget](https://github.com/rahulpnath/rahulpnath.com/blob/master/layouts/partials/widgets/search.html)and added it to the sidebar. Set up the [404 custom page](https://github.com/rahulpnath/rahulpnath.com/blob/master/layouts/404.html) to enable searching site for content.

The site is now running on Hugo + Netlify. Building the whole site (not just the drafts) takes around 3-4 seconds (cold build) and on every file change after the build watcher is running takes around 200 milliseconds. Hugo is blazing fast. If you face any issues, have any feedback kindly drop a comment or send me a [tweet](https://twitter.com/rahulpnath).