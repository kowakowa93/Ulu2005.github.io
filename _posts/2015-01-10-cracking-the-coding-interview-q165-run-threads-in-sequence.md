---
id: 145
title: 'Cracking the Coding Interview: Q 16.5 Run Threads In Sequence'
date: 2015-01-10T00:29:27+00:00
author: Ke
layout: post
guid: http://www.keblog.me/?p=145
permalink: /2015/01/cracking-the-coding-interview-q165-run-threads-in-sequence/
comments: true
dsq_thread_id:
  - 3407114453
categories:
  - programming
tags:
  - cracking the coding interview
  - multi-threading
---
With the help of C++11 multi-threading library, this question about running threads in sequence can be easily implemented. Both implementations below is public on [GitHub](https://github.com/Ulu2005/Code_Practice/tree/master/cc150/ch16).

## Question

Suppose we have the following code:
{% highlight c++ %}
public class Foo {
    public Foo() { ... }
    public void first() { ... }
    public void second() { ... }
    public void third() { ... }
}
{% endhighlight %}

The same instance of Foo will be passed to three different threads. Thread A will call first, thread B will call second, and thread C will call third. Design a mechanism to ensure that first is called before second and second is called before third.

<!--more-->

## Solution 1

The first method is using semaphores. But C++11 multi-threading library doesn&#8217;t have semaphore implementation. So here we use POSIX semaphores(semaphore.h) instead, it should be available in Mac and Linux system.

The general idea is to use binary semaphores to invoke specific thread in sequence. The following implementation works well on linux with GCC, but being compiled with Clang on Mac, race condition occurs(Clang warns that sem_init is deprecated, thus semaphores may not be well initialized).

{% highlight c++ %}
#include <iostream>;
#include <thread>;
#include <mutex>;
#include "semaphore.h";

using namespace std;

mutex mtx; //cout exclusive access

class Foo {
public:
    Foo() {
        sem_init(&sem2, 0, 0);
        sem_init(&sem3, 0, 0);
    }

    void first() {
        mtx.lock();
        cout  "first"  endl;
        mtx.unlock();

        sem_post(&sem2);
    }

    void second() {
        sem_wait(&sem2);

        mtx.lock();
        cout  "second"  endl;
        mtx.unlock();

        sem_post(&sem3);
    }

    void third() {
        sem_wait(&sem3);

        mtx.lock();
        cout  "third"  endl;
        mtx.unlock();
    }
private:
    sem_t sem2;
    sem_t sem3;
};
{% endhighlight %}

As the function std::cout is not thread-safe, I used another mutex lock to gain exclusive access.

The program running result:

[<img src="http://www.keblog.me/wp-content/uploads/2015/01/q16-5_result.png" alt="threads in sequence" width="1078" height="176" class="alignleft size-full wp-image-161" srcset="//www.keblog.me/wp-content/uploads/2015/01/q16-5_result.png 1078w, //www.keblog.me/wp-content/uploads/2015/01/q16-5_result-300x49.png 300w, //www.keblog.me/wp-content/uploads/2015/01/q16-5_result-1024x167.png 1024w, //www.keblog.me/wp-content/uploads/2015/01/q16-5_result-500x82.png 500w" sizes="(max-width: 1078px) 100vw, 1078px" />](http://www.keblog.me/wp-content/uploads/2015/01/q16-5_result.png)

## Solution 2

The second method is to use condition variable. It&#8217;s part of C++11 standard library. The general idea is to run different thread with different requirement. When specific requirement is satisfied, related thread will be awaked from wait status. Thus threads are sequentially invoked.

{% highlight c++ %}
#include <iostream>;
#include <thread>;
#include <mutex>;
#include <condition_variable>;

using namespace std;

class Foo {
public:
    Foo () {
        job_rank = 1;
    }

    void first() {
        unique_lockmutex; lck(mtx);
        while(job_rank != 1) {
           cv.wait(lck);
        }
        job_rank++;

        cout  "first"  endl;
        cv.notify_all();
    }

    void second() {
        unique_lockmutex; lck(mtx);
        while(job_rank != 2) {
           cv.wait(lck);
        }
        job_rank++;

        cout  "second"  endl;
        cv.notify_all();
    }

    void third() {
        unique_lockmutex; lck(mtx);
        while(job_rank != 3) {
           cv.wait(lck);
        }
        job_rank++;

        cout  "third"  endl;
    }
private:
    mutex mtx;
    condition_variable cv;
    int job_rank;
};

int main(int argc, char *argv[])
{
    Foo foo;

    thread th1(&Foo::first, &foo);
    thread th2(&Foo::second, &foo);
    thread th3(&Foo::third, &foo);

    th1.join();
    th2.join();
    th3.join();

    return 0;
}
{% endhighlight %}

The way member functions being passed into threads is tricky. It&#8217;s slightly different from passing a normal function into a thread.

## Reference

[http://www.cplusplus.com/reference/condition\_variable/condition\_variable/](http://www.cplusplus.com/reference/condition_variable/condition_variable/)

<http://blog.poxiao.me/p/multi-threading-in-cpp11-part-3-condition-variable/>
