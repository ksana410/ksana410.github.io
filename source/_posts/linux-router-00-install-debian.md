---
title: Linux路由补完计划 虚拟机安装Debian
tags:
  - Linux
date: 2019-07-18 00:00:02
---

## Linux路由补完计划 虚拟机安装Debian

### 前言

---

Debian作为软路由系统的基础，它的安装还是需要拿出来说一下的，当然老鸟可以忽略，本说明教程只针对小白用户(当然，让所有人都能搞明白这是一种能力，本人还做不到，大家先凑合看看吧，如果有啥不明白的记得留言吧，我能够解决的基本上都会回复的)；在接下来的说明中，我将使用虚拟机来说明，也是因为利用了虚拟机，在网络配置上还是很灵活了，废话不多说，接下来就开始吧！

<!-- more -->

### 虚拟机配置

---

> 作为二级路由，本身对于性能的要求并不是特别大，我过去使用的那台虚拟机配置就很低，这次依然沿用那台虚拟机的配置情况

* 双核处理器
* 256M内存
* 8~16GB磁盘空间
* Debian9.9系统
* 双网卡

### 准备

---

> 安装之前还是需要说明一下虚拟机的配置

1. 新建虚拟机

{% asset_img 63.png debian 01 %}

新建一个虚拟机，使用自定义配置

{% asset_img 64.png debian 02 %}

随便给这个虚拟机起个名字，这里就叫做123了

{% asset_img 65.png debian 03 %}

虚拟机的位置就只能使用默认的位置了，其它的磁盘都被RDM了

{% asset_img 66.png debian %}

虚拟机版本就用6.0版本中最高的11了

{% asset_img 67.png debian 04 %}

Linux版本就用Debian 8了，ESXI6.0提供的配置中Debian8已经是最高的了，9he10虽然没有提供，但你直接用8的配置也是可以正常使用的

{% asset_img 68.png debian 05 %}

处理器就选择单处理器，双内核了，入门款配置

{% asset_img 69.png debian %}

内存能少就少，256M我都觉得有点多，下次试试128M能不能正常跑起来

{% asset_img 70.png debian 07 %}

由于不想折腾单臂路由，那就来个两块网卡吧，分别是主路由的内网部分和软件模拟的虚拟交换机

{% asset_img 71.png debian 08 %}

SCSI控制器就用系统默认的吧

{% asset_img 72.png debian 09 %}
{% asset_img 73.png debian 10 %}
{% asset_img 74.png debian 11 %}

创建新的虚拟磁盘，就用推荐的16GB吧，即使是这个大小，也用不完

{% asset_img 75.png debian 12 %}

在配置完之前可以勾选完成前编辑虚拟机设置，这样可以直接进行手动配置

{% asset_img 76.png debian 12 %}

在虚拟机属性中主要修改两个部分，软驱和光驱

{% asset_img 77.png debian 13 %}

