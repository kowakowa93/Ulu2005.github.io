---
id: 218
title: Under XML-RPC attack recently
date: 2016-04-18T18:34:54+00:00
author: Ke
layout: post
guid: http://www.keblog.me/?p=218
permalink: /2016/04/under-xml-rpc-attack-recently/
comments: true
dsq_thread_id:
  - 4763920628
categories:
  - programming
---
The site is being down frequently in recent two months. At first I was too lazy to fix it, so I received tons of alert emails from NewRelic which is quite annoying. From NewRelic analysis page, the site was having too much traffic with POST request to xml-rpc.php file and then my database eat up all memory. As a result, site went down. This is a typical wordpress xml-rpc attack.

With the help of this <a href="https://www.digitalocean.com/community/tutorials/how-to-protect-wordpress-from-xml-rpc-attacks-on-ubuntu-14-04" target="_blank">article</a>, I&#8217;ve done the modification to my configuration files. Hopefully the site is more robust from now on.
