---
layout: post
title: Race Conditions
author: Eduardo Souza
date: '2020-07-23 18:27:23 -0300'
category:
        - security
summary: Try not to break it
thumbnail: /assets/img/posts/race_conditions.jpg
---

**Race Condition** is a logical weakness that is extremely hard to detect. The bigger is the piece of code executed by multiple threads, the harder it is to determine where exactly something went wrong. It consists in two or more operations requesting the same resource simultaneously but the operations must be done in a given sequence to be done correctly.

There are different ways to prevent Race Conditions, and in all of them, the main idea is to let only one thread at a time access the critical piece of code.

 * Use keywords provided by the programming language (e.g. synchronized)
 * Use libraries that guarantee atomicity for the desired operations
 * Use libraries that provide advanced lock implementations (e.g. util.java.concurrency library)

Try to avoid implementing your own lock because it's not a trivial thing to do. Attempting to fight race conditions, you may end up with a deadlock, i.e. with a condition when two threads are waiting infinitely for each other to release the lock.

Use proven libraries and make sure not to introduce deadlocks. 
`Monitor.Enter()` acquires a lock on the `_exchangeLock` object. Note that choosing a lock object is a tradeoff between performance and security. To keep both security and performance on a sufficient level, one should lock only the critical parts of the code.

You can use try and finally construction to make sure that the lock is released regardless of the result of the code execution.
