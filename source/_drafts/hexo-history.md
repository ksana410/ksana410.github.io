---
title: hexo折腾启示录
tags: hexo
---
image:
# Hexo折腾过程
---

## 相遇时刻
> 这是两年前的一个午后吧，申请了github，想要在上面整点项目吧，后来发现编程水平不到家，也许搞不出什么高质量的东西，那就想着做个博客；那用什么做CMS呢，用最流行的wordpress呢还是啥别的，好像对于我这个懒人，折腾wordpress太麻烦了，还是找个轻量点的东西吧，最终我遇到了Hexo，恩，就是它了，还能用github建站……
---

## 更新历史

- 2019年01月10日，开始编写初稿
---

## 折腾开始的时刻
感谢[HelloDog](https://wsgzao.github.io/)让我第一次见识到了Hexo所生成的网页，同时也在这个小站看到了Hexo的搭建过程，这上面分享的一些教程还是很值得一看的
__那就开始折腾吧__，**PS**：本文以Linux环境为例

### 安装GIT

Debian系
```shell
apt-get install git -y
```
Redhat系
```shell
yum install git -y
```

### 安装Node.JS

[Node.JS](https://nodejs.org/zh-cn/)
image

建议安装长期支持版，这个比较稳定

记录一下最麻烦的从源码安装：
```shell	
wget https://nodejs.org/dist/v10.15.0/node-v10.15.0.tar.gz
```
下载源码至本地
```shell
tar xzf node-v10.15.0.tar.gz
cd node-v10.15.0
./configure --prefix=/usr/local/node/10.15.0
```
如果此处出现如下报错则可能是gcc等组件没有安装
image：
修复也很简单，直接执行
```shell
yum install gcc g++ gcc-c++ -y
```
然后再次执行
```shell
./configure --prefix=/usr/local/node/10.15.0
```
此处的**--prefix=**是修改安装路径的，如果没有问题的话就可以开始编译了，这将会是一个漫长的过程，去看个视频吧！
```shell
make&&make install
```
image:

