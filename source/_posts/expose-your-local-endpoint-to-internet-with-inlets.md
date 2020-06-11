---
title: 又一款内网穿透工具——inlets
tags:
  - Tools
  - Linux
date: 2019-08-27 23:01:48
---

## 前言

---

随着IPV4地址的不断匮乏，现在很多电信运营商早就开始将用户的上网IP私网化，即我们常说的内网地址，这样的地址是经过NAT的，很多用户通过运营商提供的内网地址共享一个公网地址进行上网，运营商的IP资源节省了，普通用户也感觉不出什么区别，但对于需要外部访问家中或者公司中一些资源的用户来说，这就是灾难，也许有人说你可以打电话投诉来获取公网IP，电信联通可能还行得通，对于大内网的移动就无法实现了，为了解决访问内部资源的这个问题，内网穿透工具应运而生，内网穿透工具也如雨后春笋一般扎堆出现了**Ngrok，Frp，Serveo**乃至已经商业化很成功的花生壳，这类工具虽然已经很成熟了，但这并不是本篇分享的重点。通过github的推荐，我发现了一个有趣的内网穿透工具——[inlets](https://github.com/alexellis/inlets)

<!-- more -->

## 为啥推荐它

---

有时候第一眼的感觉很重要，它就是那个吸引我的工具，虽然我现在的主力依然是Frp，但我真的觉得inlets值得一试

inlets 结合了反向代理和 websocket 隧道，通过出口节点将内部的http服务暴露给公网。出口节点可以是 VPS 或具有 IPv4 IP 地址的任何其他计算机，与SSL结合使用时inlets可以与任何支持CONNECT的内网HTTP代理一起使用

## 它能做啥

---

> 开发者[Alex Ellis](https://github.com/alexellis)更新还是很频繁的，由1.x升级到2.x之后，作者直接声称其完全可以胜任生产环境，只不过在部署之前建议先测试一下

* 基础功能：

1.根据客户端设置在远程服务器上创建端口
2.基于域名进行端口复用
3.利用**SSL over websocket**进行安全的加密通信
4.断线重连
5.支持身份认证
6.多平台支持
7.与Docker和Kubernetes集成
8.原生跨平台支持，包括ARMHF和ARM64架构
9.除HTTP(s)以外，还支持在隧道内传输Websocket流量

* 计划功能支持：

1.自动配置DNS/A记录
2.基于 Azure ACI 和 AWS Fargate，以 Serverless 容器的方式运行「出口节点」。
3.通过 DNS01 challenge 使用 LetsEncrypt Staging 或 Production 签发证
...

虽然此时可以穿透的协议还只局限于HTTP或者HTTPS?，但作者还是愿意在今后的更新中添加对纯TCP流量的支持；未来肯定会越来越美好，有能力的小伙伴不妨也为其提供自己的一份力量

## 运行原理

---

### 原理图

{% asset_img 01.png inlets 01 %}

### 流程说明

* 假设将服务器的域名设定为"www.site.com"，此时inlets作为服务端监听在服务器的80端口，根据客户发起的协议匹配不同规则
* 而客户端使用websocket协议对服务端发起通讯，通讯可以使用加密或者不加密两种模式进行，这完全取决于服务端是否使用SSL加密
* 客户端和服务器通过共享密钥进行身份认证，认证通过即建立起了连接
* 此时，客户端设定的内网地址和端口就和服务端建立起了一条隧道
* 隧道建立后，服务端的80端口就相当于映射了客户端本地的127.0.0.1:3000
* 当用户访问"www.site.com"这个网址时，就相当于访问内网中的3000端口

稍微概括一下就是类似于如下的流程

客户端和服务端之间通信：

本地应用端口 <==http(s)/ws(s)==> inlets(client) <==ws(s)==> inlets(server) <==http(s)==> 用户浏览器

## 实例

---

> 光讲原理好像也没啥意义，那就用个人的一个实例来说明吧！

### 实例说明

> inlets之间使用websocket进行通讯，如果需要证书加密的话，即ssl加密，那中间就需要套接一个web服务器，它可以是nginx，也可以是caddy，此处使用配置相对简单的caddy进行演示，当然选择它的还有一个原因就是它支持自动签发证书，由于google的“恶意推广”，https网站将成为主流，给网站配置个证书也不错

### 先决条件

* 需要拥有一台拥有公网IP的主机，如果主机在国内，并且还需要开放80端口和443端口，还需要解决备案的问题
* 拥有一个域名，并且可以随意修改DNS解析，免费的，收费的都可以
* 耐心很重要，由于涉及到两台主机的操作，暂时还没有一键脚本可以使用
* 服务端和客户端只能使用64位的系统，当然现在也已经支持arm架构的硬件设备进行安装了

### 实验环境

> 也许这就是我自己的瞎折腾，但是希望能给大家以启发

* 域名：etspace.xyz
* 服务器公网ip：~~165.227.56.252~~
* 本地客户端主机：~~[NanoPi NEO2](http://wiki.friendlyarm.com/wiki/index.php/NanoPi_NEO2/zh#.E4.BD.BF.E7.94.A8.E5.BC.80.E6.BA.90.E7.A4.BE.E5.8C.BA.E4.B8.BB.E7.BA.BFBSP)~~
* 需要对外发布的服务：自用黑裙登录页面以及软路由netdata状态页

#### 配置DNS解析

vps主机肯定是有的，此处就不写怎么购买了，现在进入你的DNS管理后台，修改域名的解析地址，我使用的是[he.net](https://he.net)，此时进入后台，添加相应的解析记录

此处需要设置三个解析记录：

|域名|解析类型|计划用途|
|:------:|:------:|:------:|
|inlets.etspace.xyz|A|用于主控制通信|
|qh.etsapce.xyz|A|映射本地群晖登录界面|
|netdata.etspace.xyz|A|映射本地netdata状态界面|

#### 服务端上inlets的安装和配置

由于inlets是使用go语言进行编写的，只需要下载对应的二进制文件即可，进入release中选择合适的平台进行下载，重新登录项目地址，发现作者已经增加了不少的功能，比较可惜的是全功能的内网穿透工具inlets-pro并不是免费的，如果有能力还是支持一下作者，而对于普通用户而言，inlets已经可以满足大部分功能了

* 安装inlets

官方一键命令`curl -sLS https://get.inlets.dev | sh`

> 此处也可以使用作者新增了一个工具[inletsctl](https://github.com/inlets/inletsctl)，并且能通过这个工具进行更新及配置，一般情况我使用它来进行intels的安装，不再需要每次登录github进行下载更新了
>
> * 安装inletsctl
>
> 使用官方命令一键安装`curl -sLSf https://inletsctl.inlets.dev | sh`
此处图片----
>
> * 通过inletsctl 下载安装inlets
>
> `inletsctl download inlets`进行安装，inlets会被自动安装到`/usr/local/bin/`目录下，此时就可以适应inlets命令了

##### 在执行后续的操作之前，首先建议了解一下inlets的命令和参数

> 通过help命令直接查看

```shell
inlets --help

Inlets combines a reverse proxy and websocket tunnels to expose your internal
and development endpoints to the public Internet via an exit-node.

An exit-node may be a 5-10 USD VPS or any other computer with an IPv4 IP address.
You can also use inlets to bridge connect between private networks.

It is strongly recommended to put a reverse proxy with TLS/SSL enabled such as
Nginx or Caddy in front of your inlets server to enable an encrypted tunnel.

See: https://github.com/inlets/inlets for more information.

Usage:
  inlets [flags]
  inlets [command]

Available Commands:
  client      Start the tunnel client.
  help        Help about any command
  server      Start the tunnel server.
  version     Display the clients version information.

Flags:
  -h, --help   help for inlets

Use "inlets [command] --help" for more information about a command.
```

```shell
inlets server --help
Start the tunnel server on a machine with a publicly-accessible IPv4 IP
address such as a VPS.

Example: inlets server -p 80
Example: inlets server --port 80 --control-port 8080

Note: You can pass the --token argument followed by a token value to both the
server and client to prevent unauthorized connections to the tunnel.

Usage:
  inlets server [flags]

Flags:
  -c, --control-port int             control port for tunnel (default 8080)
      --disable-transport-wrapping   disable wrapping the transport that removes CORS headers for example
  -h, --help                         help for server
  -p, --port int                     port for server and for tunnel (default 8000)
      --print-token                  prints the token in server mode (default true)
  -t, --token string                 token for authentication
  -f, --token-from string            read the authentication token from a file
```

```shell
inlets client --help

Start the tunnel client.

Example: inlets client --remote=192.168.0.101:80 --upstream=http://127.0.0.1:3000
Note: You can pass the --token argument followed by a token value to both the server and client to prevent unauthorized connections to the tunnel.

Usage:
  inlets client [flags]

Flags:
  -h, --help                help for client
      --print-token         prints the token in server mode (default true)
  -r, --remote string       server address i.e. 127.0.0.1:8000 (default "127.0.0.1:8000")
  -t, --token string        authentication token
  -f, --token-from string   read the authentication token from a file
  -u, --upstream string     upstream server i.e. http://127.0.0.1:3000
```

* 生成预共享密钥

根据上述的帮助信息，需要注意 **-t/-f** 这两个参数，服务端和客户端之间需要一种方式来进行身份认证，谁都能接入那就不乱套了！此处使用预共享密钥来进行认证，`-t`参数对应一个字符串，`-f`参数对应一个文件，其实就是把密钥存储到一个文件里就行了

基于项目的说明，直接使用系统的随机数进行生成即可，执行`echo $(head -c 16 /dev/urandom | shasum | cut -d" " -f1)`，所生产的那串字符就是密钥，你可以直接复制了用，或者存储成一个文件，那样下载到本地就可以给客户端使用了

* 启动服务端进行连接测试

直接在tmux中运行，执行命令`inlets server -t ssssss -p 8000`，然后快捷键`ctrl+b`然后按一下`d`就能将其切换到后台，此处`server`这个参数决定了以服务端模式运行，客户端就是`client`

服务端启动了，那么本地客户端也可以启动了，在本地的nanopi上也安装好inlets，测试连接一下服务端，执行：

`inlets client -r 165.227.56.252:8000 -t ssss -u http://172.16.1.2:5000`

这条命令的意思是inlets以客户端模式运行，远程连接地址是165.227.56.252，端口8000，密钥是ssss，映射本地的<http://172.16.1.2:5000> 这个地址,此时在浏览器上输入165.227.56.252:8000就相当于访问本地的<http://172.16.1.2:5000> ，此处是我群晖的登陆界面

> 此处成功进行了连接，也实现了内网穿透，但这样的效果并不是我们所想要的，只能穿透一个算什么牛逼的工具，大家不要着急，请看我后续的操作

* 实现端口复用和多终端

单靠inlets本身实现这些功能并不现实，但我们可以拉来一个小伙伴做辅助即可，官方的实例使用的是[caddy](https://caddyserver.com/v1/)，那我们也用这个吧，有了它，证书的事情也可以迎刃而解，至于它该怎么安装，并不是本文的重点，就不写了（请允许我偷懒一下）

> 这里需要注意的是caddy此时使用的是**v1**版本，配置文件也是基于此版本，v2版本的配置上有比较大的改变，暂时还没把wiki看明白，有兴趣的朋友可以去看看，谷歌搜索caddy第一个出来的就是**v2**版本，大家一定要注意哦！

{% asset_img 02.png inlets 02 %}

> 将inlets配置成服务并且开机启动

其实就是写个开机脚本，直接在`/etc/systemd/system/`目录下新建`inlets.service`文件，写入如下内容

```shell
[Unit]
Description=Inlets Server Service
After=network.target

[Service]
Type=simple
Restart=always
RestartSec=1
StartLimitInterval=0
EnvironmentFile=/etc/default/inlets  #注意一下这条配置
ExecStart=/usr/local/bin/inlets server --port=8000 --token="${AUTHTOKEN}"

[Install]
WantedBy=multi-user.target
```

> * 此处使用了一个变量`${AUTHTOKEN}`,这个指代的就是上面生成的那串密钥，本着偷懒的本性，我直接将其加入了系统变量，那么如何加入进去呢？这就要使用这个启动脚本中所设定的变量文件`/etc/default/inlets`，直接执行`echo "export AUTHTOKEN=$(head -c 16 /dev/urandom | shasum | cut -d" " -f1)" > /etc/default/inlets`即可，不放心可以查看一下文件中的内容
>
> * 端口号可以看情况自己设定，如果不指定的话默认就是8000，但一定要和后续编写的caddyfile文件相同
>
> * 客户端的启动脚本也可以参照这样编写

后续的工作就是设置允许开机启动，并且让inlets在后台运行

```shell
systemctl enable inlets
systemctl start inlets
```

* 编写CaddyFile文件

> 之所以使用caddy完全是因为其两个优点：其一，配置简单；其二，可以自动申请证书，怎么使用我就不再赘述了，想要完全发挥它的功能，可以去查看[官方文档](https://caddyserver.com/v1/docs)，这里的所有配置主要针对inlets进行配置

inlets的服务端和客户端之间是通过[websocket](https://zh.wikipedia.org/wiki/WebSocket)进行连接的，所以caddy在这里需要将这个连接转发到inlets上，同时还要监听客户端对目的网址的http或者https访问请求，同样需要将连接转发到inlets上，那么配置文件该怎么写呢？

```txt
inlets.etspace.xyz {
  tls ddxiong0410@gmail.com
  proxy / 127.0.0.1:8000 {
    transparent
  }
  proxy /tunnel 127.0.0.1:8000 {
    transparent
    websocket
  }
}

qh.etspace.xyz {
  tls ddxiong0410@gmail.com
  proxy / 127.0.0.1:8000 {
    transparent
  }
}

netdata.etspace.xyz {
  tls ddxiong0410@gmail.com
  proxy / 127.0.0.1:8000 {
    transparent
  }
}
```

此处可以发现主控的域名下会比其它两个域名多了点内容，主要是需要将websocket的连接请求透明转发到inlets的监听端口上，默认inlets客户端模式会通过websocket协议访问域名的/tunnel目录，当caddy接受到这样的连接请求是就会透明转发到inlets的8000端口上，然后服务端和客户端通过预共享密钥进行认证，之后的事情就顺理成章了～

主控域名只需要一个，它承载了客户端和服务端之间的主要的流量，其他设置的域名都想到于是内网对外映射的入口，按照需要进行添加和设置即可

> 总体而言，inlets所需要的加密和对外连接主要需要一个主域名加上若干个服务域名构成（当然，如果只需要映射一个服务的话，主域名即可以当认证端又可以当映射入口，上述配置文件中<inlets.etspace.xyz>域名下所设置的两个**proxy**选项就是以此为目的设置的

配置文件写好了，重启一下caddy，`systemctl restart caddy`，此时caddy会自动为配置文件中的三个域名申请[Let's Encrypt](https://letsencrypt.org/zh-cn/)证书，有了证书加持，用户访问这个入口的数据也会是加密的，提高一定的安全性

* inlets客户端配置

在本地的nanopi上编写启动文件，参照一下服务端的配置，个人使用的如下：

```service
[Unit]
Description=Inlets Client Service
After=network.target

[Service]
Type=simple
Restart=always
RestartSec=1
StartLimitInterval=0
EnvironmentFile=/etc/default/inlets
ExecStart=/usr/local/bin/inlets client --remote="${REMOTEHOST}" --token="${AUTHTOKEN}" --upstream="${UPSTREAM}"

[Install]
WantedBy=multi-user.target
```

> inlets由服务端模式改成客户端模式，相应的参数也有一些变化，**token**的部分不变，多了**remote**和**upstream**，同样的，我依然会在变量文件中对他们进行设置，这样也是为了方便修改和重启

直接编辑`/etc/default/inlets`文件，此时的内容如下：

```shell
export AUTHTOKEN=ssssss
export REMOTEHOST=wss://inlets.etspace.xyz
export UPSTREAM="qh.etspace.xyz=http://172.16.1.2:5000,netdata.etspace.xyz=http://172.16.1.10:1999"
```

> 由于证书的加入，客户端和服务端之间使用了加密通讯，那么协议类型就变成了wss，而映射的本地服务需要将域名和本地ip以及端口号一一对应，有多少个就加多少个，各个项目直接通过`,`隔开

编辑完上述的文件后，就可以将inlets设置成开机启动，然后进行一下测试了

```shell
systemctl enable inlets
systemctl start inlets
systemctl status inlets
```

* 测试一下

直接打开<https://qh.etspace.xyz>看看是什么样的，基本上就可以很顺利的在外面打开家里的群晖登录界面，<https://netdata.etspace.xyz>也一样，那么如果需要建立更多的映射，就按照这样的流程进行配置即可

## 总结

---

inlets可能功能上没有Frp强，但却是一个很好玩的工具，对于一些只需要进行网页端控制的用户而言，不仅做到了对外映射，还能配合caddy进行证书加密，体验还是很不错的，端口复用，也不用担心会和v2ray这样的科学上网发生冲突，或者说，由于inlets的加入也有可能增加一定的混淆能力，毕竟访问这些域名所获取到的可是实打实的一些网站～

## 历史

---

* **2019.08.27** 建立初稿
* **2019.11.30** 稍微更新点东西
* **2020.02.22** 从冬眠中复苏，决定稍微写点什么
* **2020.02.29** 还是要更新的
* **2020.03.06** 加上图片应该就完成了
