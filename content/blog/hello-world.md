---
title: "Creating a static page in Hugo"
date: 2022-08-13T00:24:56+02:00
draft: false
tags: ['en']
---

Recently, i decided to remake my homepage and move to somekind of jamstack page generator. I'm already using Jekyll on my blog, and was thinking of moving to jamstack on my homepage for some time, but i was not sure which page generator to pick.  
I was pretty happy with my Jekyll setup, but due to lack of knowledge about CI/CD at the time and using multiple computers to write posts, i found it pretty unconvinent to create content.

I also like to change a bit my homepage ones every so often, because i become bored of it, or I do want to start using newer things to build it.

My previous website was just a hand written HTML + Bootstrap 4 CSS that was just used for grid.

Yes, I used Bootstrap pretty much only for grid, please keep in mind - i'm not a frontend developer, I just want to put out a website that looks somewhat decent without tinkering a lot with CSS. However, on current setup i added PurgeCSS, which cuts of unused CSS, so i don't waste bandwidth. On my previous website i also used PurgeCSS but manually.
For this, i just followed [this guide](https://purgecss.com/guides/hugo.html).

Someone could ask - if i want to use CMS why just not install Wordpress and call it a day? Well that is a option, but i don't need all the features of Wordpress and i prefer to have a static page.

Yes - Wordpress can be also used as a headless CMS and render content to static form, but i prefer to use tools designed for that.

Static pages have a very big adventages - they are very easy to migrate - in minutes i can deploy my website on another hosting, netlify, heroku or even in IPFS. Because there is nothing to compile and no database to call, the traffic bottleneck most likely would be my server network connection instead of database. But I doubt that I ever would have that issue.