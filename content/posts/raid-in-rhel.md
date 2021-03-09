---
title: 搞定 HP MicroServer 的 Smart Array Controller B120i 磁盘阵列, 在安装 RHEL 时
slug: work-through-raid-in-rhel
date: 2014-01-22 09:48:00
categories:
    - Linux
tags:
    - rhel
    - HP 小型机
---

MicroServer Gen8 是 HP 服务器里较新的一个系列, 其所配备的磁盘阵列卡 --- Smart Array Controller B120i, 也是比较新的一种阵列卡, 目前 HP 仅提供了 RHEL, OpenSUSE, Microsoft 的驱动程序.

我们就是要在 MicroServer Gen8 上安装 RHEL6.

MicroServer Gen8 的主板上的 ROM 上搭在了一个小型的配置系统, 叫做 Intelligence Provisioning, 在这里你可以对磁盘阵列进行分区(正如 hardware raid 都会带有一个控制系统来管理自己的说法一样), 还可以配置你要安装的操作系统(不过在通过这个配置你可以安装的系统有限, 仅限于 HP 提供了阵列卡驱动的那些系统), 还带了一些系统健康状态监控的功能.

对于上述的几个可以在 Intelligence Provisoning 中配置的操作系统, MicroServer Gen8 似乎都提供了他们的安装程序, 这点比较方便, 因为在 Intelligence Provisioning 中配置好我们想安装的操作系统之后, 重启机器就回进入这个操作系统的安装界面, 然后你只需要提供操作系统的镜像就可以继续你的安装. 但是 Gen8 预置的 RHEL 操作系统安装程序却是有一个严重的不如人意的地方, 稍候我会说明这一点.

众所周知, 安装软件时, 一般来说这个软件会提供一个安装程序, 我记得 windows 下以前最火的制作安装程序的软件叫 InstallShield 不知现在还是不是最火的, 使用这个软件就可以制作出那种傻瓜化的一路下一步的软件安装程序, 而 linux 下的安装程序, 应该就得算各种包管理系统或者是`./configure, make, make install` 三步曲了吧. 那么安装操作系统的话, 也是需要一个操作系统安装程序的, 操作系统安装程序可谓多种多样, 甚至有些已经超越了"操作系统安装程序"这个界限, 自己直接提供了一个操作系统, 比如通过制作 Ubuntu Live CD/USB 安装盘, 你都可以直接使用在安装盘上的系统而不用把它安装到你的硬盘上 --- 当然, 这似乎已经不属于"操作系统安装程序"了. 这么分吧, 一般来说, 操作系统安装程序分为两种, 带图形界面的和不带图形界面的. 不管带不带图形界面, 操作系统安装程序都会包含一些安装过程中必备的硬件驱动程序, 比如硬盘的驱动程序, 这样安装系统的时候, 操作系统安装程序才能识别出你的硬盘然后让你选择将系统安装到哪个硬盘上; 可能还有网卡驱动程序, 这样你在安装的过程中就能联网获取更新.

一般来说, 操作系统安装程序中所带的驱动已经够了, 但是现在不行, 我们想把系统安装在 raid 上, 也就是 Smart Array Controller B120i 上, 这是一种 Hardware RAID, 这就需要我们的 RHEL6 系统安装程序具备识别 Smart Array Controller B120i 的驱动, 但是不幸的是 Redhat 尚未提供这个驱动, 用在 Redhat 官网下载的系统引导镜像(系统安装程序镜像)制作成的系统安装盘里, 并不具备 Smart Array B120i 的驱动, 无论哪个版本. 幸运的是, HP 提供了适用于 RHEL6.x 各个版本的驱动程序. 这样我们就可以在安装过程中加载这个驱动程序.

对了, 我们安装的 RHEL 版本是 RHEL6.2

Ok, 下面我们就开始正式的安装过程.

#### 1. 制作操作系统安装盘

这个参考了 [redhat 官方文档][Making_Minimal_Boot_Media], 我们制作了一个 BIOS-based 的 USB 启动盘, 注意不要用 UEFI, MicroServer Gen8 还不支持 UEFI, 我试验过, 制作了 UEFI-based 的启动盘, 不被识别, BIOS 中没有选项可以启用 UEFI, HP 官网一篇文章也说了 MicroServer Gen8 不支持 UEFI, 链接我暂时找不到了.

