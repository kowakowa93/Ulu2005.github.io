---
id: 32
title: Writing Shell Lab for CSAPP
date: 2014-10-29T00:35:07+00:00
author: Ke
layout: post
guid: http://www.keblog.me/?p=32
permalink: /2014/10/csapp-shell-lab/
comments: true
dsq_thread_id:
  - 3167737452
dsq_needs_sync:
  - 1
categories:
  - programming
tags:
  - csapp
  - shell
---
Shell Lab is the 1st lab after midterm exam of 15213/18213 (Introduction to computer system), and it's more complicated than previous labs. The most annoying part about this lab must be race condition. I spent over two days on debugging a concealed race condition. It's such a disgusting bug that TA and me checked code line by line whole night to fix it.

So here are some points should take care.

# Avoid Race Condition

Several things you should do to avoid race condition.

<!--more-->


+ Always unblock signal at end of your code block.

I mean, block signals unless you have already finished all the work in that part. For fork( ) part, as in child process, you should unblock signal before exeve( ). Because child process inherits parent's signal block vector, it's necessary to unblock signal before exeve( ) is called. Otherwise, this child process can never receive signals you blocked. We need child process to receive signals like SIGINT and SIGTSTPP.

+ Use async-signal-safe function.

For example, do not use printf( ) from standard library. In lecture notes professor gave a good example of async-signal-safe printf( ). You can directly use it in your code with including headfile &#8220;stdarg.h&#8221;.
{% highlight c %}
/* safe_printf – async-signal-safe wrapper for printf */
void safe_printf(const char *format, ...)
{
    char buf[MAXBUF];
    va_list args;
    sigset_t mask, prev_mask;
    sigfillset(&mask);
    sigprocmask(SIG_BLOCK, &mask, &prev_mask);
    va_start(args, format);
    vsnprintf(buf, sizeof(buf), format, args);
    va_end(args);
    write(1, buf, strlen(buf));
    sigprocmask(SIG_SETMASK, &prev_mask, NULL);
}
{% endhighlight %}

+ Use sigsuspend( ) properly.

It used to be allowed to use sleep( ) to keep parent process waiting for foreground job in waitfg( ). But since 2013 Spring, this method is abandoned and using sigsuspend( ) is designated. Sleep( ) is resource-consuming and slow. Rather, sigsuspend( ) is graceful and more smart. It suspends the process untill it receives desired signals. The prototype of sigsuspend( ) is showed below.

<pre class="lang:c decode:true" title="sigsuspend()">int sigsuspend(const sigset_t *mask);</pre>

The tricky thing about this function is that sigsuspend() temporarily replaces the signal mask of the calling process with the mask given by mask and then suspends the process until delivery of a signal whose action is to invoke a signal handler or to terminate a process. Take care about the difference between sigsuspend( ) and sigprocmask( ). If you want to receive SIGCHLD here, you should not add SIGCHLD into mask.

Thus, to use it in this lab, you should call sigsuspend( ) with an empty signal set mask. In this way, the parent can receive any signal and passes control to the related signal handler. After handler returns, the waiting loop continues. This would help your parent process reaps foreground job properly.

# General Problems

+ When message cannot output well.

Firstly, make sure that you used async-signal-safe version of printf( ). Secondly, make sure it's exactly called by program. Thirdly, try adding fflush(stdout) after printf( ).

+ Understanding ./mytstpp

This test program sends SIGTSTP to its parent process. The parent's SIGTSTP handler is triggered then and the handler sends SIGTSTP back to program. Thus program is stopped and SIGCHLD is sent to parent by system. After that, parent's SIGCHLD handler would reap this program and continues. If this job is foreground, recall that parent process uses sigsuspend( ) to wait for the signal. If SIGCHLD or SIGTSTP is blocked by mask in sigsuspend( ), the parent process will hang in the waiting loop forever.

<div id="attachment_54" style="width: 359px" class="wp-caption aligncenter">
  <a href="http://162.243.149.86/wp-content/uploads/2014/10/mytstpp.png"><img class="size-full wp-image-54" src="http://www.keblog.me/wp-content/uploads/2014/10/mytstpp.png" alt="shell lab" width="349" height="473" srcset="//www.keblog.me/wp-content/uploads/2014/10/mytstpp.png 349w, //www.keblog.me/wp-content/uploads/2014/10/mytstpp-221x300.png 221w" sizes="(max-width: 349px) 100vw, 349px" /></a>

  <p class="wp-caption-text">
    mytstpp flowchart
  </p>
</div>

# Debugging With GDB

For this lab, gdb is really helpful. To debug a multi-process program, you need a gdb with version above 7.0. Luckily, the version of gdb which is installed on CMU shark machines is 7.2. So just type in commands below and start debugging.

The first command is

<pre class="lang:default decode:true ">(gdb) set detach-on-fork on/off</pre>

This command sets whether child process will be detached when fork( ) is called. The default value is on. Thus child process would run without any interuption until it normally ends. If you set it as off, the child would be suspended when it's forked from parent process.

The second command is

<pre class="lang:default decode:true ">(gdb) set follow-fork-mode parent/child</pre>

This command decides which process will your gdb stays in after fork( ) is called.

To switch between different processes, you need command

<pre class="lang:default decode:true ">(gdb) info inferiors
  Num  Description       Executable
* 1    &lt;null&gt;</pre>

Different process has different number, when you type

<pre class="lang:default decode:true ">(gdb) inferior 1
[Switching to inferior 1 [process 0] (&lt;noexec&gt;)]</pre>

Your gdb would be switched to process represented by that number.

For more details about debugging shell lab, watch this <a href="http://vimeo.com/9592500" target="_blank">video</a> would help a lot.

Finally, good luck on your shell lab.
