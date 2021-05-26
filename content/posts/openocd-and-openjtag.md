---
title: openocd 基础与百问网的 openjtag 介绍
slug: openocd-and-openjtag
date: 2015-04-12 15:24:03
categories:
  - 嵌入式
tags:
  - 嵌入式
  - openocd
  - openjtag
---

使用 openocd 的话, 最好是先看看 openocd 的官方手册, 100 多页, 不需要全看, 但是基本, 核心的概念要了解, 比如说 debug adapter/adapter, interface, target, board 等.

### Adapter 与 Interface 配置文件

debug adapter 或者直接叫做 adapter 呢, 就是指的你所使用的调试适配器, 一般来说就是 jtag 适配器了, 它会通过 JTAG 口与连接到开发板上, 比如 segger 家的 jlink, 开源的硬件项目 openjtag 等等.

使用不同的 adapter, 就需要在启动 openocd 时指定不同的 interface 文件, 这基本上是告诉 openocd 我的 adapter 用到了什么样的硬件, 参数是怎样的, 这样 openocd 才知道怎么和 adapter 通信, 因为 openocd 支持的 adapter 类型, 也就是支持的硬件设备类型太多了. 如果你写一个程序, 只支持 pl2303 类型的 usb-serial 设备, 那自然不需要像 openocd 那么复杂的配置了.

以我手里的百问网出品的 openjtag 为例, 它的 usb 转串口模块用的是 FTDI 的方案, 所以我们 interface 配置文件就要配置成 ftdi 啥的.

实际上百问网的 openjtag 就是参考了 openmoko [注1] 的第一个产品 --- Neo 1973 的方案, 所以对于百问的 openjtag, 我们可以直接使用 neodb.cfg 这个 interface 文件, 稍微一修改就行了. 因为 neodb.cfg 这个 interface 是随附在 openocd 安装包里的, 但是百问的 openjtag 就没有现成的 interface 文件, 这不能怪 openocd 团队, 因为 openocd 项目主页说了, 世界各地硬件那么多, interface 文件需要世界各地人们一起完善. 百问自己做了这个 openjtag, 却不向 openocd 组织提交相应的 interface 文件, 这就是百问网的不是啦.

### Target 与 Board

光为 openocd 指定 interface 文件是不够的, interface 文件只是告诉了 openocd 我们用的什么样的 adapter. 我们还需要告诉 openocd 我们用的 MCU/SoC 是什么样的. 这就需要我们指定 target 或者 board 配置文件.

安装好 openocd 后, 可以到 openocd 的 board 配置库里 (我这里是 /usr/share/openocd/scripts/board) 找找有没有你的板子的配置文件, 有的话就最好, 没有的话你就需要在 target 配置库里找找你的处理器的配置文件 (这个一般是都会有的了, 要是没有, 那你用的处理器也太罕见或者太新了), 然后参照一个和自己的板子最相近的 board 配置文件, 自己写一个 board 配置文件. 如果你自己测试用的还不错, 不要忘了可以将它提交 openocd 项目.

board 配置文件基本都是会包含它相应的 target 配置文件的. 比如 openocd board 配置库里有 mini6410 的配置文件, 这个配置文件的第一行就是: 

    source [find target/samsung_s3c6410.cfg]

openocd board 配置库里还有 smdk6410 的配置文件 (这可是三星官方的开发板, 所以自然是得有), 它的第一行也是:

    source [find target/samsung_s3c6410.cfg]

可惜, 我所用的 OK6410, 就没有现成的配置文件了.

不过, board 配置文件其实只是在所包含的 target 配置文件的基础上添加了一些初始化开发板上连接的外围设备的信息, 如果你暂时不需要使用开发板上接的那些外围设备, 而只使用处理器片内的资源, 那么你可以只指定 target 配置文件就行. 比如对于 mini6410, smdk6410, 还有我的 ok6410, 如果只是使用 s3c6410 的片内资源, 那么只指定 samsung_s3c6410.cfg 这个配置文件就行了.

当然, 不要忘了 interface 文件.

### 注释

1. 经 google 得知, openmoko 是一个专注于设计开源手机的项目, 包括硬件和操作系统, 都是开源的. Neo 1973 使它们的第一代产品.
2. 见 注1.