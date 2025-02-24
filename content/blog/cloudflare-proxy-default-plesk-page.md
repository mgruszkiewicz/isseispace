---
title: "Plesk default page instead of domain content after enabling Cloudflare proxy"
date: 2025-02-24T17:13:47+01:00
draft: false
cover: "images/2025-02-24-cloudflare-proxy-plesk/cover.png"
tags: ['en', 'cloudflare']
---
Recently i faced an interesting issue - after enabling Cloudflare proxy on domain hosted on Plesk (with Litespeed webserver, but that is not important), the domain returned Plesk default page instead of the domain, even through in domain logs i could see the requests.

<!--more-->
The solution was to enable "Full" SSL/TLS encryption in domain settings on Cloudflare. (on that domain at first it was set to 'Flexible')
![screenshot showing cloudflare dashboard, with 'SSL/TLS Overview' section highlighted](images/2025-02-24-cloudflare-proxy-plesk/Screenshot_20250224_171838.png)
