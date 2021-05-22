---
title: 浏览器追踪技术与防范
date: 2019-02-13
categories:
  - Web
tags:
  - privacy
---

# Cookie

这里说的就是我们通常所熟知的 cookie，很多第三方公司就是借助这种 cookie 实现追踪的。比如网站 A，B 都使用了 DoubleClick 的 js 脚本，DoubleClick 的脚本在用户访问网站 A 时被加载并埋下 cookie，下次用户访问网站 B 时这个信息会上报回给 DoubleClick。这种方式实现起来简单，防范也简单，定期清理 cookie 就能定期的中断跟踪，或者我们也可以直接禁止第三方 cookie 就好了，不过这样一来一些非跟踪的第三方服务比如 google analytics/adsense 也会被误杀。

# Supercookie

Supercookie 应该是一个统称，不同于普通的 cookie，supercookie 通过各种奇技淫巧达到让用户难以清除甚至无法清除的目地。比如利用 flash 插件存储 cookie 数据，这样一来 cookie 数据就存放于 flash 插件的存储区，而浏览器一般是没有提供插件数据清理功能的，这就阻拦了大部分用户清理 cookie；另外还有一种方式叫做 Image hack，它利用了浏览器默认会缓存图片的行为，给你生成一张 100 像素的每个像素的颜色对你也是固定的小图片，以此来标记你，当浏览器下次发出请求时，一段简单的 js 代码就能够读出缓存中这张图片的像素颜色特征。

这两种方式可算是奇淫，但仍然是把特征信息存在用户电脑上，只要用户想，还是能够清除他们的。然而下面这第三种方式就真的无法清除了。

第三种方式是直接由 ISP 厂商参与，在中途给你插入 cookie 信息。具体来说就是 ISP 厂商分析你的流量，发现是 HTTP 请求，就使用你的网络接入信息给你生成 cookie 信息。这一勾当最早是由美国 Verizon 电信公司实践的，Verizon 为了更好的服务自己的那些广告主们，用这种方式跟踪用户。虽然这种方式使得我们无法清除 cookie，但因为这种方式需要 ISP 分析发现 HTTP 流量，所以只要我们尽可能的只访问 HTTPS 服务（HTTPS Everywhere 插件能够帮助我们尽可能多的走 HTTPS 链路），还是能够有效防止被通过这种方式跟踪的。

# Fingerprint

不管是 cookie 还是 supercookie，它们都是基于存储特征信息的，fingerprint 也就是指纹就不是这样了。

# Do Not Track

为了更好地保护用户的隐私，DNT 被提了出来。这是一项需要浏览器厂商和网站主以及第三方跟踪服务商共同履行的协议。它大致是这样工作的：

用户在浏览器端设置是否不让网站以及第三方跟踪服务商跟踪我，一旦设置了，浏览器就会在发出的 HTTP 请求中添加一个新的头 "DNT: 1"。遵守 DNT 协议的网站收到这个头之后首先自己就不会在收集记录用户信息了。然而我们知道，访问一个网站时还会有一些第三方的脚本能够收集我们的信息，在这个问题上，网站可能有两种处理方式：

1. 第三方服务本身也是遵守 DNT 协议的，那网站就不需要做任何事
2. 如果第三方服务并不遵守 DNT 协议，那么遵守 DNT 的网站就有义务对用户屏蔽这类第三方服务

对于第二种情况，常见的案例是一些第三方社交分享按钮，它们往往也有收集用户信息的功能，如果它们不遵守 DNT，那么网站应当采取不要对用户加载这类按钮的措施。

# Firefox 浏览器内置拦截功能

较新版本的 firefox 浏览器已经内置了跟踪器的拦截功能，在“配置 - 隐私与安全 - 内容拦截”选项页中可以具体配置。这里 firefox 借助 Disconnect.me 的提供的跟踪器列表（实际上就是一个 domain 清单）首先能够识别出一些常见的跟踪器，然后可以配置拦截其它第三方 cookie 以达到拦截更多潜在跟踪器的目的。


# 参考

1. https://askleo.com/supercookies_and_evercookies/
2. https://www.makeuseof.com/tag/what-are-supercookies-and-why-are-they-dangerous/
3. https://www.eff.org/pages/understanding-effs-do-not-track-policy-universal-opt-out-tracking