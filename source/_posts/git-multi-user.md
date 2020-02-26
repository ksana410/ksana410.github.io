---
title: git多账户模式
tags:
  - Notes
date: 2020-02-26 12:13:04
---

## 前言

---

使用github有一段时间，终于算是能流畅使用了，除了那坑爹网络情况（当然这主要怪大天朝的gfw），但最近遇到一个问题，个人有两个github账号，分别存储着不同类型的内容，我不想两者有太多的联系，所以希望不同的项目使用不同的账号进行管理和提交，这个就有点犯难了，该怎么做，在谷歌上查询了一圈终于找到了一个感觉比较方便的方法，此方法依然使用https进行提交，而不是大部分人推荐的使用ssh密钥的方式，对于个人电脑而言，安全性应该还会那么差，毕竟这电脑一直都在家；那么就记录一下过程吧！

<!--more-->

## 清除默认的git配置

---

首先查看一下git的配置

```shell
git config --global -l
git config --system -l
```

> 由于https方式的push每次都需要输入密码，所以一般都会使用credential.helper把账号密码存在global里，对于单用户没啥问题，多用户就悲剧了，通过上述命令查看一下credential.helper是在全局还是系统中设置的

如果懒得看，直接全部清除

```shell
git config --global --unset credential.helper
git config --system --unset credential.helper
```

## 针对项目配置账户

---

> 如果本地的项目比较多，可能这并不是一个明智的选择

* 进入项目目录

```shell
git remote set-url origin https://用户名@github.com/用户名/项目名
git config --local credential.helper store
```

* 进入别的目录继续执行上述操作

## 正式提交项目

---

当对项目进行了修改需要提交时（一般修改配置之后第一次会出现这样的情况）

需要你重新输入一次用户名和密码，按照要求输入即可，下次就不再需要重复输入用户名和密码了

> 基本上每个项目都会这样的情况发生，所以说项目较少可以这样干，项目多就不要考虑这个办法了，还是用更加安全方便的ssh密钥吧！

## 历史

---

* **2020.02.26** 初稿即成稿，太难得了
