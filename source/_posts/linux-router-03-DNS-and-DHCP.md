---
title: Linux路由补完计划03 配置DNS和DHCP
tags:
  - Linux
date: 2019-07-26 23:03:13
---

## Linux路由补完计划03 配置DNS和DHCP

### 前言

---

上一篇文章中我将Linux配置成了一台路由器，并进行了演示，但当时由于没有配置DHCP和DNS，我只能手动的对客户机的网卡进行配置，手动的添加了IP，子网掩码，网关地址以及DNS，这期我将记录下我是如何搭建DNS和DHCP的，让所有接入的网络设备自由的获取IP，当然在此之前，一定要先把Linux的路由功能给搭建起来，否则做这些好像就没有多大意义了。

<!-- more -->

### 搭建方案

---

> Linux上的工具很多，不同的人有不同的搭建方案，这次我将使用两个方案来实现DNS和DHCP的功能，他们分别是dnsmasq和AdGuard Home+dhcpd，两个方案搭建起来都比较简单，稍微修改一下配置文件即可

* 方案一：
[dnsmasq](https://wiki.archlinux.org/index.php/Dnsmasq_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))是一款轻量化的DNS服务器，当然它的功能还不止这些，还能提供DHCP和PXE网络启动，对于家里这种比较小型的局域网，用它简直太合适了

* 方案二：
[AdGuard Home](https://adguard.com/zh_cn/adguard-home/overview.html)是AdGuard公司开源的一款使用Go语言开发的DNS服务器软件，支持家长控制和广告过滤，关键还支持[DNS over TLS](https://zh.wikipedia.org/wiki/DNS_over_TLS)，对于部署环境还不怎么挑剔；配置简单，虽然它自身也支持DHCP功能，但还处在BATE阶段，试用之后感觉也不是特别好，那就用[dhcpd](https://wiki.archlinux.org/index.php/Dhcpd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))来提供DHCP功能吧！

### 正式搭建

---

> 由于我是懒人，懒得多搭建几个测试环境，那就利用虚拟机的快照功能建立一个恢复点吧，不仅可以防止配置出错，还能多折腾几种方案，哪种方案靠谱就删掉其它的还原点就可以了

#### 建立快照

---

直接在 控制台——虚拟机——快照——生成快照

{% asset_img 01.png 快照01 %}

然后给这个快照起个名字，写点说明，等它建立好就行了

{% asset_img 02.png 快照02 %}

好了，前期的准备就结束了，进行下一步

---

#### 利用dnsmasq搭建

---

未完待续……………………

### 历史记录

---

* **2019.07.26** 撰写初稿
