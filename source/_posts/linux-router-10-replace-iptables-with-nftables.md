---
title: Linux路由补完计划10 使用nftables替换iptables
date: 2022-03-22 15:58:43
tags:
  - Linux
---

## 是时候可以考虑使用 nftables 了

---

nftables 是一个 netfilter 项目，旨在替换现有的 {ip,ip6,arp,eb}tables 框架，为 {ip,ip6}tables 提供一个新的包过滤框架、一个新的用户空间实用程序（nft）和一个兼容层。它使用现有的钩子、链接跟踪系统、用户空间排队组件和 netfilter 日志子系统。

<!--more-->
