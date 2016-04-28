---
id: 139
title: Fix UltiSnips Snippets Load Problem
date: 2015-01-08T14:15:11+00:00
author: Ke
layout: post
guid: http://www.keblog.me/?p=139
permalink: /2015/01/fix-ultisnips-snippets-load-problem/
comments: true 
dsq_thread_id:
  - 3402546099
categories:
  - programming
tags:
  - vim
---
To better use YouCompleteMe in Vim, I switched from SnipMate to UltiSnips, because UltiSnips can be integrated into YCM. I also installed snippets since UltiSnips offers an engine only.

<pre class="lang:vim decode:true">Plugin 'SirVer/ultisnips'
Plugin 'honza/vim-snippets'</pre>

But after I installed it with Vundle, it didn&#8217;t work correctly. It only load &#8216;all&#8217; filetype snippets. You can check this by typing

<pre class="lang:vim decode:true ">:UltiSnipsEdit</pre>

in Vim. If only &#8216;all.snippets&#8217; appears, then here is the problem.

<!--more-->

Then I searched internet, finally I got answer below. The problem is caused by Vundle. Vundle installed UltiSnips has a different file directory structure, thus file type detection is disabled. To fix this, we need to build soft link to proper directories.

<pre class="lang:sh decode:true">mkdir -p ~/.vim/after/plugin
ln -s ~/.vim/bundle/UltiSnips/after/plugin/* ~/.vim/after/plugin
mkdir ~/.vim/ftdetect
ln -s ~/.vim/bundle/UltiSnips/ftdetect/* ~/.vim/ftdetect</pre>

&nbsp;

&nbsp;

Reference:

<https://bugs.launchpad.net/ultisnips/+bug/1067416>
