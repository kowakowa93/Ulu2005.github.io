---
id: 118
title: 'Cracking the Coding Interview: Q 2.6 Circular Linked List Problem'
date: 2014-12-26T22:24:49+00:00
author: Ke
layout: post
guid: http://www.keblog.me/?p=118
permalink: /2014/12/cracking-coding-interview-q26-circular-linked-list/
comments: true
dsq_thread_id:
  - 3363293553
categories:
  - programming
tags:
  - cracking the coding interview
  - linked list
---
Recently I’m working on “Cracking the Coding Interview 5th Edition”. All my solutions are public on <a href="https://github.com/Ulu2005/Code_Practice/tree/master/cc150" target="_blank">Github</a>. For this typical question about circular linked list in chapter 2, the way to solve it is a bit tricky.

## Question

Given a circular linked list, implement an algorithm which returns node at the beginning of the loop.

<u>DEFINITION</u>

Circular linked list: A (corrupt) linked list in which a node’s next pointer points to an earlier node, so as to make a loop in the linked list.

<u>EXAMPLE</u>

Input: A -> B -> C -> D -> E -> C [the same C as earlier]

Output: C

<!--more-->

## Solution 1

As every node in linked list has an unique address, we can use hash map to record nodes we have visited. By traversing the whole list, the loop beginning node will appear twice sooner or later. In C++, using container set is qualified here.

The implementation is post below.
{% highlight c++ %}
#include <iostream>;
#include <set>;

using namespace std;

typedef int elemtype;
typedef struct list{
    elemtype data;
    struct list *next;
}*node;

node FindCircle(node head);

int main(int argc, const char *argv[])
{
    /* Make gcc happy */
    return 0;
}

/* implemented with approximate hash table */
node FindCircle(node head) {
    set<node>; buffer;
    node start;

    for (start = head; start != NULL; start = start->;next) {
        if (buffer.find(start) != buffer.end()) {
            return start;
        }
        else {
            buffer.insert(start);
        }
    }

    return NULL;
}
{% endhighlight %}

## Solution 2

This method is exactly the same with solution on the book. But here I’ll prove it with my own understanding.

The general idea is that using two pointers to traverse the list, one fast, one slow. The speed of slow pointer is 1 and faster pointer is 2. If the linked list has a loop in it, two pointer will meet somewhere in circle. So the question becomes how to find the beginning of loop after two pointer meet.

[<img class="aligncenter size-full wp-image-119" src="http://www.keblog.me/wp-content/uploads/2014/12/SOL_21.png" alt="cracking the code interview" width="620" height="270" srcset="//www.keblog.me/wp-content/uploads/2014/12/SOL_21.png 620w, //www.keblog.me/wp-content/uploads/2014/12/SOL_21-300x131.png 300w, //www.keblog.me/wp-content/uploads/2014/12/SOL_21-500x218.png 500w" sizes="(max-width: 620px) 100vw, 620px" />](http://www.keblog.me/wp-content/uploads/2014/12/SOL_21.png)Assume the list has a “non-looped” part of length k, the length of loop circle is n. Two pointers meet at length of x over the loop beginning pointer. In other words, they are n-x length behind the loop beginning pointer.

When two pointer meet, fast pointer has moved 2m steps and slow pointer moved m steps repectively. It’s easy to prove,

<pre class="lang:default decode:true ">m = k + x
2m = k + pn + x</pre>

Thus,

<pre class="lang:default decode:true ">2m - m = pn, p = 1, 2, 3, 4…</pre>

So,

<pre class="lang:default decode:true ">m = k + x = pn

k = pn - x
  = (p - 1)n + (n - x)</pre>

From last equation, it’s clear to see that if we put fast pointer back to list head and let it move with speed 1. As both pointers continue to move forward, they will meet again at the loop beginning position.

The implementation can be found on book.