需要的镜像 rhel-server-6.2-x86\_64-boot.iso 可以在网上下载到. 制作时需要一个 U 盘.

上面我们也说了 Gen8 预置了各种系统的安装程序, 为什么我们这里还要制作系统安装盘呢, 因为 Gen8 预置的安装程序不能进入 boot prompt 界面, 而我们后面要进入这个界面手动加载 HP 提供的 Smart Array B120i 驱动. 这就是 Gen8 预置系统安装程序不尽人意的地方.

注意, 如果安装的是 rhel6.2, 就老老实实用 rhel-server-6.2-x86\_64-boot.iso 这个镜像制作安装盘, 不要用 rhel-server-6.3-x86\_64-boot.iso, 我一开始因为找不到 6.2 的boot 镜像, 但是找到了 6.3 的就用了 6.3 的 boot 来安装 rhel6.2, 但是在加载 Smart Array B120i 驱动那里折腾了好久 --- hpvsa-1.2.8-140.rhel6u2.x86\_64.dd 无论如何都加载不上.

要用 6.3 的 boot 就只能加载 hpvsa-1.2.8-140.rhel6ur3.x86\_64.dd 这个驱动, 但这是没道理的, 为何要用 6.3 的 boot 加载 6.3 的 Smart Array B120i 驱动然后安装 6.2 的 rhel? 这完全是找折腾受.

#### 2. 系统镜像盘制作

系统镜像 rhel-server-6.2-x86\_64-dvd.iso 网上可以下到, 下载之后要严格按照 [redhat 官方的文档][s1-steps-hd-installs-x86] 制作系统镜像盘, 这里我们需要另一个 U 盘. 

#### 3. 下载 HP 提供的 Smart Array B120i 驱动

HP 提供的驱动在 HP 官网下载, 要下载与自己系统对应的版本, 我们这里是 hpvsa-1.2.8-140.rhel6u2.x86\_64.dd, 下载之后这个驱动程序放在另一个 U 盘, 这个 U 盘不必是空的, 总之目前为止我们需要第三个 U 盘.  

#### 4. 开始安装

1.  插入第一个 U 盘, 启动 MicroServer Gen8, 首先进入 BIOS Setup 界面, 选择 Embedded SATA Configuration -> Enabel Dynamic HP Smart Array B120i RAID Support, 然后重启
2.  选择临时启动介质为 USB Key 启动
3.  进入 rhel server 的安装界面, 这个界面让你选择安装方式, 让光标停留在 "Install or Upgrade exsiting system" 这一项上, 然后按下 TAB 键直接在当前界面编辑引导参数, 或者是按下 ESC 键进入 boot prompt 界面, 插入第二和第三个 U 盘(包含 HP Smart Array B120i 驱动的那个)
4.  无论你在哪个界面, 输入 "linux dd blacklist=ahci", linux dd 命令使得你可以附加额外的驱动给系统安装程序, blacklist=ahci 这句是为了防止安装程序加载了普通的硬盘驱动将磁盘阵列识别为多个硬盘而不是一个阵列, 个人觉得可能不加 blacklist=ahci 也能成功但最好加上
5.  安装程序会询问你是否有包含驱动的磁盘, 选 Yes, 然后找到你刚才插入的 U 盘(不知道是那个的话就挨个打开看看), 找到里面的 hpvsa-1.2.8-140.rhel6u2.x86\_64.dd, 按 Enter, 等安装完了之后, 安装程序会问你还有没有别的驱动要安装, 我们没有别的驱动要装了, 选 No.
6.  安装程序会正式进入安装界面, 让你选则安装语言等, 在选择安装介质的时候, 选择第二个 U 盘, 继续, 会出现一个界面, 里面会显示安装程序检测到的所有连接到机器的上的存储介质, 不出意外的话, 你就会看到你的 RAID 阵列也在里面.
7.  接下来就是常规的系统安装操作, 调整分区, 下一步, 下一步....

至此, 安装安成.

[Making_Minimal_Boot_Media]: https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Installation_Guide/Making_Minimal_Boot_Media.html
[s1-steps-hd-installs-x86]: https://access.redhat.com/site/documentation/en-US/Red\_Hat\_Enterprise\_Linux/6/html/Installation\_Guide/s1-steps-hd-installs-x86.html