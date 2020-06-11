---
title: Linux路由补完计划 挖坑起因
date: 2019-07-17 23:19:36
tags:
  - Linux
---

## 前言

前段时间我一直都在发布编译OpenWRT的相关视频（其实自己也没发布几个，重度懒症患者），可是实际上我并不是一个OpenWRT用户，很难相信吧！具体是什么情况，就让我娓娓道来吧～

---

<!-- more -->

## 简单展示

<iframe width="560" height="315" src="https://www.youtube.com/embed/01FVI6bnCcU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---

## 历程

作为一个网络技术人员，如果单纯使用百度的话真的找不到多少有用的资料，特别是当初学习Linux的时候，虽然我的入门书是[**《鸟哥的私房菜》**](http://linux.vbird.org/linux_basic/)这样的写得非常详细，甚至说有些罗嗦的书，但实际操作中还是有很多机会遇到解决不了的问题，那怎么办呢！只能用搜索引擎，当时正值Google还未退出中国，Google引擎还是很好用的时候，后悔入坑了Google全家桶，搞得我至今还是无法割舍Google帐号所带来的便利，后来的事情相信大家也是知道的，Google退出中国，只能使用香港服务器，到后来Gmail被屏蔽，Google彻底被墙，那不能用怎么办呢，当初还非常天真的认为baidu应该还能凑合吧，可是最后发现那就是个垃圾～～～

没办法，先从一些免费的工具开始吧，手机上安装了很多免费翻墙软件，然后随着天朝加紧GFW的建设，这些软件都退出了历史舞台，直到Shdowsocks的出现，我发现了可以在路由器上配置透明代理这样的操作，正值[**极路由**](https://zh.wikipedia.org/wiki/%E6%9E%81%E8%B7%AF%E7%94%B1)兴起的智能路由器风潮（当时并没有了解过OpenWRT，大家不要喷我），然后就入坑了极1S，然后官方的第三方应用被限制，没办法，求助google，仔细去研究了一下OpenWRT，刷了感觉还凑合，可是极1S是百兆的路由器，家里的宽带支持多拨，它的性能跟不上……

15年的时候软路由开始兴起，一开始并没有入坑，不过呢，本身在公司就有服务器资源，使用[**ESXI**](https://en.wikipedia.org/wiki/VMware_ESXi)搭建了软路由来进行测试，其中也包括一些大名鼎鼎的防火墙产品，比如[**PFSENSE**](https://www.pfsense.org/)，[**Panabit**](http://www.panabit.com/)等，它们都很好，但是呢不能挂梯子，这是我所非常迫切的功能，没办法，最终只能使用x86版本的OpenWRT，当时好像对于多核和大内存支持还有问题，多拨的表现也很不咋地，那就稍微妥协一下吧，用OpenWRT做二级路由，通过它来挂梯子，在公司内部测试感觉方案成熟了，然后我就利用一块志强X3360+超微X7SBL-LN2在家整了一台，此时已经到了17年，KoolShare论坛的固件开始崭露头角，可是在使用了之后发现挂梯子稳定性真的捉急啊（也许是我的错觉吧，每天不重启一下梯子准挂），最终让我放弃OpenWRT的主要是它吧，当然其中也经历了大半年的煎熬。当然OpenWRT还是不错的，只是我不想再用了，[**Lean大神**](https://github.com/coolsnowwolf/lede)的固件还是非常靠谱的（原谅我一直没用他的源码进行编译）。

{% asset_img 01.png linux_router %}

最终我只能寻找替代方案，此时我正在用ss-local挂代理，然后研究了一下OpenWRT的挂梯子原理，知道了ss-redir这个Shdowsocks中的组件，好了，就你了，从挂代理一下子就进化到了透明网关，之后就开始了我Linux路由折腾之路，从第一个成熟版本到现在，差不多已经有两年时间了

---

## Linux路由进化史

* 实现路由功能：iptables命令配置nat及forward
* 透明代理达成：iptables配置转发及ss-redir模式实现
* DNS防污染：能挂梯子但是却无法获取到正确的地址，那等于没翻墙
* 广告屏蔽功能实现：发现一个有意思的项目Pihole，利用起来效果还可以
* 实现一些附加功能：KMS、内网穿透、状态监测、VPN等

> 以上即是我折腾出的功能

---

## 自用Linux系统情况及改进计划

| 功能 | 过去 | 现在 | 未来计划 |
| :------: | :------: | :------: | :------: |
| 操作系统 | Debian | Debian | alpine |
| 处理器核心数 | 2 | 2 | 2 |
| 内存大小 | 1G | 256M | 128M |
| 梯子应用 | shadowsocks | v2ray | v2ray/clash |
| 防火墙 | iptables | iptables | nftables |
| 广告过滤 | none | pihole | Adguard-home |
| DNS防污染 | ss-tunnel | dnscrypt-proxy | Adguard-home |
| 附加功能 | none | kms\frpc\wireguard\netdata | ipxe\iscsi\upnp |

> 这只是大致的情况，具体的会在后续内容中更新

---

## 教程更新计划

> 需要一定的网络和Linux基础，懒癌患者托更比较严重，望周知

### 基础

* [x] Linux路由补完计划00 虚拟机安装Debian
* [x] Linux路由补完计划01 基本概念说明
* [x] Linux路由补完计划02 Debian安装及配置基础路由功能
* [x] Linux路由补完计划03 配置DNS和DHCP

### 进阶

* [x] Linux路由补完计划04 利用DNS进行广告过滤
* [x] Linux路由补完计划05 DNS防污染
* [ ] Linux路由补完计划06 透明代理搭建——V2ray篇
* [ ] Linux路由补完计划07 透明代理搭建——Shadowsocks篇
* [ ] Linux路由补完计划08 利用二级路由搭建Wireguard VPN
* [ ] Linux路由补完计划09 应用容器化（docker应用实例）

### 扩展

* [ ] Linux路由补完计划10 LVM or RAID
* [ ] Linux路由补完计划11 搭建文件共享
* [ ] Linux路由补完计划12 内网穿透

---

## 更新历史

* **2019.07.17** 初始版本
* **2019.07.18** 增加部分链接及说明调整
* **2019.08.04** 完善更新计划
* **2019.09.10** 格式调整
* **2020.03.03** 计划调整
