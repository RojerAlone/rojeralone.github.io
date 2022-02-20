---
title: "OSTEP 读书笔记: 虚拟化之多处理器任务调度"
date: 2021-06-21T11:06:29+08:00
draft: false

categories:
- 读书笔记
tags:
- ostep
- 操作系统
---
# Multiprocessors Scheduling

之前讨论的都是单 CPU 上的多个任务调度，但是现代 CPU 大都是多核 CPU，多线程编程也随之出现，编程方式也和以前不一样，这里就讨论下多核调度策略。

多核带来的第一个问题是缓存的一致性问题， CPU 会缓存一些数据到 CPU 核中，多核情况下，一个任务从 A 核调度到了 B 核，那么在修改了值以后可能读到缓存的旧值，可以通过总线来解决，每个 CPU 核监听总线上的时间，当发现缓存数据对应的内存中数据变化时，标记脏数据或更新数据。

另一个问题是并发问题，最后一个问题是缓存亲缘性，一个任务最好一直在同一个 CPU 处理器上运行，这样就能使用之前的缓存了。

## SQMS

`single-queue multiprocessor scheduling` 的实现方式是将所有的任务都在同一个队列中，多处理器调度时都从这个队列中进行调度，优点是能够在之前提到的调度器上稍微修改就能运行。

缺点也很明显，多处理器消费一个队列，要加锁才能保证并发安全， 而当处理器核数很多的时候，性能就会变差。另一个问题是缓存亲缘性，这个问题也可以解决，通过忽略一些在其他处理器上运行的任务，但是实现起来比较复杂。

## MQMS

很自然想到，一个任务队列有问题，那么多个任务队列呢？这就是 `multi-queue multiprocessor scheduling`。 它同时解决了并发和缓存亲缘性问题。

MQMS 的问题是负载均衡，当任务放到一个 CPU 核的队列内后不再变化，任务提交时任务的执行时间不可预测，会出现某个 CPU 的任务全部完成了，另一个 CPU 还要运行好久才能完成任务。

这时候就不得不放弃一些缓存亲缘性了，将一些等待执行的任务迁移到空闲的 CPU 上。一种实现机制是工作窃取，一个 CPU 上的调度器偶尔会偷看另一个 CPU 上的调度负载，如果发现另一个 CPU 比较繁忙而自己比较空闲，那么就“偷”一个或一些任务来运行。这里又有一个权衡，不能太频繁偷看，否则消耗太高，这又是一个调参找阈值的地方。

## Linux Multiprocessor Schedulers

Linux 的调度器没有采用统一的多队列，O(1) 和 CFS 调度器都采用了多队列，但是 BFS 调度器并没有，它采用了单队列。书中并没有详细说明这些调度器的策略，下面是维基的页面：

-   [O(1) Scheduler](https://en.wikipedia.org/wiki/O(1)_scheduler)
-   [BFS](https://en.wikipedia.org/wiki/Brain_Fuck_Scheduler)