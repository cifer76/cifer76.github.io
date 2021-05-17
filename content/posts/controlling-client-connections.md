---
title: 如何在应用层控制最大客户端连接
date: 2018-02-23
categories:
  - Programming
tags:
  - C
---

当有客户端连接, 而程序中没有去处理时, select 就回持续不断的返回这个文件描述符可写, 例如, 下面是我以前写的一段有 bug 的程序:

```
int csocks[MAX_CONNECTION];
memset(csocks, -1, MAX_CONNECTION * sizeof(int));

FD_SET(sock, &rset);
while(1) {
    if (select(FD_SETSIZE, &rset, NULL, NULL, NULL) <= 0) {
        return ;
    }
    if (FD_ISSET(sock, &rset)) {
        // looking for an unused socket
        for (int i = 0 ; i < MAX_CONNECTION; ++i) {
            if((-1 == csocks[i]) && (-1 != (csocks[i] = accept(sock, NULL, NULL))))
                break;
        }
    }
}
```

这段程序里, sock 是一个侦听套接字, 负责侦听客户端的连接, 一有连接就会去调用 accept 来接受客户端的连接 --- 当然, 这是有条件的, 那就是能够接收的最大的客户端数量是 MAX_CONNECTION, 由上面的程序里可以看到, 当连接的客户端的数量已经超过了 MAX_CONNECTION 时, 将不会再接受任何连接.

然后上面的代码有一个潜在的问题, 那就是, 当客户端连接满了, csocks 数组已经没有为 -1 的元素了, 那么新的连接到达时, accept 将永远不会被调用. 似乎没有问题是吗? 不, 问题很严重, accept 永远不会被调用的话, select 调用会知道 sock 文件描述符没有被你读过, 那么下一次 while 循环时, select 将会再次成功返回以表明 sock 文件描述符可读.

这是灾难性的, 你可以想象, 一个 while 循环, 里面 select 调用飞速的执行, 返回, 执行, 返回....

这是 select 调用的行为, 其实在串口通信, 普通文件读写时也是这样的情况: 当你用 select 监听一个串口文件, 当串口有数据到达, 但是你就是不去读取的话, 那么之后不管你调用多少次 selelct 调用, 它都会返回正数, 同时表明你传递的串口文件的描述符可以读.

回到最初的问题, 那么, 我们想要限制客户端连接的数量, 那么用上面的方法可不行, 当客户端数量到达了我们应用程序里设定的最大数目, 当新的客户端连接到达时, 我们也不能仅仅就是简单的不去 accept 它 --- 它会一直在那里, 下一次 select 会再次告诉你它在那里.

那么到底怎么做呢, 我们总得处理一下, 首先我们要告诉 select 调用, 这个新的客户端连接请求我已经处理过了, 下一次你就不用再把这个客户端的连接请求报告我了. 其次, 我们最好向这个新客户端返回一个 RESET 消息.


**Update:**

这个问题, 我已经在 stackoverflow 上提问并得到了答案: http://stackoverflow.com/questions/23379029/how-to-deny-clients-connection-properly-in-socket-programming/23379631#23379631