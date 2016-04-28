---
id: 177
title: Why Bare Git Repository
date: 2015-01-17T02:28:47+00:00
author: Ke
layout: post
guid: http://www.keblog.me/?p=177
permalink: /2015/01/bare-git-repository/
comments: true 
dsq_thread_id:
  - 3428606196
categories:
  - programming
tags:
  - git
---
For homework in 18-645(How to write fast code), I’m required to set up a bare git repository with command

<pre class="lang:sh decode:true ">$ git init --bare fastcode</pre>

Noticing the flag “&#8211;bare”, I wonder why we need this flag. After some searching, now it’s clear to me.

## What a git repository contains

A normal git repository is a directory that contains

  * Project directories and files (so called “working tree” or “working directory” ).
  * A single directory called .git containing all of git&#8217;s administrative and control files.

As image below shows, we work on codes in working directory then “git add” changed files to staging area. Finally, we &#8220;git commit&#8221; to local repositoy.

<!--more-->



[<img class="aligncenter size-full wp-image-178" src="http://www.keblog.me/wp-content/uploads/2015/01/git_git_add.png" alt="git_add_commit" width="336" height="194" srcset="//www.keblog.me/wp-content/uploads/2015/01/git_git_add.png 336w, //www.keblog.me/wp-content/uploads/2015/01/git_git_add-300x173.png 300w" sizes="(max-width: 336px) 100vw, 336px" />](http://www.keblog.me/wp-content/uploads/2015/01/git_git_add.png)

However, a bare repository does not contain working directory. It contains entire .git directory itself. Thus it’s not allowed to use “git add” or “git commit” in a bare repository directory. It’s created with “&#8211;bare” flag and conventionally named with suffix “.git”. For example, the bare version of git repository “repo” should be named “repo.git”.

## When to use a bare repository

In short, when we want to share a repository.

A bare repository is mostly used as a central repository which receive push and accept being cloned or pulled. Why don’t we use a non-bare repository directly? Because pushing branches to a non-bare repository has the potential to overwrite changes.

A typical senario would be like this:

On machine A,

<pre class="lang:sh decode:true ">$ cd ~/repo/
$ git init
$ git add a.file
$ git commit -m a</pre>

Then on machine B,

<pre class="lang:sh decode:true ">$ git clone repo repo_b
$ cd repo_b
$ git add b.file
$ git commit -m b
$ git push</pre>

From code above, commit made on machine B is pushed to original repo on machine A. Now, if we “git status” on machine A, what will happen?

Actually, it would respond

<pre class="lang:default decode:true "># On branch master
# Changes to be committed:
# (use "git reset HEAD &lt;file&gt;..." to unstage)
#
#     deleted: b.file</pre>

Why? Because git compares files in working directory with information from local .git directory, which is assumed to be always right. For here, on machine A, with newly pushed commit, .git directory says that there should be two files a.file and b.file. But in A working directory only a.file exists. Then git concluded that you deleted b.file.

Making similiar changes on machine B will always get ‘opposite’ results on machine A.

This kind of confusion can be avoided by using a bare repository. As bare repository doesn’t have a working directory, it doesn’t do comparision with working directory. It simplely records new commits and branch when you do push, or give you latest version of code when pull or clone.

## Conclusion

Shared repositories should always be created with the &#8220;&#8211;bare&#8221; flag.

In fact, with latest version of git, the typical senario would not be repeatable. Because git sets configuration variable “receive.denyCurrentBranch” with “refuse” as default. Error message would be printed when you try to recur this example.

For repositories on GitHub server, I guess they should be bare. But their web interface has done lots of magic work to make it look like both bare and non-bare. Related discussion can be found on stackoverflow.

## Reference

<http://www.saintsjd.com/2011/01/what-is-a-bare-git-repository/>

<https://www.atlassian.com/git/tutorials/setting-up-a-repository/git-init/>

<http://stackoverflow.com/questions/20854879/whats-the-difference-between-github-repository-and-git-bare-repository>
