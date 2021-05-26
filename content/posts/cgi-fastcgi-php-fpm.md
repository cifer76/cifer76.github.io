---
title: 通俗地解释 CGI, FastCGI, php-fpm 之间的关系
date: 2017-11-09
categories:
  - Web
tags:
  - cgi
  - fastcgi
  - php
---

这要了解一点万维网 (WWW) 的历史, 才能更好地了解个中关系.

早期的网站基本都是静态的, 那时候的 web server 几乎所有工作就是给访问者提供静态资源, 网站与访问者之间缺乏交互. 后来随着 WWW 的发展网站变得交互性强了起来, 交互性强了也意味着 web server 端的业务逻辑复杂了起来, 不再是简单地解析 url, 定位并返回用户请求的资源, 而是要处理很多用户请求的动态资源以及许多复杂的业务, 这些工作都交给 web server 来做是不现实的, 因为单纯作为 web server 是不知道也不应该关注业务的.

于是 CGI 出现了, 它使得 web server 可以把复杂的业务逻辑交给 cgi 脚本程序来做, CGI 协议定义了 web server 与 cgi 程序之间通信的 context, web server 一收到动态资源的请求就 fork 一个子进程调用 cgi 程序处理这个请求, 同时将和此请求相关的 context 传给 cgi 程序, 像是 path_info, script path, request method, remote ip 等等...

但是显然每次来个请求 web server 就去 fork 子进程是很低效的, 在网站访问量逐渐增大时网站性能问题日益凸显. 所以后来 fastcgi 出来了, 它定义了一种通信规范使得 cgi 程序和 web server 之间能通过 socket 通信, 这样一来 cgi 这边也需要有一个专门的 daemon 进程来和 web server 保持连接, php-fpm 就是这么一个 daemon, 不仅如此, 而且 php-fpm 还是一套进程管理框架, 能够同时管理开启多个 fastcgi 进程, 保障通信的稳定性.

另外, 我们应该也都听过 apache 的 php5_module, 这是 apache 自己搞的一个插件架构, apache 自己有一套类似于 cgi 协议的东西叫做 sapi, php 的处理程序是直接静态或者动态编译进 apache 的, 可以直接被 apache 通过函数调用来调用, 这个并不能算是所有的事都交给了 apache 去干, 这得益于 apache 的插件架构, 而这个插件架构背后的思想和 cgi/fastcgi 实际上已经没有本质区别了.