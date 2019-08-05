---
title: Linux路由补完计划02 Debian安装及配置基础路由功能
date: 2019-07-20 22:16:56
tags:
  - Linux
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

在ESXI中安装虚拟机应该不难，Debian9的安装也很简单，大致上过去在Hyper-V上安装Ubuntu一样，为了节省时间，我就直接忽略这部分了（希望大家不要打我），只不过为了有效利用256M的内存，需要在组件选择的时候稍微调整一下，如下图所示：

{% asset_img 25.png install vm %}

> 需要取消掉桌面环境和打印服务组件，启用ssh服务，桌面的环境肯定会消耗不小的性能，而打印机我觉得作为二级路由暂时还用不到，用不到就不浪费空间来安装了，然后启用SSH的话就可以远程进行管理了

#### 正式配置

---

##### 1.安装Open-vm-tool组件

安装Open-vm-tool之后会比较方便的进行虚拟机开关机和重启，安装也很简单

首先需要点击控制台上的虚拟机选项——客户机—— **安装/升级VMware Tools**，中间会跳出几个对话框，不用管，直接确定

{% asset_img 26.png configure debian 01 %}
{% asset_img 27.png configure debian 02 %}
{% asset_img 28.png configure debian 03 %}

之后在控制台中执行下述命令

```shell
mount /dev/cdrom /mnt #挂载光驱至mnt目录下
cd /mnt && cp VMwareTools-10.2.5-8068406.tar.gz /tmp #此处的VMwareTools的版本会随着ESXI版本的不同而不同，多使用Tab键补全命令
cd /tmp
tar xzf VMwareTools-10.2.5-8068406.tar.gz #将刚刚复制的压缩包解压
cd vmware-tools-distrib/ #进入刚刚解压出来的压缩包目录
ls
```

此处即可看到安装组件的脚本

{% asset_img 29.png configure debian 04 %}

直接执行 `./vmware-install.pl` 即可，此处会进入交互安装模式，除了第一个需要回答之外，其它的基本上一路回车即可，此处直接输入y并回车

{% asset_img 30.png configure debian 05 %}

安装完成之后就可以在ESXI的摘要界面上看到虚拟机此时使用的IP地址了，出现这个就说明安装成功并正常运行了

{% asset_img 31.png configure debian 06 %}
{% asset_img 32.png configure debian 07 %}

##### 2.安装基础软件及配置环境

> 此处安装的软件并不是必须的，但可以提升Linux的操作便利性（**只用命令行还谈什么便利性**），我一般会执行这样的命令

`apt-get install vim htop lrzsz git mlocate tmux -y`

当然，由于使用了中文语言环境，在控制台上显示的中文都是方块，还是用SSH配置比较好

{% asset_img 34.png configure debian 08 %}

之后再调整一下快捷命令，执行如下命令 `vi ~/.bashrc` ，此处的 **~** 表示的是用户家目录，类似于Windows上Administrator目录，里面存放着用户的个人文件和配置， **.** 表示隐藏的文件

{% asset_img 36.png configure debian 09 %}

将上述的内容修改成如下所示

{% asset_img 37.png configure debian 10 %}

这些内容主要是让文件进行彩色显示，同时设置一些快捷命令，比如“ll”就相当于执行了“ls -l”，有兴趣的可以对比一下这些命令的差异，保存并退出之后，执行 `source ~/.bashrc` 即可应用上面的那些修改

##### 3.修改SSH用密码登录

如果要使用SSH登陆的话需要修改一下SSH的配置文件，此处执行 `vi /etc/ssh/sshd.config` ，找到这个位置

{% asset_img 38.png configure debian 11 %}

修改成如下内容即可

{% asset_img 39.png configure debian 12 %}

之后执行 `systemctl restart sshd` 重启一下服务就可以应用修改了

##### 4.配置网卡地址

按照原定的计划，虚拟机的两张网卡分别配置成WAN口和LAN口，此处执行 `ip a` 查看网卡名，**ens192** 将作为WAN口， **ens224** 作为LAN口

{% asset_img 40.png configure debian 13 %}

网卡的配置文件是“/etc/network/interface”,直接执行 `vi /etc/network/interface` 进行编辑，下面的是我的配置，除了IP，网关等信息外，我还使用 **auto** 替换了 **allow-hotplug** ，确保在系统启动时候无论网卡处在何种状态都启用网卡

{% asset_img 55.png configure debian 14 %}

直接执行 `systemctl restart networking` ，然后通过 `ip a` 查看网卡情况，配置的网卡信息就已经生效了

{% asset_img 42.png configure debian 15 %}

