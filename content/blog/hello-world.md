---
title: "Creating a static page in Hugo"
date: 2022-08-13T00:24:56+02:00
draft: false
tags: ['en']
---

Recently, i decided to remake my homepage and migrato to Jamstack page generator. I'm already using Jekyll on my blog, and was thinking of moving to Jamstack on my homepage for some time, but i was not sure which page generator to pick.  
I was pretty happy with my Jekyll setup, but due to lack of knowledge about CI/CD at the time and ignoring the fact, that using Git for storing static blog content is actually pretty convinient I didn't really created any new content.

I also like to change a bit my homepage ones every so often, because i become bored of it, or I do want to try new tools on semi-prod scale.

My previous website was a hand written HTML + Bootstrap 4 CSS that was just used for grid.

Yes, I used Bootstrap pretty much only for grid, please keep in mind - i'm not a frontend developer, I just want to put out a website that looks somewhat decent without tinkering a lot with CSS. However, on current setup i added PurgeCSS, which cuts of unused CSS, so i don't waste bandwidth. On my previous website i also used PurgeCSS but manually on every change, instead of automatic process while building the website.
For this, i just followed [this guide](https://purgecss.com/guides/hugo.html).

Someone could ask - if i want to use CMS why just not install Wordpress and call it a day? Well that is a option, but i don't need all the features of Wordpress and i prefer to have a static page.

Yes - Wordpress can be also used as a headless CMS and render content to static form, but i prefer to use tools designed for that.

Static pages have a very big adventages - they are very easy to migrate - in minutes i can deploy my website on another hosting, netlify, heroku or even in IPFS. Because there is nothing to compile and no database to call, the bottleneck of serving the website most likely would be my server network connection instead of database or CPU. But I doubt that I ever would have that issue.

Of course, Jamstack is not a solution for every problem, but for a simple blog it works great.

I decided to try Hugo - i heart about it before, but after watching video from Fireship "Hugo in 100 Seconds" i decided to give it a try  
{{< youtube 0RKpf3rK57I >}}

For mainpage i was always a fan of simple site with general information and links, so i just created simple layout. As for blog i did used theme from panr - [terminal](https://github.com/panr/hugo-theme-terminal), it is pretty popular and liked theme in the Hugo community, and you probably seen it already somewhere else.

