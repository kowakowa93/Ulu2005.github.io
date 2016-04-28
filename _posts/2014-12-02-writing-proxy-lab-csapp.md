---
id: 92
title: Writing Proxy Lab for CSAPP
date: 2014-12-02T11:04:00+00:00
author: Ke
layout: post
guid: http://www.keblog.me/?p=92
permalink: /2014/12/writing-proxy-lab-csapp/
comments: true
dsq_thread_id:
  - 3283423089
categories:
  - programming
tags:
  - csapp
  - network
  - proxy
---
In contrast with previous malloc lab, proxy lab is not that hard in CSAPP. But proxy lab could be more complicated if you intend to make it robust. I met many weird problems when writing this lab. As a rookie to network programming, I cannot explain everything well, but I am able to offer some measures to solve problems.

Generally, writing a proxy at proxy lab level is relatively simple in algorithm. You proxy should act as a server when connecting to a client and act as a client when connecting to remote web servers. You may get some inspiration by reading source code of Tiny Proxy, which is included in proxy lab handout.

Like my previous articles about CSAPP, I will talk about some gists only.

<!--more-->

## 1. Use Robust IO function wrapper wisely

In 3/e version, which is the latest version, of csapp.c many functions, such as _openclientfd,_ are thread-safe, thus you should definitely use the thread-safe ones. If not, please make it thread-safe.

Generally, Rio functions wrapper call _unix_error_ when error happens. But _unix_error_ calls _exit(0)_ after error message is printed. It will make your proxy crash when broken pipe is being used. It’s better to comment _exit(0)_ out and leave program return. Calling a thread-level exit, we say, _pthread_exit(NULL)_, may have some unexpected results. I would not recommend you doing so.

## 2. Signal

In this lab, lots of things were done for robustness. Signal handlling is one of them.

When sending data to a closed socket connection twice, the system kernal will raise a SIGPIPE signal. The default action of SIGPIPE handler is terminating the process. It’s obviously not what we want. So we can block it by using

<pre class="lang:c decode:true ">signal(SIGPIPE, SIG_IGN);</pre>

Note that SIG_IGN specifies that the signal should be ignored.

It’s also possible to see SIGSEGV (I think everyone takes this course should be familiar with this signal =3= ) when running proxy. I’m not sure why this happens because when SIGSEGV is caught in GDB, _backtrace_ command offered me nothing useful for debugging. SIGSEGV would make proxy crash either, if you simply ignores it. What you should do is to write a SIGSEGV handler to catch it, just like what you have done in shell lab. For me, I called _pthread_exit(0)_ in handler and luckily it works.

## 3. Use reader/writer lock in pthread

As required, our proxy should be concurrent and with cache. For reading a cache, it’s mostly a reader-writer problem. You may implement your thread model by using codes in course lecture. But it’s not very convenient since it maintains two semaphore variables.

Actually, in pthread library, there are several functions prepared for reader-writers problem. They are named with prefix “pthread_rwlock”. First let’s have a look at some function prototypes.

{% highlight c %}
#include "pthread.h"
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,
const pthread_rwlockattr_t *restrict attr);
{% endhighlight %}

From man page you can see this:

The _pthread_rwlock_init()_ function shall allocate any resources required to use the read-write lock referenced by rwlock and initializes the lock to an unlocked state with attributes referenced by attr. If attr is NULL, the default read-write lock attributes shall be used; the effect is the same as passing the address of a default read-write lock attributes object. Once initialized, the lock can be used any number of times without being reinitialized. Results are undefined if _pthread_rwlock_init()_ is called specifying an already initialized read-write lock. Results are undefined if a read-write lock is used without first being initialized.

{% highlight c %}
#include "pthread.h"
#include "pthread.h"
/* Apply a read lock */
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);

/* Apply a write lock */
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);

/* releases a lock held on the read-write lock object referenced by rwlock. */
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
{% endhighlight %}

The usage should be clear when you see these prototypes. One useful usage is that use _trywrlock_ when read thread modifies cache timestamp. In this way, rather than block there, the function will return instantly if the rwlock is currently held by other thread. Note that in lab requirement it’s not necessary to follow strict LRU strategy, and there is nearly no difference in access time for concurrent read threads. So use _trywrlock_ would be fine.

