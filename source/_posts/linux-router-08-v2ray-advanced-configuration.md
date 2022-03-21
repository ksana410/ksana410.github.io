---
title: Linux路由补完计划补充篇 分流及路由优化
tags:
  - Linux
date: 2019-10-19 23:51:15
---

## 前言

---

一直写不出自己满意的内容，那么就推翻重来吧，上一篇主要是搭建起了透明代理，上外网应该是没多大问题了，但是如果真的按照这样的方法就会遇到一个问题，部分国内的网站会出现访问缓慢的问题，这可能是系统没有正确的判断ip地址的所属地，把它当成国外的网站进行了代理，这明显是愚蠢的行为，所以这篇就完善一下这个方案，分享一下我个人的操作～

<!-- more -->

## v2ray作用模块

---

想要实现路由及分流，最主要需要编辑的是如下模块：

* inbounds
* outbounds
* routing

> 这三个部分的作用比较大，其它的暂时就不做分析了

## v2ray工作流程

inbounds <----> routing <----> outbounds

> 本地客户端通过 v2ray 对外开放的端口进行接入，此处常用socks或者dokodemo-door

## v2ray进行分流

---

v2ray的分流主要集中在 **routings** 部分进行配置

### 域名匹配

> 域名匹配支持多种模式，字符串匹配、正则表达式、子域名、完整匹配、预定义域名列表、从文件中加载域名，官方推荐的方法是使用子域名，但个人比较推荐使用预定义域名列表，v2ray 本身已经内置在了安装包中
，可以直接使用，调用也很简单

预定义域名列表由 [domain-list-community](https://github.com/v2fly/domain-list-community) 项目维护，预置于每一个 v2ray 的安装包中，文件名为 geosite.dat。这个文件包含了一些常见的域名，使用方式：geosite:listname，如 geosite:google 表示对 domain-list-community 项目 data 目录里的 google 文件内包含的域名，进行路由筛选或 DNS 筛选。

常见的域名有：

* category-ads：包含了常见的广告域名。
* category-ads-all：包含了常见的广告域名，以及广告提供商的域名。
* tld-cn：包含了 CNNIC 管理的用于中国大陆的顶级域名，如以 .cn、.中国 结尾的域名。
* tld-!cn：包含了非中国大陆使用的顶级域名，如以 .hk（香港）、.tw（台湾）、.jp（日本）、.sg（新加坡）、.us（美国）.ca（加拿大）等结尾的域名。
* geolocation-cn：包含了常见的大陆站点域名。
* geolocation-!cn：包含了常见的非大陆站点域名，同时包含了 tld-!cn。
* cn：相当于 geolocation-cn 和 tld-cn 的合集。
* apple：包含了 Apple 旗下绝大部分域名。
* google：包含了 Google 旗下绝大部分域名。
* microsoft：包含了 Microsoft 旗下绝大部分域名。
* facebook：包含了 Facebook 旗下绝大部分域名。
* twitter：包含了 Twitter 旗下绝大部分域名。
* telegram：包含了 Telegram 旗下绝大部分域名。
更多类别，可以查看 [data](https://github.com/v2fly/domain-list-community/tree/master/data) 目录

以下示例将自动屏蔽常见的广告域名，并直连 apple 和大陆域名站点，自动将 google 等域名转发到两个不同的对外出口上（如果你有多个服务器的话可以这样操作），没有指定的域名会自动选择默认的出口进行转发。

```json
{
  "routing": {
  "domainStrategy": "IPIfNonMatch",
  "domainMatcher": "mph",
  "rules": [
    {
      "type": "field",
      "outboundTag": "blockout",
      "domain": [
        "geosite:category-ads-all" //匹配广告域名，并路由到 blockout 部分，即被拦截
      ]
    },
    {
      "type": "field",
      "outboundTag": "direct",
      "domain": [
        "geosite:apple", //匹配 Apple 旗下域名，并路由到 direct 部分，即直连
        "geosite:cn" //匹配大陆域名，并路由到 direct 部分，即直连
      ]
    },
    {
      "type": "field",
      "outboundTag": "proxy01",
      "domain": [
        "geosite:google", //匹配 Google 旗下域名，并路由到 proxy01 部分，即转发到 proxy01 出口
        "geosite:facebook" //匹配 Facebook 旗下域名，并路由到 proxy01 部分，即转发到 proxy01 出口
      ]
    },
    {
      "type": "field",
      "outboundTag": "proxy02",
      "domain": [
        "geosite:microsoft", //匹配 Microsoft 旗下域名，并路由到 proxy02 部分，即转发到 proxy02 出口
        "geosite:twitter",  //匹配 Twitter 旗下域名，并路由到 proxy02 部分，即转发到 proxy02 出口
        "geosite:telegram" //匹配 Telegram 旗下域名，并路由到 proxy02 部分，即转发到 proxy02 出口
      ]
    }
  }
}
```

### IP文件

和预定于域名列表一样，v2ray 在安装包中也内置了预定义的 IP 列表，使用方法和上述的域名列表类似，实例如下：

```json
{
  "type": "field",
  "outboundTag": "direct",
  "ip": [
    "geoip:cn", //匹配大陆 IP 列表，并路由到 direct 部分，即直连
    "geoip:private" //匹配私有 IP 列表(常见的内网 IP 地址，如 192.168.1.0/24)，并路由到 direct 部分，即直连
  ]
}
```

### 第三方文件

除了安装包中内置的 geosite.dat 和 geoip.dat 文件外，v2ray 还可以使用第三方的域名列表文件，我个人使用了

* [V2Ray-SiteDAT](https://github.com/ToutyRater/V2Ray-SiteDAT)
* [v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat)

### 实例

## 历史

---

* **2019.10.19** 初稿
* **2019.11.04** 协议分析
* **2022.03.22** 部分调整
