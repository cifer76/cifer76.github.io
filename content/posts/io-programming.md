---
title: 深入解读同步/异步 IO 编程模型
date: 2017-11-06
categories:
  - Programming
tags:
  - async io
---

所谓 "同步" 和 "异步" 是从调用者的角度来说的. 如果调用者不得不等待 IO 完成才能执行后续的工作, 那就是同步; 否则, 就是异步. 这是我对 "同步" 和 "异步" 的定义, 这个定义清晰精炼, 巧妙的帮我们把 "理解什么叫做异步" 这项工作简化成了 "理解什么叫做 IO 完成".

在 \*nix 系统中, IO 操作分为两个阶段. 第一阶段是从用户空间发起请求到数据真正就绪的等待阶段, 第二阶段是数据就绪后从用户空间或者内核空间拷贝给对方的数据拷贝阶段. 只有这两个阶段都完成了, 才叫做 "IO 完成".

如果看过圣书 Unix Network Programming Volume 1 , 就知道 Richard 介绍了 5 种 IO 模型, 下面我们按照上面的定义给这 5 中模型分个类.

![图片缺失]()

# Blocking IO

这个模型是最简单的, 程序流调用 `read/writei`, 如果运气好正好有数据, 就进行 IO 第二阶段, 否则就卡在第一阶段等数据就绪. 当第二阶段结束 `read/write` 返回后, 继续执行后面的程序.

```
/* processing work A */

read(fd, buf, size);     /* blocked here */

/* continue processing work B */
```

显然这个模型是同步的, 程序流必须等 IO 两个阶段都完成了, 才能得以执行后续的工作.

# Non-blocking IO

这个模型是这样的:

```
while (read(fd, buf, size) < 0) {
    sleep(3);
}
```

如果数据没有就绪, `read()` 就会返回负数, 程序流就睡上几秒然后再去 `read()`. 显然这儿模型也是同步的, 因为同样只要 `read()` 没有完成, 程序流就会一直卡在这里, 无法往后执行.

# IO Multiplexing

这个模型最简单的情况是这样的:

```
select(fds, /* never timeout */);
/* find the ready fd */
read(fd, buf, size);
```

注意 `select()` 调用中 `timeout` 参数我们指定为永不过时, 这样一来这个模型也是同步的. 实际上这和 Blcoking IO 没有本质区别, 只是将检测数据就绪的工作交给 `select()` 调用去做了, 所以 fd 本身是阻塞还是非阻塞就无所谓了, 因为到了 `read()` 这一步的时候, 一定是数据就绪了.

`select()` 存在的意义是方便同时我们检测多个 fds, 只要有一个是数据就绪了, `select()` 就返回, 我们就去读. 但它并不能改变这个模型是同步模型的本质.

随着 nginx 火起来的事件驱动模型的核心就是 IO Multiplexing, 事件驱动模型是这样的:

```
while (1) {
    select(fds, timeout));
    
    if (/* exist ready fd */) {
        /* find the ready fd */
        read(fd, buf, size);
    } else {
        /* processing timer task */
    }
}
```

这种事件驱动模型也被普遍称为 "异步非阻塞 IO 模型", 一般用作程序的主干结构 main loop.

之所以说它是异步, 是因为这里 `select()` 的指定了一个有限的 timeout, 如果在 timeout 的时间内所有的 `fds` 都没有数据就绪的话, 就执行 else 分支的工作, 这样实际上就是 IO 没有完成, 程序流就继续往后执行了. 实际上这里就算没有 else 分支, 只要 timeout 不是无限的, 就可以说这个模型是异步的了.

# Signal-Driven IO

这个模型利用操作系统提供的信号机制:

```
function callback() {
    /* find the ready fd, time O(n) */
    for i in 0 ... fd count {
        if (read(fd, buf, size) > 0) {
            /* processing data in buf */
        }
    }
}


/* register a callback for SIGIO signal */
sigaction(SIGIO, callback);

/* processing continues */

/* at some time the callback be called by kernel */
```

显然这个模型是异步的, 程序流注册了一个回调之后就继续执行后续的工作了.

在实际应用中, 这个模型是有局限性的, 首先操作系统的信号数量有限, 供 IO 通知的更是只有 SIGIO 这么一个, 就算你可以再用 USR1/USR2 信号, 比起应用程序潜在的关心的文件描述符的数量仍然是少之又少, 所以回调方法被调用我们只是知道有内核数据就绪事件发生了, 但要知道是哪个描述符数据就绪了, 我们得把所有描述符读一遍试试, 这就要求我们将所有的描述符设为非阻塞的, 这样就绪的描述符就能成功读回数据, 没就绪描述符的读就返回负数.

但是, Richard 老爷子在 UNP 一书中将这种模型归类为同步模型, 因为 UNP 认为: 只有第一和第二阶段都不卡住程序流, 才叫异步, 否则都是同步. Signal-Driver 模型并没有解决第二阶段会卡一下程序流的问题, 因此也被 UNP 归到了同步模型中去.

那么, 接下来我们就看看 UNP 中定义的真-异步模型.

# Asynchronous IO

这个模型, 把第一和第二阶段都交给内核去做了, 用户程序需要提供一个会调, 以及一块缓冲区给内核, 然后执行 aio 调用. 程序流就继续执行, 然后在某个时刻操作系统调用我们的回调, 当回调被调用时, 我们会发现数据已经在我们提供的缓冲区里了 - 都不需要我们去 read!

这个模型就是 UNP 的真-异步模型. 当然, 按照我们上面的定义的话, 这个模型也毫无疑问是异步的.