You may find more infomation about pthread rwlock <a href="https://docs.oracle.com/cd/E19455-01/806-5257/6je9h032u/index.html" target="_blank">here</a>.

## 4. Handle file descriptor leakage

If your proxy do not reclaim all allocated file descriptors, there would be a file descriptor leakage. You can test it by visiting http://www.amazon.com/ several times.

Then use _“lsof -a -p pid”_ command to check the leakage.

[<img class="aligncenter wp-image-93 size-large" src="http://www.keblog.me/wp-content/uploads/2014/12/lsof_command-1024x394.png" alt="proxy lab" width="584" height="224" srcset="//www.keblog.me/wp-content/uploads/2014/12/lsof_command-1024x394.png 1024w, //www.keblog.me/wp-content/uploads/2014/12/lsof_command-300x115.png 300w, //www.keblog.me/wp-content/uploads/2014/12/lsof_command-500x192.png 500w, //www.keblog.me/wp-content/uploads/2014/12/lsof_command.png 1412w" sizes="(max-width: 584px) 100vw, 584px" />](http://www.keblog.me/wp-content/uploads/2014/12/lsof_command.png)At bottom, it lists all file descriptors allocated for proxy connection. If everything goes well, there should be a few connections here. You can see my proxy is listening on port 23884, the connection file descriptor is 3 while 0, 1, 2 are reserved for system.

## 5. Compilation flags

If your proxy met some weird problems and you enabled optimizaion flag, such as -O2, in makefile, you may need to delete them. In my situation, when I set compile flag with -O2 or -O1, GCC would optimize out some important pointers. Then segmentation fault happens when dereferencing these pointers.

If optimization is necessary for you, remember to declare your pointer with keyword volatile like below:

<pre class="lang:c decode:true ">volatile int *args;</pre>

Volatile tells the compiler not to optimize anything that has to do with the volatile variable.

=================================================================

## Test Proxy With Netcat

As my friend wonders how to verify request headers were sent correctly, here I add a short tutorial about how to use _netcat_.

1. Get a free port on shark machine.

Use free-port.sh to get a random free port. Here I get 4500.

2. Start netcat as a server listening on port you get.

The command should be &#8220;nc -l <port>&#8221;.

3. Send request by curl

Type in command as follow:
{% highlight shell %}
curl -v --proxy http://localhost:23884/ http://localhost:4500/
{% endhighlight %}

The request generated by curl is output on screen. Notice that it&#8217;s different from what netcat received.

[<img class="aligncenter size-large wp-image-111" src="http://www.keblog.me/wp-content/uploads/2014/12/curl_proxy_lab-1024x225.png" alt="proxy lab" width="584" height="128" srcset="//www.keblog.me/wp-content/uploads/2014/12/curl_proxy_lab-1024x225.png 1024w, //www.keblog.me/wp-content/uploads/2014/12/curl_proxy_lab-300x66.png 300w, //www.keblog.me/wp-content/uploads/2014/12/curl_proxy_lab-500x110.png 500w" sizes="(max-width: 584px) 100vw, 584px" />](http://www.keblog.me/wp-content/uploads/2014/12/curl_proxy_lab.png)Here I output request generated by my proxy.

[<img class="aligncenter  wp-image-112" src="http://www.keblog.me/wp-content/uploads/2014/12/proxy_proxy_lab.png" alt="proxy lab" width="346" height="59" />](http://www.keblog.me/wp-content/uploads/2014/12/proxy_proxy_lab.png)4. Check results

Check request forwarded by proxy on terminal which runs netcat.

[<img class="aligncenter size-large wp-image-110" src="http://www.keblog.me/wp-content/uploads/2014/12/nc_proxy_lab-1024x373.png" alt="nc_proxy_lab" width="584" height="212" srcset="//www.keblog.me/wp-content/uploads/2014/12/nc_proxy_lab-1024x373.png 1024w, //www.keblog.me/wp-content/uploads/2014/12/nc_proxy_lab-300x109.png 300w, //www.keblog.me/wp-content/uploads/2014/12/nc_proxy_lab-500x182.png 500w, //www.keblog.me/wp-content/uploads/2014/12/nc_proxy_lab.png 1201w" sizes="(max-width: 584px) 100vw, 584px" />](http://www.keblog.me/wp-content/uploads/2014/12/nc_proxy_lab.png)
