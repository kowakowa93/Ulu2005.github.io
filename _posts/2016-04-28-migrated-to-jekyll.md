---
title: Site Migrated to Jekyll
date: 2016-04-28
author: Ke
layout: post
permalink: /2016/04/migrated-to-jekyll/
comments: true
categories:
  - programming
---

Finally I migrated from Wordpress to Jekyll. The main reason is performance and security. And the site is now hosted on Github pages, this saves me both money and time. I really don't like wasting time in tuning MYSQL or Apache due to the limited resource on DigitalOcean droplet(I chose the cheapest one).

Although the site looks primitive now, I'm not sure I will improve it. Front-end stuff is not interesting for me and current one is just acceptable.

For migration process, I've done following things:

1. Export all posts in Wordpress by plugin.
2. Set up Jekyll site locally and import all posts.
3. Fix format and image issues in imported posts.
4. Migrate comments in Disqus.
5. Deploy the site to Github pages.

<!--more-->
To be honest, Jekyll's document is not satisfying. But luckily Jekyll itself is simple, the overall migration process is pleasant.
