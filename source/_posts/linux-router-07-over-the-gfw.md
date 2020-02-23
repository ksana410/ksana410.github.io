---
title: Linux路由补完计划07 翻越长城
tags:
  - Linux
date: 2019-09-16 23:42:21
---

## 前言

---

前期的准备工作已经基本上完成了，接下来就是翻越长城，世界那么美好，为啥不出去看看呢！本篇将利用近几年比较热门的[V2Ray](https://github.com/v2ray/v2ray-core)配合iptables实现自动翻墙，实现方式和openwrt上是一样的，只不过本文使用的是普通的Linux发行版而已。

<!-- more -->

## 构成

---

**V2Ray** V2Ray是[Project V](https://v2ray.com/)项目的核心组成，它是由一群爱好自由的有志之士共同开发的，至今已经发展出多种方案来应对GFW对翻墙的封锁，这也是在SS和SSR流量有嫌疑被精确识别的前提下最好的翻墙方案 **（当然，这只是个人的观点，不同地区GFW对不同工具会有不同的侧重）**

{% asset_img 01.png v2 01 %}

**iptables** Linux上的防火墙配置工具就无需过多介绍了吧，前期用来配置路由就是靠它，接下来在科学上网中它也将担任重要的角色

## 原理

---

> 本例中使用了V2Ray自身支持的一个功能协议，名字是 **[dokodemo-door](https://v2ray.com/chapter_02/protocols/dokodemo.html)** 官方的说明文档把每个参数都进行了说明，如果配置成功的话，基本上透明代理的效果会很好（前提你自己有一个比较高速的梯子，或者机场也行，最近支持V2Ray的机场好像变多了）
>
> 当然对于大部分人来说这样的说明真的会一头雾水，让人无从下手，还好官网直接提供了两个白话文教程，[白话文教程](https://toutyrater.github.io/app/transparent_proxy.html)、[新白话文教程（社区版）](https://guide.v2fly.org/app/transparent_proxy.html#%E8%AE%BE%E7%BD%AE%E6%AD%A5%E9%AA%A4)，社区版可能会更加容易让人接受一点

{% asset_img 11.png v2 02 %}

那它是怎么实现科学上网的呢？图不高兴画了，就直接用文字来表述一下吧，具体的过程主要包括如下几个部分：

### 发起访问

假设你想访问被墙的网站，你会对路由器发起一次通讯请求，此时浏览器会对默认的网关发送一些数据包，这些数据包包含了来源地址，域名信息等必要的信息；

### 数据接收部分

到达网关之后，此处就是我的软路由，数据包首先会被**iptables**（实际上并不是它，而是内核中的[Netfilter](https://zh.wikipedia.org/wiki/Netfilter)，此处只是为了方便说明而已，使用iptables这个防火墙配置工具代指Linux系统的防火墙部分）接收到，此时iptables就会对这些数据包进行分析匹配；

### 路由判断

iptables会根据数据包中所携带的目标域名或IP判断它的走向，首先是判断是否是本机或者本地局域网的地址，如果是的话就发送过去，如果不是的话就会通过我配置的防火墙规则转发给V2Ray的透明代理端口——dokodemo-door协议入口；

### V2Ray处理数据

如果是此时V2ray接收到了数据，它内置了一套路由系统，也会对数据包进行分析，如果数据包目标地址是国内的地址，那V2Ray不会对数据包进行处理，直接就按照原来的目标地址发送过去，而如果是国外的地址，就会通过预先配置好的梯子通道转发出去，这样就实现了科学上网，当然V2Ray的功能不止于此，迫于篇幅，本篇就不详细展开了。

## 开始配置

---

### 1.给系统配置代理

基本上没有把科学上网配置好之前，这一步都是必须的，此处需要说一句**感谢伟大的防火墙**

{% asset_img 06/png v2 03 %}

```shell
export http_proxy=http://172.16.1.10:1081
export https_proxy=$http_proxy
```

> **此处主要使用的是http代理，可以通过ss或者v2ray提供**

### 2.使用V2Ray官方一键安装脚本进行安装

V2Ray项目直接提供了官方的一键安装脚本，由于是使用GO语言编写的，除非自己喜欢折腾，基本上都不需要进行编译安装，一键脚本会自动从github上下载最新的编译好的二进制版本进行安装，同时它也会自动配置好自动启动，由于国内访问github的速度不理想，所以我在第一步的时候就配置好了代理以提高一键脚本的执行速度，直接执行`bash <(curl -L -s https://install.direct/go.sh)`

{% asset_img 07.png v2 04 %}

执行完成之后会自动随机生成端口和UUID，并配置好自动启动，此处使用的是debian，如果是其它发行版的话，建议使用systemd版本的，一键脚本对于init.d的自动启动配置支持的并不好

{% asset_img 08.png v2 05 %}

### 3.V2Ray配置文件说明

V2Ray的默认配置文件是`/etc/v2ray/config.json`，如果使用的是一键脚本的话，默认安装的是服务端配置，此处我打算建立一个本地的科学上网软路由，那么就需要将其修改为客户端模式，不过在这之前首先要对V2Ray的配置文件格式做一下简单介绍，打开配置文件`vi /etc/v2ray/config.json`，如下图所示：

```json
{
  "inbounds": [{
    "port": 36133,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "599a3694-2d63-45da-9738-a568d24c41c2",
          "level": 1,
          "alterId": 64
        }
      ]
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```

根据官方教程，整个配置文件应该由如下几个部分构成，但这些部分都是可以精简的，但有几个部分是必须的，如果是服务端的话，**inbounds**、**outbounds**、**routing**这三个部分是必要的；而如果是客户端的话，一般会多一个**dns**模块提供域名解析的支持，当然如果你对配置文件比较熟悉的话，可以通过增加更多的模块实现更多的功能，完整的结构如下：

{% asset_img 10.png v2 06 %}

**inbounds**这个部分作为主机的接收端，会监听进入系统的一些连接，一般情况下监听本机端口的都是对外提供服务的，此处也不例外，不管是服务端还是客户端，都是在**inbounds**处提供服务，它相当于是V2Ray的入口

**outbounds**进行对外连接，如果是在服务端，那么就是客户端流量的访问出口，同时也能对一些流量进行屏蔽，而在客户端，主要会提供与服务端进行对接的服务

**routing**可以说是V2Ray上最有用的一个部分，Linux可以通过防火墙配置相关的策略对数据流量走向进行调整，而此处的这个功能也可以对**inbounds**和**outboounds**处的数据流量进行调整，好像除了NAT功能它不能实现，其它的部分都可以，当然此处没有考虑到系统资源开销的问题

> 相对而言，这个配置文件还是比较复杂的，如果要深入讲的话受限于篇幅，好像不现实，本篇还是以实现透明代理为主吧。

### 4.修改配置文件开启透明代理

客户端部分的主体结构是这样的：

```json
{
  "inbounds": {},                                 //此处主要提供透明代理服务，计划监听在10000端口，同时也可以提供socks或者http代理等功能
  "outbounds": {},                                //主要用于对接vps上部署的V2Ray服务端
  "dns": {},                                      //对传入的带域名信息的请求进行解析，也能一定程度上提供抗污染的DNS解析
  "routing": {}                                   //对传入传出的数据流进行路由选择，国内流量走国内的，国外的走代理，如果是广告的话就进行屏蔽，亦或者一些其它的操作
}
```

> **为了方便理解，我会将每个部分分别配置一个实例**

#### 配置透明代理监听在10000端口

> 此处参照两个白话文教程：[白话文教程](https://toutyrater.github.io/app/transparent_proxy.html)、[新白话文教程（社区版）](https://guide.v2fly.org/app/transparent_proxy.html#%E8%AE%BE%E7%BD%AE%E6%AD%A5%E9%AA%A4)

作为客户端的话，**inbounds**部分将监听本地的一个或者多个端口，我会创建一个透明代理和一个socks5代理，socks5代理主要用于测试科学上网是否成功，废话不多说，直接开始吧。

```json
  "inbounds": {
    {
      "protocol": "dokodemo-door",               //协议类型选择dokodemo-door，透明代理就用这个
      "port": 10000,                             //监听的端口
      "settings": {
        "network": "tcp,udp",                    //传输协议，只支持tcp和udp，icmp是不支持的，所以用来ping被墙的ip也是没用的
        "followRedirect": true                   //此处必须是开启，否则它将不能识别由iptables转发来的流量
      },
      "sniffing": {
        "enabled": true,                         //此处可以开启流量侦测，一定程度上解决DNS污染
        "destOverride": ["http","tls"]           //流量侦测的协议类型，主要是网络上最常用的两种http和tls
      }
    },                                           //至此，透明代理所需要的数据入口就配置完成了
    {
      "protocol": "socks",                       //这是我所配置的socks代理协议，此处兼容socks5，socks4e，socks4，它们是什么？不用管，这样配置就对了
      "port": 1080,                              //socks默认监听的端口1080
      "listen": "0.0.0.0",                       //监听的地址，默认情况下是本机回环"127.0.0.1"，如果想要局域网内的设备也连接这个代理，使用"0.0.0.0"，此处需要搞明白"127.0.0.1"和"0.0.0.0"的区别
      "settings": {
        "auth": "noauth",                        //如果不需要对连接进行身份认证，那么就用"noauth"，一般情况下都是在局域网里面使用的，就用不认证吧！
        "udp": false                             //udp就不开启了，此处默认不开启，可以省略
      },
      "sniffing": {
        "enabled": true,                         //同样的开启流量侦测
        "destOverride": ["http","tls"]
      },
      "tag": "socksin"                           //为这个socks代理添加一个标记，方便后期在**routing**中进行调用
    }
  }
```

**inbounds**部分就结束了，此处主要是对局域网和本机提供服务，被动的对局域网内或者本机的连接进行响应；

#### 与远程服务器建立连接（与梯子建立连接）

> V2Ray对外连接的配置主要集中在**outbounds**部分，此处集中了梯子的连接，可以是一个或者多个，默认情况下如果没有在**routing**部分进行指定，系统都会使用由上至下第一个对外出口作为默认出口使用，所以排在第一个的具有举足轻重的地位，以下将会放上我自己的配置：

```json
  "outbounds": {
    {
      "protocol": "vmess",                       //对外的协议类型，一般与vps对接的话都是用的vmess协议，此处先不考虑这个协议是否靠谱，你只要把它当成是一种加密方式即可
      "settings": {
        "vnext": [
          {
            "address": "域名或IP",               //远程服务的域名或者IP，建议使用域名，可以通过tls进行混淆以欺骗gfw对数据流量的干扰，就是ssr曾经引以为豪的东西
            "port": 443,                        //tls一般都是用443端口的，此处我也使用了443端口
            "users": [
              {
                "id": "65a8ee27-b88b-0530-64d8-aad37d4980d8",     //uuid就类似于密码，可以使用`v2ctl uuid`命令生成，当然此处需要和服务端相同
                "alterId": 64,                  //此处也需要和服务端相同
                "security": "auto"              //此处的加密一般是在客户端指定的，服务端会根据客户端的设置情况自动进行相应的加密，但tls本身也是一种加密，个人认为如果使用了tls混淆的话此处直接none即可
              }
            ]
          }
        ]
      },
      "streamSettings": {                       //这个项目对通讯协议进行一些微调
        "network": "ws",                        //使用websocket连接
        "security": "tls",                      //tls混淆加密
        "wsSettings": {
          "connectionReuse": true,
          "path": "/ray"                        //当访问这个目录的时候使用websocket连接，由于官方手册中使用了这个目录，很容易被针对，个人建议改成一个字符串或者别的什么内容
        }
      },
      "tag": "proxy"                            //给这个出口加个标签
    },
    {
      "protocol": "freedom",                    //直连出口
      "settings": {},                           //没啥需要设置的
      "tag": "direct"                           //标签不要忘记，下面的路由部分会用到的
    },
    {
      "protocol": "blackhole",                  //黑洞，你只要敢来我就敢把你全吞了，一般用来做广告屏蔽等操作
      "settings": {},                           //依然不需要设置些啥
      "tag": "blocked"                          //标签不能忘
    }
  }
```

**outbounds**的配置大致上就是这样的，此处配置了三个出口

#### DNS服务器配置

> 如果没有添加DNS服务器的配置的话，V2Ray会自动使用系统配置好的地址，由于我已经解决了DNS污染，所以我完全可以在此处进行省略，但考虑到方案并不是只有一种，多一条路多一种选择，还是稍微举个例子吧！

```json
  "dns": {
    "servers": [
      "8.8.8.8",                                //谷歌的
      "9.9.9.9",                                //IBM的
      "localhost"                               //本机配置好的，如果是我的软路由的话污染已经解决了
    ]
  }

```

此处直接使用了最简单的配置，使用这样的配置的话，当用户使用socks5代理进行上网时，会优先使用谷歌和IBM的服务器进行域名解析，如果上述的服务器都挂了，那么就使用系统自带的DNS服务器进行解析，v2ray的内置DNS功能还是很强大的，受限于篇幅，此处就不展开了

#### 强大的路由功能

> 一般的科学上网工具除了提供分流外基本上不会再有别的功能了，而V2Ray可通过**routing**这个功能模块提供类似于**Linux防火墙**一样的功能，根据配置好的规则选择对外出口，负载均衡，识别连接，屏蔽广告，基于端口匹配等等，当然这些功能对于初学者而言简直就是天书，本次也不会详细去说明，但大家只需要知道一点，那就是善用**routing**模块

```json
  "routing": {
    "domainStrategy": "IPIfNonMatch",           //必要的选项，如果匹配不到就直接解析成ip
    "rule": [
      {
        "type": "field",
        "domain": [                             //匹配域名
          "geosite:category-ads-all"            //匹配常见广告域名
        ],
        "outboundTag": "blocked"                //利用标记将出口设置为“黑洞”，相当于对广告进行屏蔽
      },
      {
        "type": "field",
        "domain": [
          "geosite:cn"                         //匹配国内的域名
        ],
        "outboundTag": "direct"                 //满足这个规则的网址直连不做操作
      },
      {
        "type": "field",
        "ip": [
          "geoip:cn"                            //匹配国内的ip
          "geoip:private"                       //匹配常用内网的ip
        ],
        "outboundTag": "direct"                 //满足此规则的ip直连
      }
    ]
  }
```

以上的配置屏蔽了一些常用的广告域名，同时不代理内网IP、中国区IP、中国区域名、常见的没被墙的域名，`而没有手动指定的流量会使用默认的一条规则，这个默认规则不需要进行手动设置，默认是**inbounds**中配置的第一条出口`；这个模块下的匹配规则和Linux防火墙类似，逐条依次进行匹配，需要屏蔽的放在上面，之后的规则会依次匹配，V2Ray的规则匹配也是这样的顺序，由上至下，哪个在上面哪个先匹配，一旦匹配到就跳出规则，所以在配置路由的时候需要好好规划好顺序

#### 完整配置文件

```json
{
  "log": {
    "loglevel": "info",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [
    {
      "protocol": "dokodemo-door",
      "port": 10000,
      "settings": {
        "network": "tcp,udp",
        "followRedirect": true
      },
      "sniffing": {
        "enabled": true,
        "domainOverride": ["http","tls"]
      }
    },
    {
      "protocol": "socks",
      "port": 1080,
      "listen": "0.0.0.0",
      "settings": {
        "auth": "noauth",
        "udp": false
      },
      "tag": "socksin"
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "xxxxxxx",
            "port": 443,
            "users": [
              {
                "id": "65a8ee27-b88b-0530-64d8-aad37d4980d8",
                "alterId": 64,
                "security": "auto"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "connectionReuse": true,
          "path": "/ray"
        }
      },
      "tag": "proxy"
    },
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "dns": {
    "servers": [
      "8.8.8.8",
      "9.9.9.9",
      "localhost"
    ]
  },
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rule": [
      {
        "type": "field",
        "domain": [
          "geosite:category-ads-all"
        ],
        "outboundTag": "blocked"
      },
      {
        "type": "field",
        "domain": [
          "geosite:cn"
        ],
        "outboundTag": "direct"
      },
      {
        "type": "field",
        "ip": [
          "geoip:cn"
          "geoip:private"
        ],
        "outboundTag": "direct"
      }
    ]
  }
}
```

在上述的配置文件中我增加了一个log模块，主要是为了进行故障分析，在`/var/log/v2ray`目录下生成两个日志文件，方便遇到问题之后进行分析，整个配置文件除了科学上网部分没有配置外，其它都能实际使用，这个部分是可以当作模板进行使用的

当然使用vim进行文本编辑的话对于普通用户而言实在是太痛苦了，那不妨把它拷贝下来在本地计算机上面进行编辑，我使用的是windows平台上最好的文本编辑器之一的[vscode](https://code.visualstudio.com/)，首先先把配置文件下载到本地：

{% asset_img 18.png v2 07 %}

如果你使用的是xshell这个ssh工具的话，直接安装**lrzsz**这个工具即可，安装完之后执行`sz /etc/v2ray/config.json`，然后选择保存的位置就能将配置文件下载到本地了

{% asset_img 17.png v2 08 %}

如果没有使用上述工具的话，使用[winscp](https://winscp.net/eng/index.php)也是可以的，或者如果是windows 10操作系统的话，直接使用内置的scp命令也是可以将文件下载到本地的，打开powershell或者cmd，执行`scp root@172.16.1.50:/etc/v2ray/config.json d:\desktop\config.json.v2`，这样就复制到我的桌面了

{% asset_img 25.png v2 09 %}

之后就用**vscode**进行编辑，由于我下载下来的时候修改了文件名，在后面加上了一个v2后缀（这一步并不是必须的，如果直接下载下了json后缀的，就不需要以下步骤）

{% asset_img 26.png v2 10 %}
{% asset_img 27.png v2 11 %}
{% asset_img 28.png v2 12 %}

修改好配置文件之后就可以上传到服务器了，使用scp的话顺序改一下就好

{% asset_img 30.png v2 13 %}

`cat /etc/v2ray/config.json.v2 > /etc/v2ray/config.json`即可

当把配置文件修改好之后，记得检测一下文件格式是否有问题，执行命令`/usr/bin/v2ray/v2ray --test /etc/v2ray/config.json`如果没问题，直接保存退出（如果使用vscode进行编辑的话，基本上不会有多少问题），之后执行`systemctl restart v2ray`重启服务即可

### 5.配置防火墙规则

> 刚刚配置好的V2Ray只能提够一个透明代理的入口，它还没有那么强大控制数据流量走向，接下来的事情就交给防火墙吧，至于是什么原理，小孩子知道那么多干啥，照着做就是了

```shell
# 直接对tcp动手（不知道tcp是啥玩意的也不用在意，照着做就是了）
# 新建一条叫做v2的新链
iptables -t nat -N v2
# 忽略掉远程服务器的地址，避免出现环回
iptables -t nat -A v2 -d vps地址 -j RETURN
# 排除掉内网地址，防止访问局域网地址的时候也被代理
iptables -t nat -A v2 -d 172.16.0.0/12 -j RETURN
iptables -t nat -A v2 -d 192.168.0.0/16 -j RETURN
# 除已经被排除的地址，其它的全部交给V2ray进行处理，此处全部转发到了10000端口
iptables -t nat -A v2 -p tcp -j REDIRECT --to-ports 10000
# 这一句会将系统接受到的所有不是本机为地址的数据全部转发到v2链
iptables -t nat -A PREROUTING -p tcp -j v2
# 这句可以对本机的流量进行代理，个人觉得没有必要，有些时候反而容易出现问题
iptables -t nat -A OUTPUT -p tcp -j v2
```

{% asset_img 32.png v2 14 %}

> 请忽略中间发生的输入错误
> 此处我只配置了TCP流量部分，UDP部分我觉得普通用户暂时用不到，一般游戏用户会比较需要，但v2ray对游戏的支持并不是很好，如果有兴趣的话我会在下一篇文章中进行说明

执行上述命令之后呢，墙是什么？快去愉快的上网吧

## 配置完验证一下

---

### 验证对外的梯子是否通了

> 我在入口部分除了建立了透明代理，同时也配置了一个socks5代理，此处就用这个来验证一下对外的连接是否正常

`curl -x socks5://127.0.0.1:1080 google.com`这个命令执行之后如果能出现如下的界面就说明对外的梯子出口是连接成功的，V2Ray可以提供科学上网的功能，此处没问题的话即可进行下一步，如果有问题的话进行下述的排查：

{% asset_img 31.png v2 16 %}
{% asset_img 33.png v2 15 %}

#### **服务器和客户端之间的时间差不能超过一分钟**

查看两者之间的时间差，如果超过了一分钟就无法正确的建立连接，可以通过自动同步时间解决

#### **查看配置文件中的域名，uuid，端口号等有没有存在差异**

仔细对比服务端和客户端之间的配置项目，常见的错误主要是uuid和端口号不对应

### 透明代理测试

> 要测试其实很简单，打开一台电脑，通过自建的软路由上网，看看能不能科学上网就可以判断出来了

此处我依然使用我一直使用的manjrao进行测试，打开火狐，打开谷歌等不存在的网站，看看能否正常打开，然后再放个[YOUTUBE](https://youtube.com)看是否可以正常播放，如果没问题的话那就说明你配置成功了，接下来该干嘛干嘛了，世界离你就差延时那点距离了

{% asset_img 34.png v2 17 %}

> 打开[ip111.cn](http://ip111.cn)确认一下是否能够正常访问外网，只要后面两个显示的是你vps地址就说明是正常了

{% asset_img 35.png v2 18 %}

## 规则的自动加载

---

防火墙规则过去在网卡的开关过程中就能自动进行保存和加载，如果怕防火墙规则丢失的话，建议手动保存一下防火墙规则

`iptables-save > /root/iptables.rule`就可以了，这样开机就会自动加载了，当然你如果搞错了防火墙的输入顺序就有可能会出问题了

## 总结

---

本篇内容中涉及到了比较多的V2Ray配置文件实例，虽然看上去很多，但并没有深入去分析每一个语句的作用，大家看看就好，直接使用最后的实例即可，如果想要深入的了解的话，我会在下篇做出比较详细的分析，先劝个退，不认真学习的话真的会搞不懂的哦，越看越迷糊都有可能哦！

对比两年前初次使用V2Ray，感觉官方文档完善了好多，但依然有很多坑需要去解决，下期预告，对于配置文件的详细分析（希望自己的拖延症不会像这次这样严重）！

## 历史

---

* **2019.09.16** 初稿
* **2019.09.25** 根据测试结构完善大纲
* **2019.10.18** 磨磨蹭蹭的把坑填上，应该是可以上线了
