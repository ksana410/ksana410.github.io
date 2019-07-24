---
title: Linux路由补完计划02 Debian安装及配置基础路由功能
date: 2019-07-20 22:16:56
tags:
---

## Linux路由补完计划02 Debian安装及配置基础路由功能

### 前言

---

现在就开始建立自己的Linux路由吧，我将会根据情况将这一过程划分成及部分进行说明，每个部分实现一个或多个主要功能，并给出一些个人建议，此教程并不适合所有人，但我希望能给大家启发。

<!-- more -->

### 功能说明

---

本教程将以Debian为基础，在ESXI提供的虚拟平台上实现一个带有翻墙功能的路由系统，由于自己是一个懒癌晚期患者，故暂时不会提供一键脚本来实现此功能。我希望通过我一步步的操作，让大家对整个Linux路由有比较完整的认识，本期将演示下述内容：

* 虚拟机建立，及配置说明；
* Debian安装注意点；
* 网卡配置，内外网划分；
* 开启转发，实现基础路由功能；

### 需要的工具

---

* 虚拟机（本例以ESXI为配置平台，如果有实体机并带有两个网卡的话也是可以的，此处不涉及到基于VLAN的单臂路由）
* Debian9.x光盘镜像（由于计划分配的内存较少，此处使用32位镜像）
* SSH工具（如果是Win10的话可能只需要一个CMD即可）
* 一个梯子

### 开始折腾

---

#### 下载Debian镜像

---

虽然Debian10正式版已经发布了，考虑到源同步的问题，还是使用Debian9版本比较靠谱，并且考虑到分配的内存只有256M，我选用了32位系统镜像（**相信今后大部分Linux发行版都将舍弃32位系统，不过现在对我的影响不大**），当然如果你想用64位的系统也是可以的，内存稍微分配大点即可；由于在官网提供的是最新的版本，故我们要去别的地方下载。

