---
title: Linux路由补完计划03 配置DNS和DHCP
tags:
  - Linux
date: 2019-07-26 23:03:13
---

## Linux路由补完计划03 配置DNS和DHCP

### 前言

---

上一篇文章中我将Linux配置成了一台路由器，并进行了演示，但当时由于没有配置DHCP和DNS，我只能手动的对客户机的网卡进行配置，手动的添加了IP，子网掩码，网关地址以及DNS，这期我将记录下我是如何搭建DNS和DHCP的，让所有接入的网络设备自由的获取IP，当然在此之前，一定要先把Linux的路由功能给搭建起来，否则做这些好像就没有多大意义了。

<!-- more -->

### 搭建方案

---

> Linux上的工具很多，不同的人有不同的搭建方案，这次我将使用两个方案来实现DNS和DHCP的功能，他们分别是dnsmasq和AdGuard Home+dhcpd，两个方案搭建起来都比较简单，稍微修改一下配置文件即可

* 方案一：
[dnsmasq](https://wiki.archlinux.org/index.php/Dnsmasq_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))是一款轻量化的DNS服务器，当然它的功能还不止这些，还能提供DHCP和PXE网络启动，对于家里这种比较小型的局域网，用它简直太合适了

* 方案二：
[AdGuard Home](https://adguard.com/zh_cn/adguard-home/overview.html)是AdGuard公司开源的一款使用Go语言开发的DNS服务器软件，支持家长控制和广告过滤，关键还支持[DNS over TLS](https://zh.wikipedia.org/wiki/DNS_over_TLS)，对于部署环境还不怎么挑剔；配置简单，虽然它自身也支持DHCP功能，但还处在BATE阶段，试用之后感觉也不是特别好，那就用[dhcpd](https://wiki.archlinux.org/index.php/Dhcpd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87) )来提供DHCP功能吧！

### 正式搭建

---

> 由于我是懒人，懒得多搭建几个测试环境，那就利用虚拟机的快照功能建立一个恢复点吧，不仅可以防止配置出错，还能多折腾几种方案，哪种方案靠谱就删掉其它的还原点就可以了

#### 建立快照

---

直接在 控制台——虚拟机——快照——生成快照

{% asset_img 01.png 快照01 %}

然后给这个快照起个名字，写点说明，等它建立好就行了

{% asset_img 02.png 快照02 %}

好了，前期的准备就结束了，进行下一步

---

#### 利用dnsmasq搭建

---

##### 安装dnsmasq

**dnsmasq**已经内置在debian的软件源之中，安装起来非常简单，直接执行`apt-get install dnsmasq -y`即可，之后系统会直接进行安装，并将启动文件，配置文件等项目创建好了，开机自启也添加到了启动项

##### dnsmasq的配置文件

接下来要做的主要就是修改**dnsmasq**的配置文件，配置文件主要是`/etc/dnsmasq.conf`和`/etc/dnsmasq.d`目录，前面一个是主配置文件，后面的那个目录属于扩展配置，当然为了管理更加方便，此处我会使用扩展配置的方式进行设置，但是默认情况下**dnsmasq**并不会调用`/etc/dnsmasq.d`中的配置文件，为了解决这个问题，需要在主配置文件中修改一个参数，`vi /etc/dnsmasq.conf`打开配置文件，移动到文件末尾，删除`conf-dir=/etc/dnsmasq.d/,*.conf`前的 **#** 符号，之后保存退出，这样**dnsmasq**就会自动进入 **/etc/dnsmasq.d** 目录中查找配置文件

{% asset_img 05.png dnsmasq 01 %}

之后就可以进入 **/etc/dnsmasq.d** 目录中进行相关的配置了，在我的服务器上，我对每个功能设置了一个配置文件，如图：

{% asset_img 06.png dnsmasq 02 %}

> 这里先不讨论这些配置文件分别对应什么功能，这样做只是为了更方便配置，哪个功能出现了问题就配置对应的文件，考虑到今后功能会越来越多，配置会越来越复杂，这种方式相比于一个文件解决一堆问题会有条理很多（没办法，男人对于自己电脑有种莫名的强迫症）

现在需要让**dnsmasq**实现DNS和DHCP的功能，那就创建两个配置文件吧！此处如图：

{% asset_img 07.png dnsmasq 03 %}

##### 配置DNS功能

实现DNS功能其实非常简单，打开配置文件，`vi /etc/dnsmasq.d/dns.conf`，输入下述内容：

```shell
listen-address=127.0.0.1,192.168.100.1 #监听本地和192.168.100.1两个地址，主要提供本机和LAN口的DNS服务
no-resolv #不使用/etc/resolv.conf中的DNS服务器，后面server选项已经指定了上级服务器，故不需要用到系统网卡配置的DNS
cache-size=1000 #保留1000个缓存地址，提高解析的速度，特别是有污染的情况下
server=114.114.114.114 #上级服务器，此处选择了国内很常用的114 DNS
server=8.8.4.4 #同样的我也使用了谷歌的服务作为后备，这个根据实际的情况进行调整，当然，如果后期要做DNS防污染，此处配置同样重要
clear-on-reload #重启后清除缓存
```

轻量级DNS服务器的话，上述配置就足够了，接下来就是DHCP功能。

##### 配置DHCP功能

在修改配置前，首先记录一下LAN口的网卡MAC地址，这个地址在后面的配置中需要用到，执行`ip a`即可查询到，此处查询到的MAC地址是**00:0c:29:c4:0a:41**

{% asset_img 08.png dnsmasq 04 %}

接下来执行`vi /etc/dnsmasq.d/dhcp.conf`，输入如下内容：

```shell
interface=ens224 #提供DHCP服务的网卡
bind-interfaces #绑定网卡，防止对其它网卡的干扰
dhcp-range=192.168.100.100,192.168.100.200,12h #地址分配范围及租用时间，我设置的是100～200这101个地址，实际上根本就用不了这么多，有效期12个小时，格式一定要注意，中间要用英文的逗号
dhcp-host=00:0c:29:c4:0a:41,192.168.100.1 #DHCP网卡的信息，MAC地址及IP，此处为必要项
dhcp-leasefile=/etc/dnsmasq.leases #保存DHCP的分配的主机和地址
```

> 简单的DHCP功能只需要这些配置就足够了，感觉很高大上，其实需要修改的也就这么点东西，现在就不引入一些高阶的功能了，够用就行

##### 验证一下

* 使用`dnsmasq --test`命令验证一下配置文件，如果没有问题的话会显示`dnsmasq: syntax check OK.`

---

* 重启一下**dnsmasq**来应用配置，`systemctl resatet dnsmasq`，验证一下状态，`systemctl status dnsmasq`，此处可以大致看到dnsmasq运行的状态，没问题的话按**q**退出

{% asset_img 09.png dnsmasq 05 %}

---

* 使用验证依然使用Manjaro啦，打开虚拟机，修改一下网卡配置，从原来的手动指定修改为自动获取，DNS也改成自动，重启一下网卡之后就能自动获取到ip地址了

{% asset_img 10.png dnsmasq 06 %}
{% asset_img 11.png dnsmasq 07 %}

那如何验证不是我暗箱操作呢？直接在服务器上查看DHCP分配情况就可以了，查看对应的记录文件`cat /etc/dnsmasq.leases`

{% asset_img 12.png dnsmasq 08 %}

之后那就放个[《罗小黑战记》](https://www.bilibili.com/bangumi/media/md1733)作为结尾吧，（等下，还有AdGuard Home呢，大家不要失去耐心啊～）

{% asset_img 13.png dnsmasq 09 %}

---

#### 利用AdGuard Home搭建

---

### 历史记录

---

* **2019.07.26** 撰写初稿
* **2019.07.29** 完成dnsmasq配置部分
