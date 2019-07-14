---
title: openwrt内核编译
date: 2019-06-27 01:18:02
tags:
---

## openwrt内核编译

**OpenWRT**的内核是Linux，Linux内核是操作系统的核心，也是操作系统最基本的部门

<!-- more -->

---

## 在开始之前，首先你要知道自己在做什么，打算实现什么样的功能

## openwrt内核配置的构成

**OpenWRT**配置内核主要在两个位置，模块配置主要在**make menuconfig**中配置，而内核功能配置需要进入单独的一个界面中进行，开启命令是**make kernel_menuconfig**

### 内核模块配置命令

``` shell
make menuconfig
```

{% asset_img 1.png openwrt_kernel_building_1 %}

标准配置界面中，涉及到内核编译的主要是**linux modules**，其实这也不算是内核的编译，此处进行的配置主要是会生成一些内核模块，这些模块并不会编译进内核，但是会在系统开机的时候自动加载，这些模块有些是驱动，有些是加速模块，或者一些其他的功能，由于linux是一个宏内核，如果你愿意的话可以把所有市面上的硬件设备都在内核中集成相应的功能和驱动，但是，这样做的话内核会非常庞大，故此处对其进行内核的配置和裁剪

```shell
Block Devices //块设备支持
CAN Support
Cryptographic API modules //加密算法支持
Filesystems  //文件系统支持
FireWire support
Hardware Monitoring Support
I2C support
Industrial I/O Modules
Input modules
LED modules
Libraries
Native Language Support
Netfilter Extensions
Network Devices
Network Support
Other modules
PCMCIA support
SPI Support
Sound Support
USB Support
Video Support
Virtualization
Voice over IP
W1 support
WPAN 802.15.4 Support
Wireless Drivers
```
