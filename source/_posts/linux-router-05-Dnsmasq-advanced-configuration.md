---
title: Linux路由补完计划05 dnsmasq进阶配置
tags:
  - Linux
date: 2019-08-22 00:38:23
---

## Linux路由补完计划05 dnsmasq进阶配置

### 前言

---

相比于AdGuard Home，我更喜欢使用dnsmasq，主要是因为他配置起来会更加随意，配置一些其他的工具可以实现很多进阶的功能，本篇文章主要是在DNS和DHCP的基础上进行增强的演示和说明，希望可以给大家一些灵感

<!-- more -->

### 进阶功能

---

> 使用下述功能可以有效的增强挂梯子的效果，大家不妨尝试一下

* DNS防污染
* 对接DNS_over_HTTPS和DNSCrypt客户端
* 解析分流（中国域名使用国内解析服务器进行解析，国外的使用国外的解析）
* 广告过滤（限于篇幅，本篇暂未收录，主要是通过Pi-hole实现）

### 功能实现

---

#### 利用[DNSCrypt-Proxy](https://github.com/jedisct1/dnscrypt-proxy)搭建无污染DNS服务器

dnsmasq如果要作为一个DNS服务器存在，必须添加上级服务器，而此处添加的上级服务器将决定它能为下级客户端提供怎样的解析服务，上次做演示的时候使用了114.114.114.114和8.8.4.4这两个DNS服务器，如果大家看过上一篇文章的话，应该知道普通的DNS协议容易被投毒，为了获取的正确的没有被污染的地址，此处我将替换掉这两个地址，并利用DNSCrypt-Proxy搭建另一个DNS服务器

{% asset_img 01.png dnsmasq 01 %}

为啥需要两个服务器呢，原因主要是dnsmasq可以提供缓存功能，而没有污染的DNSCrypt-Proxy需要解析地址的话延迟感人，如果每一次都交由它来进行解析的话那体验真心难受，在[Linux路由补完计划3](https://youtu.be/Aez-j5dENaU)中的翻车现场就是因为DoH服务延迟太高造成的，所以在本地提供一个缓存可以弥补高延迟所带来的问题，同时也可以利用dnsmasq的一些其它的功能

##### 安装DNSCrypt-Proxy

DNSCrypt可以加密和认证用户和 DNS 解析服务器之间的数据传输。IP 数据本身没有任何变化，DNScrypt 可以避免 DNS 查询欺骗，确保 DNS 相应来自选择的 DNS 服务器。而DNSCrypt-Proxy这是这个工具的本地化工具，而且它支持的平台还特别多，基本涵盖市面上大部分的平台

* **项目地址：** <https://github.com/jedisct1/dnscrypt-proxy>

* **下载最新安装包：** 由于使用的32系统，所以我下载了[i386](https://github.com/jedisct1/dnscrypt-proxy/releases/download/2.0.25/dnscrypt-proxy-linux_i386-2.0.25.tar.gz)版本的
```shell
wget https://github.com/jedisct1/dnscrypt-proxy/releases/download/2.0.25/dnscrypt-proxy-linux_i386-2.0.25.tar.gz

tar xzf dnscrypt-proxy-linux_i386-2.0.25.tar.gz

cd dnscrypt-proxy


```

## 历史

* **2019.08.22** 初稿
* **2019.08.23** 添加内容
