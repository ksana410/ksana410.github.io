---
title: Linux路由补完计划05 dnsmasq进阶配置
tags:
  - Linux
date: 2019-08-22 00:38:23
---

## Linux路由补完计划05 dnsmasq进阶配置

### 前言

---

相比于AdGuard Home，我更喜欢使用dnsmasq，主要是因为他配置起来会更加随意，配置一些其他的工具可以实现很多进阶的功能，本篇文章主要是在DNS和DHCP的基础上进行增强的演示和说明，希望可以给大家一些灵感

<!-- more -->

### 进阶功能

---

> 使用下述功能可以有效的增强挂梯子的效果，大家不妨尝试一下

* DNS防污染
* 对接DNS_over_HTTPS和DNSCrypt客户端
* 解析分流（中国域名使用国内解析服务器进行解析，国外的使用国外的解析）
* 广告过滤（限于篇幅，本篇暂未收录，主要是通过Pi-hole实现）

### 功能实现

---

#### 利用[DNSCrypt-Proxy](https://github.com/jedisct1/dnscrypt-proxy)搭建无污染DNS服务器

dnsmasq如果要作为一个DNS服务器存在，必须添加上级服务器，而此处添加的上级服务器将决定它能为下级客户端提供怎样的解析服务，上次做演示的时候使用了114.114.114.114和8.8.4.4这两个DNS服务器，如果大家看过上一篇文章的话，应该知道普通的DNS协议容易被投毒，为了获取的正确的没有被污染的地址，此处我将替换掉这两个地址，并利用DNSCrypt-Proxy搭建另一个DNS服务器

{% asset_img 01.png dnsmasq 01 %}

为啥需要两个服务器呢，原因主要是dnsmasq可以提供缓存功能，而没有污染的DNSCrypt-Proxy需要解析地址的话延迟感人，如果每一次都交由它来进行解析的话那体验真心难受，在[Linux路由补完计划3](https://youtu.be/Aez-j5dENaU)中的翻车现场就是因为DoH服务延迟太高造成的，所以在本地提供一个缓存可以弥补高延迟所带来的问题，同时也可以利用dnsmasq的一些其它的功能

##### 安装DNSCrypt-Proxy

DNSCrypt可以加密和认证用户和 DNS 解析服务器之间的数据传输。IP 数据本身没有任何变化，DNScrypt 可以避免 DNS 查询欺骗，确保 DNS 相应来自选择的 DNS 服务器。而DNSCrypt-Proxy这是这个工具的本地化工具，而且它支持的平台还特别多，基本涵盖市面上大部分的平台

* **项目地址：** <https://github.com/jedisct1/dnscrypt-proxy>

{% asset_img 02.png dnsmasq 02 %}

* **下载最新安装包：** 由于使用的32系统，所以我下载了[i386](https://github.com/jedisct1/dnscrypt-proxy/releases/download/2.0.25/dnscrypt-proxy-linux_i386-2.0.25.tar.gz)版本的

* **安装过程：**

```shell
# 进入opt目录，我将把DNSCrypt-Proxy安装在这个目录下
cd /opt

# 下载对应的压缩包
wget https://github.com/jedisct1/dnscrypt-proxy/releases/download/2.0.25/dnscrypt-proxy-linux_i386-2.0.25.tar.gz

# 将压缩包解压
tar xzf dnscrypt-proxy-linux_i386-2.0.25.tar.gz

# 进入解压到的目录
cd linux-i386
```

当执行`ls`命令时，可以看到DNSCrypt-Proxy的构成很简单

{% asset_img 04.png dnsmasq 03 %}

使用这个目录下的执行文件就可以完成DNS服务器的安装

```shell
# 生成配置文件，利用模板文件来生成
cp example-dnscrypt-proxy.toml dnscrypt-proxy.toml

# 通过配置文件修改服务的监听端口
vi dnscrypt-proxy.toml
```

通过关键字搜索需要修改的项目

{% asset_img 05.png dnsmasq 04 %}

由于dnsmasq作为主服务器已经占用了53端口，为了避免程序出错，此处修改为5353，同时由于ipv6还没有在本项目中实施，所以也删除ipv6监听端口，修改完之后是这样的

{% asset_img 06.png dnsmasq 05 % }

保存并退出，继续执行下述的操作

```shell
# 将DNSCrypt-Proxy安装为服务
./dnscrypt-proxy --service install

# 启动服务
./dnscrypt-proxy --service start
```

{% asset_img 07.png dnsmasq 06 %}

此时DNSCrypt-Proxy就已经安装好了，并且已经添加了开机启动项

* **验证是否运行：**

直接使用`htop`命令查看进程，DNSCrypt-Proxy进程已经正常运行

{% asset_img 08.png dnsmasq 07 %}

##### 替换dnsmasq上游DNS服务器

`vi /etc/dnsmasq.d/dns.conf`打开dns配置文件，将原来添加的114.114.114.114和8.8.4.4替换为127.0.0.1:5353，此处由于没有使用53端口作为监听口，所以必须手动指明端口号

{% asset_img 09.png dnsmasq 08 %}

执行`systemctl restart dnsmasq`即可生效

##### 选择DNS解析查询模式

DNSCrypt-Proxy不仅可以支持其自身的DNSCrypt协议，同时也支持通过DNS_over_HTTPS的方式来进行查询

选择哪种模式可以在配置文件中进行配置，`vi /opt/linux-i386/dnscrypt-proxy.toml`

{% asset_img 10.png dnsmasq 09 %}

默认情况下两种协议都是开启的，需要关闭哪种就将哪种后面的`true`改成`false`即可，同时重启一下服务即可`systemctl restart dnscrypt-proxy`

#### DNS查询分流

> 虽然添加了DNSCrypt-Proxy作为上游服务器来解决DNS污染，但由于使用的加密公共服务器普遍都在国外，解析国内网站就比较蛋疼了，为了解决延迟问题，对国内的域名使用国内的解析服务器进行查询会比较靠谱

利用[dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list)这个项目来解决这个问题，作者收集了大量的国内网站域名

## 历史

* **2019.08.22** 初稿
* **2019.08.23** 添加内容
