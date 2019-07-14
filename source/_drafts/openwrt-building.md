---
title: OpenWRT编译篇1- Ubuntu系统安装
tags: openwrt
date: 2018-12-30 16:03:47
---

# OpenWRT介绍

## 起源

> 2003年底Linksys公司推出WRT-54G，一款基于MIPS架构的无线路由器，使用802.11g标准使得带宽在理论上能够达到54M，在当时是一次巨大的进步。WRT-54G操作系统以Linux取代vXworks，哥伦比亚大学法学院教授Eben Moglen向Linksys提出开源要求。2003年7月，Linksys迫于压力，开源了WRT54G的firmware，不久sveasoft公司开发了Alchemy。从此无线路由器进入了可以刷机的时代。2004年1月出现所谓的OpenWRT，第一个版本是基于Linksys源码及uclibc中的buildroot项目。
> [维基百科](https://zh.wikipedia.org/wiki/OpenWrt)

## 常见分支

* LEDE
* DD-WRT
* Asuswrt

---

<!-- more -->

## 编译准备

### 所需工具

1. 一台电脑
2. linux操作系统
3. 梯子
4. 足够的耐心
5. 零食（如果怕胖就不要准备）

### 可能需要的软件

1. 虚拟机软件
2. Ubuntu 64bit系统镜像
3. Shadowsocks客户端

---

## Linux系统安装

* 安装虚拟机软件

下载虚拟机软件太麻烦了，既然我使用的是win10那就用windows自带的**Hyper-V**吧！
那就开始吧：

### 1.首先就是安装Hyper-V，启用其实很简单，直接在控制面板中启用即可

依次打开控制面板——程序——启用或关闭Windows功能，之后找到Hyper-V，勾选前面的方框即可，点击确定，系统配置跑完一圈之后重启即可

### 2.重启完之后的操作

在开始菜单中直接搜索Hyper-V 管理器，根据自己的需求修改相应的配置，比如虚拟磁盘的位置，编译需要的磁盘空间还是有点大的，所以还是选择一个磁盘空间稍微大的位置来存放虚拟磁盘（如果你的C盘够大，也可以忽略这条）

### 3.配置好之后就可以开始今天的主角了











---
OpenWRT和LEDE在2018年1月5日正式合并
常用的免费虚拟机软件有VMware公司的[VMware Workstation Player](https://my.vmware.com/zh/web/vmware/free#desktop_end_user_computing/vmware_workstation_player/15_0)和ORACLE公司的[VM VirtualBox](https://www.virtualbox.org/wiki/Downloads)
Hyper-V是微软提出的一种系统管理程序虚拟化技术，能够实现桌面虚拟化
