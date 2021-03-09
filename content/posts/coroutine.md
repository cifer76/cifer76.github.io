---
title: 对协程的一点认识
date: 2018-11-07
categories:
  - Programming
tags:
  - coroutine
---

# 协程的调度

我们知道线程是 CPU 的基本调度单元，线程调度靠的是时钟中断.

协程是执行于线程之内的更细粒度的执行单元，他的调度无法依赖时钟中断，而是要靠一个用户态的调度器，这个调度器可以是抢占式或非抢占式，抢占式调度器需要语言的运行时支持，据我所知只有 erlang 实现了协程的抢占式调度。大部分的协程实现都是非抢占式调度，非抢占式调度实际上是依靠协程之间相互让权 (yield) 来得到执行。

在非抢占式协程下，不存在协程同步问题。而在抢占式协程下则语言我们也考虑数据竞争，协程同步问题。

# 协程的好处

协程的一个典型应用是用在生产者 - 消费者问题中. 我们知道生产者 - 消费者问题也可以用多线程解决, 生产者线程和消费者线程共享一个上了锁的消息队列, 靠内核调度这两个线程执行来完成生产和消费过程, 然而这里有两个不足之处:

1. 靠内核调度线程, 存在线程切换开销
2. 消息队列加锁, 存在锁竞争和线程同步问题

内核调度线程的时机不确定, 如果在调度消费者时队列中没有消息, 消费者只能什么也不干就退出, 白白浪费了一次调度而如果用协程解决的话, 就不存在上述问题. 首先生产者和消费者协程位于统一线程里, 不存在线程切换的开销; 其次由于是单线程, 无需加锁, 也就不存在锁竞争问题; 最后由于协程之间的执行是靠主动让权 (yield), 我们可以在实现的时候仅当队列不空时才让权给消费者, 同理消费者仅当队列不满时才让权给生产者.

另外使用协程还有一个好处就是能够以看似同步的方式写异步的代码.

# 协程实现

要实现协程就需要自己在线程中维护第二层栈空间 (第一层是线程自己的栈空间), 因为线程的切换内核会为我们将当前上下文 (主要是各个寄存器的值) 保存在线程栈空间中, 现在由于线程需要自己调度协程, 所以线程需要为每个协程维护栈空间, 好在协程切换时保存协程的上下文.

这里需要线程能够获去到当前执行上下文, 很多操作系统内核会提供相应的系统调用, 实现方式其实也很简单就是写一段内嵌的汇编获取各个寄存器的值.

在 C/C++ 中, setjmp/longjmp 帮我们完成了这个任务. 关于其 setjmp/longjmp 的实现原理, 这里有篇 Google 排名第一的文章 讲的很清楚. C/C++ 中实现协程当然也可以不借助 setjmp/longjmp 而自己去实现上下文的获取和维护, Google 可以搜到不少.

# IO 阻塞问题

由于协程实际上就是不同的代码片段在同一个线程中交替执行 (这也被称作协作执行), 有一个协程因 IO 阻塞, 整个线程也就阻塞了, 其它协程也捞不着执行. 为了避免这个问题, 有些协程实现会 hook 所有的 IO 系统调用, 转化为非阻塞的, 然后协程调度器集中 select.

# 生成器

很多语言现在都支持生成器, 它们一般都是用关键字 yield 让一个方法变成生成器. 生成器其实是协程的一种特殊形式, 它被实现为在让权 (yield) 的时候总是让权给调用自己的位置, 而不是像自己写协程那样可以随便让权给任意位置.

使用生成器的好处, 一个是生成器为延迟计算提供了支持, 延迟计算就是指需要的时候才提供结果, 而不是事先提供好一堆结果, 这在大数据量处理的情境下可以节省大量内存. 另外就是使用生成器的代码一般来说能更简洁优雅, 提高代码可读性.

# Continuation

TBD...

# 参考
1. https://en.wikipedia.org/wiki/Coroutine
2. [libgo 的设计以及协程发展史介绍](https://my.oschina.net/yyzybb/blog/1817226)