Debian老版本下载网站请点击[这里](http://cdimage.debian.org/cdimage/archive/)，一直向下滚就可以看到我们需要的Debian9.9。

{% asset_img 01.png download debian 9 01 %}

点击我框出的位置，然后点击i386（此处表示32位系统），如下图所示：

{% asset_img 02.png download debian 9 02 %}

选择iso-cd，安装是会自动下载一些安装包，如果网络不是很好的话也可以选择iso-dvd，dvd版本会集成大部分需要用到的组件；

{% asset_img 03.png download debian 9 03 %}

随后即可点击你所需要的镜像了，我选择了netinst版本，即在安装的过程中会自动下载组件包的版本；

{% asset_img 04.png download debian 9 04 %}

当然，如果你懒得自己去找下载地址，我也会提供相应的下载链接（方便大家一下吧！）：

##### Debian9.9.0 下载链接

> 如果通过多线程下载软件如[IDM](https://www.internetdownloadmanager.com/)依然速度不理想的话，建议还是通过[BT](https://zh.wikipedia.org/wiki/BitTorrent_(%E5%8D%8F%E8%AE%AE))下载比较好

|32 or 64 |下载地址|
| :---: | :---: |
|32|[bt-cd](http://cdimage.debian.org/cdimage/archive/9.9.0/i386/bt-cd/debian-9.9.0-i386-netinst.iso.torrent)|
|32|[bt-dvd1](http://cdimage.debian.org/cdimage/archive/9.9.0/i386/bt-dvd/debian-9.9.0-i386-DVD-1.iso.torrent)/[bt-dvd2](http://cdimage.debian.org/cdimage/archive/9.9.0/i386/bt-dvd/debian-9.9.0-i386-DVD-2.iso.torrent)/[bt-dvd3](http://cdimage.debian.org/cdimage/archive/9.9.0/i386/bt-dvd/debian-9.9.0-i386-DVD-3.iso.torrent)|
|32|[iso-cd](http://cdimage.debian.org/cdimage/archive/9.9.0/i386/iso-cd/debian-9.9.0-i386-netinst.iso)|
|32|[iso-dvd1](http://cdimage.debian.org/cdimage/archive/9.9.0/i386/iso-dvd/debian-9.9.0-i386-DVD-1.iso)/[iso-dvd2](http://cdimage.debian.org/cdimage/archive/9.9.0/i386/iso-dvd/debian-9.9.0-i386-DVD-2.iso)/[iso-dvd3](http://cdimage.debian.org/cdimage/archive/9.9.0/i386/iso-dvd/debian-9.9.0-i386-DVD-3.iso)|
|64|[bt-cd](http://cdimage.debian.org/cdimage/archive/9.9.0/amd64/bt-cd/debian-9.9.0-amd64-netinst.iso.torrent)|
|64|[bt-dvd1](http://cdimage.debian.org/cdimage/archive/9.9.0/amd64/bt-dvd/debian-9.9.0-amd64-DVD-1.iso.torrent)/[bt-dvd2](http://cdimage.debian.org/cdimage/archive/9.9.0/amd64/bt-dvd/debian-9.9.0-amd64-DVD-2.iso.torrent)/[bt-dvd3](http://cdimage.debian.org/cdimage/archive/9.9.0/amd64/bt-dvd/debian-9.9.0-amd64-DVD-3.iso.torrent)|
|64|[iso-cd](http://cdimage.debian.org/cdimage/archive/9.9.0/amd64/iso-cd/debian-9.9.0-amd64-netinst.iso)|
|64|[iso-dvd1](http://cdimage.debian.org/cdimage/archive/9.9.0/amd64/iso-dvd/debian-9.9.0-amd64-DVD-1.iso)/[iso-dvd2](http://cdimage.debian.org/cdimage/archive/9.9.0/amd64/iso-dvd/debian-9.9.0-amd64-DVD-2.iso)/[iso-dvd3](http://cdimage.debian.org/cdimage/archive/9.9.0/amd64/iso-dvd/debian-9.9.0-amd64-DVD-3.iso)|

#### ESXI配置虚拟机

---

> 使用ESXI已经很多年了，相对而言配置更加得心应手，当然其他的虚拟化工具也是可以使用的，本次主要以ESXI为例，希望大家不要吐槽。

那就开始吧！

##### 上传镜像

将下载好的镜像上传至ESXI的内部存储中，等会安装虚拟机需要用到的；

##### 配置网络

家里原有的网络结构暂时不想改变，利用ESXI的虚拟网卡，这次我新建一个网络，在这个网络中进行调试，不影响我原有的网络结构；

网络结构及网段划分

###### ROS----172.16.1.0/24----Linux Router----192.168.100.0/24----Manjaro

> 家中的主路由是RouterOS，内网网段是172.16.1.0/24，计划将这台Linux Router的WAN口配置为172.16.1.50/24，LAN口配置为192.168.100.1/24，连接在LAN的设备所处网段是192.168.100.0/24，我将利用上次做演示的Manjaro做接入，到时候看看效果如何。

配置网络也很简单，进入 **配置----网络---点击右上角的添加网络向导** ，然后就可以添加新的网络了；

{% asset_img 05.png configure network 01 %}

* 连接类型选择虚拟机

{% asset_img 06.png configure network 02 %}

* 创建vSphere标准交换机（此处我并没有选择物理网卡，我直接使用内部通讯虚拟出一个网络交换机）

{% asset_img 07.png configure network 03 %}

* 给新建的网络起个名字

{% asset_img 08.png configure network 04 %}

* 直接点击完成，之后就可以看到网络中新出现的标准交换机

{% asset_img 09.png configure network 05 %}
{% asset_img 10.png configure network 06 %}

* 随后点击新的标准交换机的属性，进行编辑，在安全选项卡中启用混杂模式，此时会提示没有物理网卡，不用管，直接确定

{% asset_img 11.png configure network 07 %}
{% asset_img 12.png configure network 08 %}
{% asset_img 13.png configure network 09 %}

至此新的网络的配置就完成了。

##### 新建虚拟机

* 新建虚拟机，由于我使用的是6.0版本，ESXI上没有Debian9的虚拟机类型，那就选择Debian8吧，问题不大

{% asset_img 14.png configure vm 01 %}
{% asset_img 15.png configure vm 02 %}

* 选择两个处理器，内存就用原计划的256M吧

{% asset_img 16.png configure vm 03 %}
{% asset_img 17.png configure vm 04 %}

* 网卡选择两块，VM Network对应172.16.1.0/24网段，作为Linux Router的WAN口，Linux router对应192.168.100.0.24网段，作为LAN口，硬盘直接使用默认配置，16G足矣

{% asset_img 18.png configure vm 05 %}
{% asset_img 19.png configure vm 06 %}

* 以上结束之后记得勾选上完成前编辑虚拟机设置

{% asset_img 20.png configure vm 07 %}

* 在编辑界面需要修改两个位置，移除软盘，添加ISO镜像到虚拟光驱，并且勾选上打开电源是连接，如下图所示

{% asset_img 21.png configure vm 08 %}
{% asset_img 22.png configure vm 09 %}

> 至此虚拟机的前期配置就搞了，如果使用6.5或者6.7版本的ESXI，在网页上配置基本上也类似

##### 安装虚拟机

---

在ESXI中安装虚拟机应该不难，Debian9的安装也很简单，大致上过去在Hyper-V上安装Ubuntu一样，为了节省时间，我就直接忽略这部分了（希望大家不要打我），只不过为了有效利用256M的内存，需要在组件选择的时候稍微调整一下，如下图所示：

{% asset_img 25.png install vm %}

> 需要取消掉桌面环境和打印服务组件，启用ssh服务，桌面的环境肯定会消耗不小的性能，而打印机我觉得作为二级路由暂时还用不到，用不到就不浪费空间来安装了，然后启用SSH的话就可以远程进行管理了

##### 正式配置

---

###### 1.安装Open-vm-tool组件

安装Open-vm-tool之后会比较方便的进行虚拟机开关机和重启，安装也很简单

```shell
mount /dev/cdrom /mnt #挂载光驱至mnt目录下
cd /mnt && cp VMware
```

### 更新历史

---

* **2019.07.21** 下载镜像及配置虚拟机网络
