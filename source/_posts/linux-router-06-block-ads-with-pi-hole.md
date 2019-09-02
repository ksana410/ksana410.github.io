---
title: Linux路由补完计划06 广告屏蔽就是这么简单——Pi-hole篇
tags:
  - Linux
date: 2019-08-29 22:33:34
---

## 前言

---

在上上期中我介绍了使用AdGuard Home来过滤广告的方法，今天我再推荐一款同样好用的本地DNS及广告过滤工具——[Pi-hole](https://pi-hole.net/)，相比于AdGuard Home来说，Pi-hole诞生更早，一开始它是为树莓派开发的一款广告过滤工具，在许多开发者的努力下，它被发扬光大了，也正因为这款工具的诞生，让我在过去那段时间里能让自己的移动设备免受广告的侵扰，以至于至今我还把它当成主力的广告过滤工具在我的软路由上默默的为全家手机服务

<!-- more -->

## Pi-hole是什么

---

Pi-hole的核心组件pihole-FTL实际上就是一个dnsmasq的分支，它相比于原版的dnsmasq在功能上有专门针对广告过滤的优化，它只是将过去需要手动进行的广告过滤规则进行了自动化配置；原版的dnsmasq可以通过address=这样的配置进行广告屏蔽，如果广告域名较多的话，那配置起来简直让人抓狂，而Pi-hole作为一款好用的工具简化了广告过滤规则的配置过程，真乃懒人之福音

{% asset_img 01.png pihole 01 %}

## 搭建过程

---

> 由于Pi-hole的本质就是dnsmasq，为了不影响其正常的安装，在安装之前需要卸载掉本机已经安装的dnsmasq，这也是为什么我在上期视频中没有附加广告过滤部分的原因

### 前期准备

* 备份dnsmasq配置并将其卸载:

执行命令`systemctl stop dnsmasq`将dnsmasq服务停止，之后就是将上次编写的配置文件给备份（实际上并不需要备份，基本上会自动生成），此处我使用`mv`命令，直接将整个目录重命名了，这样有两点好处：

1.备份

2.防止安装Pi-hole之后因为配置冲突造成启动失败

{% asset_img 30.png pihole 02 %}

此处实际上应该还要卸载，只是我忘记了，大家记得要执行下述命令来进行dnsmasq的卸载`apt-get autoremove dnsmasq -y`

* 安装必要的下载工具**curl**

安装Pi-hole的一键脚本需要用到，那就装呗，`apt-get install curl -y`

{% asset_img 31.png pihole 03 %}

* 配置代理

感谢伟大的GFW，装个国外的软件真心好痛苦，还好咱有代理可以用，`export`命令走起

{% asset_img 32.png pihole 04 %}

### 一键安装

想要安装Pi-hole还是很简单的（这是现在，过去可是很坑的，动不动依赖就会出问题，当然大部分原因都是有伟大的GFW），官方直接提供了一键脚本

{% asset_img 02.png pihole 05 %}

根据官方给出的命令，执行就是了`curl -sSL https://install.pi-hole.net | bash`

{% asset_img 33.png pihole 06 %}

如果不出意外的话，基本上都能顺利的进入配置界面

### 安装配置

* 在安装部分依赖之后，系统会自动进入配置的界面，如下图所示：

{% asset_img 05.png pihole 07 %}

{% asset_img 06.png pihole 08 %}

{% asset_img 07.png pihole 09 %}

> Pi-hole的一些说明之后就是正式的安装配置了

* 选择提供服务的网卡，此处是LAN口的**ens224**

{% asset_img 08.png pihole 10 %}

* 选择上游服务器，现在图省事我直接使用了Google的，然后直接确定进入下一步

{% asset_img 09.png pihole 11 %}

* 广告过滤的策略，先保持默认吧

{% asset_img 10.png pihole 12 %}

* 选择IP类型，**IPv4** OR **IPv6**，现在**IPv6**还有很多问题，暂时就只用**IPv4**吧

{% asset_img 11.png pihole 13 %}

