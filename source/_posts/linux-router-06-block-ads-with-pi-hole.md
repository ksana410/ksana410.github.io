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

* 确认一下网络设置情况，保证能够正常上网

{% asset_img 12.png pihole 14 %}

{% asset_img 13.png pihole 15 %}

* 之后系统会询问是否需要安装web界面，小白必备，自然是需要的

{% asset_img 14.png pihole 16 %}

{% asset_img 16.png pihole 17 %}

* 开启查询记录，此处我选择了记录全部内容

{% asset_img 18.png pihole 18 %}

{% asset_img 19.png pihole 19 %}

之后Pi-hole就会基于刚刚的配置开始自动安装，这期间需要下载一些组件，如果网络不好的话就有可能会失败，所以代理还是很有必要的

{% asset_img 20.png pihole 20 %}

{% asset_img 24.png pihole 21 %}

{% asset_img 26.png pihole 22 %}

接下来就进入最后的一部分配置，包括防火墙，登陆密码等

* 配置防火墙（如果没有启用防火墙，这边可以随意），建议安装Pi-hole需要用到的防火墙规则，默认会打开其所需要用到的端口:`53`、`67`、`4711~4720`、`80`

{% asset_img 27.png pihole 23 %}

{% asset_img 39.png pihole 24 %}

* 完成安装及显示登录密码，之后可以通过显示的登陆地址和登陆密码进行后台管理

{% asset_img 37.png pihole 25 %}

{% asset_img 38.png pihole 26 %}

## 后台管理

---

### 登陆

登陆地址在安装完之后会自动显示，其实就是两个网卡所对应的地址，默认直接输入<http://ip/admin>即可进入界面，如果没有在IP后面加上`/admin`，那默认会进入如下界面：

{% asset_img 40.png pihole 27 %}

直接点击`Did you mean to go to the admin panel`即可进入后台界面，之后看到的就是基本的状态界面，上面可以直观的看到请求数，被拦截的查询等

{% asset_img 41.png pihole 28 %}

这个界面只是状态展示，如果要进行管理，需要登录进去，点击左侧的**Login**进行登录，密码在Pi-hole安装完之后已经显示在界面上了

{% asset_img 45.png pihole 29 %}

如果嫌自动生成的密码太难记，直接可以使用`pihole -a -p`命令进行修改，执行命令后输入两次新密码即可

{% asset_img 44.png pihole 30 %}

登陆之后所有功能都会在左侧显示，状态界面显示的内容也会更丰富

{% asset_img 46.png pihole 31 %}

### 功能配置

> Pi-hole的大部分功能配置都是在系统设置中调整的，点击左侧的**Settings**选项，出现系统配置的几个选项卡

{% asset_img 47.png pihole 32 %}

* **上游DNS调整**

选择**DNS**选项卡，在安装过程中我使用了谷歌的DNS服务器，此时我依然打算使用DNSCrypt-Proxy作为主要的上游服务器，取消谷歌的服务器的勾选，在**Custom 1(IPv4)**中填入DNSCrypt-Proxy的监听地址及端口，此处是`127.0.0.1#5353`

{% asset_img 48.png pihole 33 %}

点击右下角的**SAVE**之后即可生效，同时在`/etc/dnsmasq.d/`目录下的配置文件也会出现相应的修改，由于pihole-FTL就是一个dnsmasq的增强版本，所以dnsmasq上使用的语法在它身上依然可以使用，`vi /etc/dnsmasq.d/01-pihole.conf`

{% asset_img 49.png pihole 34 %}

> 此处可以看到服务器部分已经是自定义的那个服务器地址了

* **启用DHCP功能**

选择**DHCP**选项卡，勾选**DHCP server enabled**，并配置好起始和结束的IP地址，当然，也不要忘记把网关地址设置正确，同样的，保存之后就可生效，相应的在`/etc/dnsmasq.d`目录下也会生成对应的配置文件

{% asset_img 50.png pihole 35 %}

此时DHCP功能的配置文件名是`02-pihole-dhcp.conf`

{% asset_img 51.png pihole 36 %}

* **添加合理的广告过滤规则**

点击**Blocklists**查看默认的广告过滤规则，这些广告过滤规则并不符合中国用户的使用习惯，此处我建议全部取消前面的勾选

{% asset_img 52.png pihole 37 %}

此处我推荐一个广告过滤规则的Github项目：[neohosts](https://github.com/neoFelhz/neohosts)

{% asset_img 53.png pihole 38 %}

项目提供的几种类型的规则地址，我主要推荐如下两个，这两个任选其一即可，具体的不同可以查看项目的说明

{% asset_img 54.png pihole 39 %}

直接复制规则的地址，填入**Blocklists**选项卡的规则地址位置，点击**Save and Update**使其生效即可

{% asset_img 55.png pihole 40 %}

{% asset_img 56.png pihole 41 %}

> 此时添加的规则已经可以满足轻量级的广告过滤需求了，当然，如果想要更加强力点的广告过滤能力，建议移植[Adblock plus](https://adblockplus.org/)中的Easylistchina等规则，由于Pi-hole并不兼容Adblock plus的规则语法，所以只能移植其中收集到的广告域名，虽然功能上受限制了，但是也是对原有广告过滤功能的增强

在`/opt`目录下新建一个脚本文件`adblock.sh`，输入如下内容：

```shell
curl -s -L https://easylist-downloads.adblockplus.org/easylistchina+easylist.txt https://easylist-downloads.adblockplus.org/malwaredomains_full.txt https://easylist-downloads.adblockplus.org/fanboy-social.txt > adblock.unsorted

sort -u adblock.unsorted | grep ^\|\|.*\^$ | grep -v \/ > adblock.sorted

sed 's/[\|^]//g' < adblock.sorted > adblock.hosts

rm adblock.unsorted adblock.sorted
```

{% asset_img 57.png pihole 42 %}

保存并退出，然后执行`bash ./adblock.sh`，稍等片刻，之后就会生成一个包含大量域名的文件`adblock.hosts`

{% asset_img 58.png pihole 43 %}

同样的在**Blocklists**中填写规则地址，由于`adblock.hosts`文件是本地文件，所以规则的地址需要变化一下，将`http(s)://`替换为`file://`，之后在后面跟上文件的完整路径即可，此处的规则地址是`file:///opt/adblock.hosts`

{% asset_img 60.png pihole 44 %}

* **DNS查询分流**

这个功能在上一期中已经介绍过，在`/etc/dnsmasq.d`目录下引入[dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list)的配置文件即可，这里我依然使用软链接的方式进行添加，添加完成之后直接执行`pihole restartdns`即可重启Pi-hole服务

{% asset_img 62.png pihole 45 %}

## 总结

---

至此Pi-hole算是搭建完成了，虽然广告过滤是使用了DNS解析过滤的方式，效果并不是特别好，但好处是不会影响网速，相比于浏览器插件而言，各有优缺点吧！

如果想要自定义一些域名的处理方式的话，在左侧的**Whitelist**和**Blacklist**中可以手动进行调整，具体的就不展开了，大家有兴趣去自己尝试一下

## 历史

---

* **2019.08.29** 建立初稿
* **2019.09.04** 完成内容
