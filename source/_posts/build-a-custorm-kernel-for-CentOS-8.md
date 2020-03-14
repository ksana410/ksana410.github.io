---
title: 为CentOS 8编译自定义内核
tags:
  - Notes
date: 2020-03-13 15:41:11
---

## 前言

---

CentOS 8发布一段时间了，我也想体会一下这个版本有啥新特性，所以就在自己的ESXi主机上虚拟一个折腾一下，既然想折腾那就折腾的彻底一点，昨天狠下心来把主机上的黑群晖彻底弃用了，然后直通了一块相当于捡垃圾买到的DELL PERC 6iR直通卡，挂载了一堆小容量的硬盘做存储，但理想很丰满，现实很骨感，直通卡下挂的硬盘死活识别不到，为了解决这个问题，那就好好在google上晃悠了，最后发现可能需要重新编译下内核才能解决这个问题，那么就尝试一下吧，记录一下我编译内核的过程！

<!--more-->

## 编译环境要求

---

> 编译Linux内核其实和编译opnwrt差不多，所以准备工作也是类似的，大致上有以下几点：

1. 准备一个稍微大点的存储空间，个人测试编译到最后应该不会超过16G的容量，划分一个20G空间足够了

2. 安装必要的编译环境

3. 稍微有点耐心（编译的速度主要取决于处理器性能）

## 编译

* 下载内核源码

进入伟大的[kernel.org](https://www.kernel.org)

yum install hmaccalc zlib-devel binutils-devel elfutils-libelf-devel

yum install bc openssl-devel

cp /boot/config-4.18.0-147.5.1.el8_1.x86_64 ./.config

yum groupinstall "development tools"

yum install bc openssl-devel zlib-devel binutils-devel elfutils-libelf-devel ncurses-devel

make menuconfig or make nconfig

make -j 6 V=s

make modules -j 6 V=s

make modules_install

make install

ls -l /boot/vmlinux*

grubby --info=/boot/vmlinux-5.5.9

grubby --default-index

grubby --set-default=/boot/vmlinuz-4.18.0-147.5.1.el8_1.x86_64