移除软驱，这年头还有多少用，连[linus](https://zh.wikipedia.org/wiki/%E6%9E%97%E7%BA%B3%E6%96%AF%C2%B7%E6%89%98%E7%93%A6%E5%85%B9)都已经觉得过时了，那就真的过时了，然后就是在光驱中加载系统的镜像，并且需要勾选**在打开电源时连接**这个选项，这样开机才会从光驱引导

> 虚拟机的配置就是这些，资源使用真是不高

### 开始Debian安装

---

> 我将对每一步操作都做出一些说明

#### 安装过程

{% asset_img 01.png debian 14 %}

打开虚拟控制台并启动电源，如果配置没有问题的话，都会从光盘启动，然后就会出现上述界面，鼠标点击安装画面进入控制台，直接回车即可

{% asset_img 02.png debian 15 %}
{% asset_img 03.png debian 16 %}

后期会使用SSH登陆，有中文支持会比较舒服，直接选择中文，当然由于中文字符的特殊性，系统还是会警告一下询问你是否使用中文进行安装，不管了，直接用方向键左右选择**是**进入下一步

{% asset_img 04.png debian 17 %}
{% asset_img 05.png debian 18 %}

区域选择直接中国，当然键盘模式也选择汉语啦

{% asset_img 06.png debian 19 %}

之后安装程序就会开始检测硬件并安装临时安装系统（不会写入硬盘）

{% asset_img 08.png debian 20 %}
{% asset_img 09.png debian 21 %}

按照要求配置计算机名，当然你保持默认都行，注意使用的字符类型和格式，域名暂时就不配置了

{% asset_img 10.png debian 22 %}
{% asset_img 11.png debian 23 %}

配置一下Root密码，记住Root用户拥有最高的权限，在局域网中可以随意一点，如果将主机对外开放了，还是稍微设置的复杂一点会比较好,密码的配置基本上需要输入两次以防止因为输入失误导致的无法登陆

{% asset_img 12.png debian 24 %}
{% asset_img 13.png debian 25 %}
{% asset_img 14.png debian 26 %}
{% asset_img 15.png debian 27 %}

当然出于安全性考虑，系统会要求创建一个较低权限的普通用户，那我就随便建立一个了，同样的密码也需要输入两次

{% asset_img 16.png debian 28 %}

接下来安装程序会进行检测硬盘和别的设备，比如网卡什么的

{% asset_img 17.png debian 29 %}

由于使用的是网络安装镜像，安装过程需要联网进行，此处系统会询问你使用哪块网卡作为主网络接口，此处一般情况都是ens192，也是路由的WAN口，如果能够正常获取到IP，那就会自动进入下一步，如果不行的话就换一个网卡

{% asset_img 18.png debian 30 %}

网卡选好了，那就配置磁盘分区吧，这里使用了LVM，使用这个可以方便的在后期进行磁盘扩容，也许以后会搞个共享，那容量太小不能扩容就悲剧了，还是用LVM吧，比较灵活

{% asset_img 19.png debian 31 %}
{% asset_img 20.png debian 32 %}
{% asset_img 21.png debian 33 %}
{% asset_img 22.png debian 34 %}
{% asset_img 23.png debian 35 %}

选择哪个16G的硬盘，不用想太多，回车就是了，新手比较时候直接使用单个分区，这也是为啥建议使用LVM的原因，直接选择**分区设定结束并将修改写入磁盘**，最后确认的时候选择是就将进行自动分区并挂载

{% asset_img 25.png debian 36 %}

当询问你是否需要扫描其他的CD或DVD的时候，选择否，直接通过网络获取更新就已经足够了

{% asset_img 26.png debian 37 %}
{% asset_img 27.png debian 38 %}

要进行网络安装，肯定需要告诉程序从哪里下载安装文件，此处就就近原则，使用中国的地址，在这里可以看到有Debian官方的，也有清华大学，网易等，其实我想用阿里云的，可惜内置的并没有，此处的选择也会影响安装完系统后系统所使用的默认源地址

{% asset_img 28.png debian 39 %}

是否需要使用代理根据实际情况来，家里的网络质量还可以，而且使用了中国区的源，暂时就不配置代理了

{% asset_img 29.png debian 40 %}
{% asset_img 30.png debian 41 %}

组件从网络下载需要时间，具体取决于你的网络情况，不知道是不是网络抽风呢还是我最近在家配置IPV6的原因，下载速度不咋地

{% asset_img 31.png debian 42 %}

系统还会询问你是否要参加软件包流行度调查，个人是没有参加过，哪个小伙伴如果有时间，可以支持一下，此处我选择了**否**

{% asset_img 33.png debian 43 %}
{% asset_img 34.png debian 44 %}

虚拟机是用来配置路由器的，自然是不需要提供桌面支持的，所以此处勾选掉桌面，家里也没打印机，同样勾选掉，SSH工具还是很有必要的，勾选上，就按照这样的选择进行应用安装吧

{% asset_img 35.png debian 45 %}
{% asset_img 36.png debian 46 %}

再一次的需要下载一堆安装包，速度依然不咋地，后悔当初没有设个代理试试

{% asset_img 37.png debian 47 %}

出现这个画面的时候基本上系统已经安装完成了，就差个引导了

{% asset_img 38.png debian 48 %}
{% asset_img 39.png debian 49 %}

选择安装到虚拟硬盘，即`/dev/sda`，写入很快，之后就可以选择继续进行回车，至此系统就安装好了

#### 系统配置

> 安装完也不是马上就能使用的，还是需要适当调节一下的

##### 安装VMware Tools

VMware Tools作为虚拟机和主机之间通讯的桥梁，不仅能提高性能，还能对系统进行一些检测增强，特别是想要自动开关机功能的，这是必装组件

* 登陆系统

{% asset_img 40.png debian 50 %}
{% asset_img 41.png debian 51 %}

使用root登陆系统，密码就是安装时候设置的，此处输入密码是没有提示的，不要觉得没有输进去而不断尝试的去输入，直接输入回车即可

* 加载安装镜像

{% asset_img 42.png debian 52 %}

勾选 **虚拟机————客户机————安装/升级VMware Tools**，在光驱中加载安装镜像

{% asset_img 43.png debian 53 %}
{% asset_img 44.png debian 54 %}

```shell
mount /dev/cdrom /mnt
cd /mnt
ls
```

挂载镜像，并将光驱挂载到 **/mnt** 目录

* 解压缩安装包并进行安装

{% asset_img 46.png debian 55 %}
{% asset_img 47.png debian 56 %}
{% asset_img 48.png debian 57 %}

```shell
cp VMwareTools-10.2.5-8068406.tar.gz /tmp
cd /tmp
tar xzf VMwareTools-10.2.5-8068406.tar.gz
ls
cd vmware-tools-distrib

./vmware-install.pl
```

虽然把镜像进行了挂载，但其本身是不支持写入的，所以需要复制出安装包到一个可读写的目录进行操作，这里直接复制到临时目录进行安装；执行`./vmware-install.pl`安装命令后只需要按下**y**同意即可，其它的选项直接回车即可安装完成

{% asset_img 49.png debian 58 %}
{% asset_img 50.png debian 59 %}

本身软件包就不大，安装还是很迅速的，就是需要点不少次回车

{% asset_img 62.png debian 71 %}

安装完成之后就可以在虚拟机摘要部分看到虚拟机配置到的网卡IP信息了，基本上出现这个就说明**VMware Tools**安装成功了

##### 对终端进行个性化

由于没有安装图形界面，那命令界面将是我们长期面对的交互界面，当然喜欢它稍微美观点，好用点，那就稍微个性化一下吧

* 编辑配置文件

{% asset_img 51.png debian 60 %}

终端的配置文件一般在用户目录的根目录中，并且都是隐藏的，类Unix系统的隐藏文件基本上都是以“.”+“文件名”的形式存在的，在root目录下执行`vi .bashrc`

{% asset_img 52.png debian 61 %}
{% asset_img 53.png debian 62 %}

删除部分项目前的 **#** 符号即可启用这条功能，其中**alias**后设置自定义命令，今后可能会用到，按下**Esc**键，输入`:wq`后回车即可保存并退出

* 使配置文件生效

{% asset_img 54.png debian 63 %}

执行`source .bashrc`就可以让命令生效

##### 安装部分软件

> 由于包管理系统的存在，Debian安装常用应用很简单

{% asset_img 55.png debian 64 %}
{% asset_img 56.png debian 65 %}

```shell
apt-get update
apt-get install tmux htop lrzsz vim -y
```

上述命令将安装我常用的一些应用，每个人有自己的习惯，大家根据自己的喜欢安装即可，当然由于普通的终端对中文字符支持的问题，此处的中文说明都变成了方块，还是通过SSH登陆比较好

##### 允许root账户通过SSH登陆

> 由于root账户的权限太高，基于安全性考虑，默认情况下SSH都不会允许通过root帐号进行登录，那如果我就想要有root那样自由的权限呢，那就可以通过修改SSH的配置文件来启用这个功能

{% asset_img 57.png debian 66 %}

执行`vi /etc/ssh/sshd_config`打开SSH配置文件

{% asset_img 58.png debian 67 %}
{% asset_img 59.png debian 68 %}
{% asset_img 60.png debian 69 %}

通过`/Root`命令定位到关键参数处，删除前方的 **#** 符号，然后将后面的值改成 **yes** 后保存退出即可

{% asset_img 61.png debian 70 %}

执行`systemctl restart sshd`即可重启SSH服务并生效

### 总结

---

安装其实很简单，主要是由耐心，也不要对命令界面太排斥，当你习惯之后你会发现习惯了就回不去了，一条命令比鼠标点好多下效率高多了

后续就将开始正式的介绍Linux Router的搭建，文字版和视频版互补，希望能够得到大家的支持

### 历史

---

* **2019.07.18** 初稿
* **2019.07.20** 基本内容完成
* **2019.08.05** 细节修改
