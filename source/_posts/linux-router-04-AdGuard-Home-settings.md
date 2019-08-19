---
title: Linux路由补完计划04 配置AdGuard Home实现广告过滤及DNS防污染
tags:
  - Linux
date: 2019-08-11 14:46:53
---

## Linux路由补完计划04 配置AdGuard Home实现广告过滤及DNS防污染

### 前言

---

上一篇教程中我分别使用dnsmasq和AdGuard Home实现了DNS和DHCP功能，如果只是实现这些功能，那它们就太Low了，这期内容将以AdGuard Home为例，利用其老本行，实现广告过滤及DNS防污染功能，特别是DNS防污染功能，对于后期的挂梯子至关重要，废话不多说，直接开始配置吧！

<!-- more -->

### 前情提要

---

对于Linux路由来说，找到一款合适的广告过滤工具并不容易，它本身不同于openwrt，拥有大量的插件提供无数功能，而对于Linux系统，虽然属于通用操作系统，openwrt也是由它衍生出来的，但通用和专用系统还是有很大差异的，就如同cpu一样，它再牛逼，干gpu的事情也不行，这样的比喻也许并不恰当，但也不是不能用是吧！

虽然openwrt上的一些工具已经有移植的版本，比如**koolproxy**，但这个并不是我今天所要讨论的重点，我选择了相对比较原始的过滤广告的方法，即DNS过滤，限制那些广告域名的解析，变相的屏蔽广告，效果可能没有那么好，但过滤个八九成广告还是问题不大的，关键一点是使用现有的工具就能做到。

当然，屏蔽广告很重要，防DNS污染也很重要哦，不要忘记初衷，搭建二级路由主要就是为了挂梯子了，获取到正确的IP至关重要，感谢**GFW**让我们感受到了DNS污染的牛逼，由于DNS协议太老旧了，污染它是成本的方法，甚至于我使用iptables配置一下都能劫持它，看吧，国家也是要考虑运营成本的啊，当然是选择便宜的大规模部署啦！所以使用常用的DNS服务地址都会被污染，比如常用的**114.114.114.114**，**8.8.8.8**等等，基本上都难逃污染，通过它们解析到的IP基本上都是不能用来翻墙的，都是错误的；连门牌号都搞错了，那如何才能找到你想要的东西呢，所以这是在建立梯子前必须解决的问题

### 说明

---

> AdGuard Home本身就是一款广告过滤工具，那广告过滤什么的就没啥压力了，同样的AdGuard Home还支持DNS_over_HTTPS和DNS_over_TLS等加密解析功能，利用这一点获取到正确的IP，就决定这样干，具体会不会有啥坑呢？先操作一下试试吧！

AdGuard Home整个系统已经将你需要的功能都构建好了，该怎么用完全取决于你使用哪种过滤策略，使用哪种上级DNS地址查询正确的IP地址，下面是我所使用的规则及服务器：

推荐的广告过滤规则
|规则名|规则地址|
| :---: | :---: |
|EasyList China| <https://easylist-downloads.adblockplus.org/easylistchina.txt>|
|CJX's EasyList Lite|<https://raw.githubusercontent.com/cjx82630/cjxlist/master/cjxlist.txt>|
|EasyList|<https://easylist.to/easylist/easylist.txt>|

上游DNS服务器选择
|DNS类型|可靠性分析|推荐地址|
| :---: | :--- | :---: |
|UDP_DNS|常规的DNS查询方式，使用UDP明文传输，安全性差，易污染|1.1.1.1，114.114.114.114，8.8.8.8，9.9.9.9|
|[DNS_over_TLS](https://zh.wikipedia.org/wiki/DNS_over_TLS)|默认使用853端口,使用TCP进行传输，相当于给DNS解析进行了加密，由于存在特定端口，容易被封锁（当然也可以使用443端口）|<tls://dns.adguard.com>,<tls://dns.google>|
|[DNS_over_HTTPS](https://zh.wikipedia.org/wiki/DNS_over_HTTPS)|为了反审查，DNS查询也需要加入混淆，将DNS查询伪装成HTTPS协议来欺骗GFW,这样就很难分辨哪些流量是正常的网络访问，哪些是DNS解析了，但是这样势必会造成时延增高|<https://cloudflare-dns.com/dns-query>,<https://dns.google.com/resolve>,<https://sdns.233py.com/dns-query>|
|TCP_DNS|使用TCP来代替UDP，同时使用非标准端口会较容易打开国外打不开的网站|<tcp://208.67.222.222:5353>,<tcp://208.67.220.220:5353>|
|[DNSCrypt](https://dnscrypt.info/stamps/)|一款小众的DNS查询系统，不同于现有DNS查询方式，但是协议特征明显，容易被识别，但现在GFW暂时没有对其进行大规模限制，在现有使用规模下它还是安全的|建议自建[客户端](https://github.com/jedisct1/dnscrypt-proxy)|

推荐优先级 DNS_over_HTTPS > DNS_over_TLS > DNSCrypt > TCP_DNS > UDP_DNS

DNS_over_HTTPS最为首选，如果使用国外的话延迟就太高了，国内的话使用<https://sdns.233py.com>提供的服务效果会好一点，如果自身技术水平较强的话，在周边国家采购vps自建也不错，比如在套路云上自建延迟会比较有优势

## 配置

---

未完待续

### 历史

---

* **2019.08.11** 成稿
* **2019.08.12** 随手写一点
* **2019.08.13** 添加部分链接
* **2019.08.19** 完善推荐列表
