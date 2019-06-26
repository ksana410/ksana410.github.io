---
title: OpenWRT编译篇1- Ubuntu系统安装
tags: openwrt
date: 2018-12-30 16:03:47
---


# OpenWRT介绍
## 起源

> 2003年底Linksys公司推出WRT-54G，一款基于MIPS架构的无线路由器，使用802.11g标准使得带宽在理论上能够达到54M，在当时是一次巨大的进步。WRT-54G操作系统以Linux取代vXworks，哥伦比亚大学法学院教授Eben Moglen向Linksys提出开源要求。2003年7月，Linksys迫于压力，开源了WRT54G的firmware，不久sveasoft公司开发了Alchemy。从此无线路由器进入了可以刷机的时代。2004年1月出现所谓的OpenWRT，第一个版本是基于Linksys源码及uclibc中的buildroot项目。
>> [维基百科](https://zh.wikipedia.org/wiki/OpenWrt)

## 常见分支

* LEDE[^1]
* DD-WRT
* Asuswrt

---

# 编译准备
## 所需工具
1. 一台电脑
2. linux操作系统
3. 梯子
4. 足够的耐心
5. 零食

## 可能需要的软件
1. 虚拟机软件 [^2]
2. Ubuntu 64bit系统镜像
3. Shadowsocks客户端

---

# Linux系统安装
-  安装虚拟机软件
下载虚拟机软件太麻烦了，既然我使用的是win10那就用windows自带的__Hyper-v__ [^3]吧！
首先打开控制面板










[^1]: OpenWRT和LEDE在2018年1月5日正式合并
[^2]: 常用的免费虚拟机软件有VMware公司的[VMware Workstation Player](https://my.vmware.com/zh/web/vmware/free#desktop_end_user_computing/vmware_workstation_player/15_0)和ORACLE公司的[VM VirtualBox](https://www.virtualbox.org/wiki/Downloads)
[^3]: Hyper-V是微软提出的一种系统管理程序虚拟化技术，能够实现桌面虚拟化