之后就可以通过SSH工具进行连接了，此处我选择了[Xshell](https://www.netsarang.com/zh/xshell/),使用个人和学生版即可，商业版是需要收费的

{% asset_img 43.png configure debian 16 %}

##### 5.开启转发并配置iptables

> Linux如果要成为一台路由器，必须先开启转发的功能

 `vi /etc/sysctl.conf`对 **sysctl.conf** 文件进行编辑，定位到如下位置

{% asset_img 46.png configure debian 17 %}

删除此行前面的 **#** 符号，然后保存，执行 `sysctl -p` 即可生效，并且重启也不会受到影响

---

接下来就是重头戏了，iptables命令，此处就不深入进行讲解了，如果大家有兴趣我会更新一下这方面的知识

```shell
# 重置iptables配置
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
iptables -t raw -F
iptables -t raw -X

# 转发两块网卡之间的数据
iptables -A FORWARD -i ens224 -s 192.168.100.0/24 -j ACCEPT
iptables -A FORWARD -i ens192 -d 192.168.100.0/24 -j ACCEPT

# 源地址伪装，将所有内网的地址全部伪装成WAN口的地址，所有数据包都修改了来源的地址，这里是172.16.1.50/24
iptables -t nat -A POSTROUTING -o ens192 -j MASQUERADE

# 开启必要的端口和策略，此处开启WAN口的SSH端口
iptables -I INPUT 1 -i lo -j ACCEPT
iptables -I INPUT 1 -i ens224 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -i ens192 -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 配置默认的防火墙规则，防火墙一般都是以先允许后阻止这样的原则进行配置的，并且防火墙的规则是从上而下依次进行匹配的，需要开放的规则必须要放在阻止的规则上面，此处就是最后的阻止部分
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
```

{% asset_img 47.png configure debian 18 %}

配置完上述命令之后，路由功能基本上就实现了

##### 6.实测

口说无凭，还是拿出点证据来吧！我这次就用上次演示的Manjaro虚拟机吧，修改网络连接，使用最新建立的那个网卡

{% asset_img 48.png configure debian 19 %}
{% asset_img 49.png configure debian 20 %}

同时修改一下虚拟机的网卡地址，此次我只是配置了路由，DHCP和DNS功能并没有配置，所以需要手动指定，将网络设置为192.168.100.10，和 **ens224** 网卡同一个网段，网关直接指向 **ens224** 的地址，DNS就是用最常用的114.114.114.114吧

{% asset_img 50.png configure debian 21 %}

先ping一下看看通不通

{% asset_img 51.png configure debian 22 %}

就用[B站](https://www.bilibili.com/)验证一下吧

{% asset_img 52.png configure debian 23 %}
{% asset_img 53.png configure debian 24 %}

右侧显示的是 **ens224** 的流量信息，看1080P的视频一点问题都没有（原谅我Manjaro的性能有点差，4K是跑不动了）

##### 7.iptables规则固化

Debian的iptables不会开机启动，如果不进行一些操作的话，一重启一开始配置的iptables规则就会失效，路由功能就废了，为了解决这个问题，需要在开机的时候就加载iptables规则

执行`cd /etc/network`打开network目录，此时在这个目录下有如下几个文件

{% asset_img 54.png configure debian 22 %}

每个文件对应的功能如下：

|文件名|说明|
| :---: | :---: |
|if-down.d|网卡关闭时执行动作|
|if-post-down.d|网卡关闭后执行动作|
|if-pre-up.d|网卡启动前执行动作|
|if-up.d|网卡启动时执行动作|
|interface|网卡配置信息|
|interface.d|网卡扩展配置|

此处需要用到两个目录，**if-pre-up.d** 和 **if-post-down.d**  ，我需要写两个脚本，分别在网卡开启前导入iptables规则和在网卡关闭后自动保存iptables规则，看似功能很“高级”，但实际上实现非常简单

在**if-post-down.d**目录中新建一个文件，`vi /etc/network/if-post-down.d/iptables`，然后输入如下内容

```shell
#!/bin/bash

/sbin/iptables-save > /etc/iptables/rule.v4 #将iptables规则保存至/etc/iptables目录下的rule.v4文件中
```

同样的在**if-pre-up.d**目录中新建一个文件，`vi /etc/network/if-pre-up.d/iptables`并输入如下内容

```shell
#!/bin/bash

/sbin/iptables-restore < /etc/iptables/rule.v4 #将/etc/iptables/rule.v4中的iptables配置导入系统
```

以上的操作即可在系统重启时自动保存iptables规则，并开机加载

### 更新历史

---

* **2019.07.21** 下载镜像及配置虚拟机网络
* **2019.07.26** 完成主体内容
* **2019.07.28** 增加重启后加载iptables的方法
* **2019.07.29** 修正部分错误
