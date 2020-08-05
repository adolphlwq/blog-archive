# install wireless firmware on archlinux
**编者按**：这篇文章介绍了笔者解决archlinux+kde plasma环境无法链接无线网的问题。笔者通过查阅archlinux wiki与Google定位问题，重新安装Linux无线网卡固件解决了问题。

## 背景介绍
- [hp440](http://item.jd.com/3690967.html),i54200U 64bit.
- archlinux 20170101.iso
- kde5 plasma

## 问题描述
安装NetworkManager（负责提供网络功能的服务，简称`nm`）和[network-manager-applet](https://www.archlinux.org/packages/?name=network-manager-applet)，network-manager-applet适用于使用plasma桌面的场景，它是一个托盘程序，通过图形化的界面管理计算机的网络，下面简称`nm-applet`，正确启动nm后，点击nm-applet图标后只显示**有线链接**，不显示可用的无线网列表。

![](/content/images/2017/01/archlinux-wireless-01.jpg)

即使自己手动添加的`linkernetworks2`也无法激活。

## 解决步骤

### 参考archlinux wiki cn
[archlinux wiki-NetworkManager (简体中文)](https://wiki.archlinux.org/index.php/NetworkManager_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))主要介绍了：

- archlinux上NetworkManager的安装与使用
- GUI的安装

这部分信息适合安装archlinux后配置基本的网络链接。不适用无线网络的debug。

### archlinux上无线网络的配置
先在[kde-cn频道](https://userbase.kde.org/IRC_Channels/zh-cn)上提问并没有解决自己的问题。

又参考[Wireless network configuration (简体中文)](https://wiki.archlinux.org/index.php/Wireless_network_configuration_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

- 查看pci设备没发现问题
```bash
$ lspci -k
09:00.0 Network controller: Broadcom Limited BCM43228 802.11a/b/g/n
        Subsystem: Broadcom Limited Device 05e2
        Kernel driver in use: bcma-pci-bridge
        Kernel modules: bcma

```

- 查看网卡设备，发现没有无线网卡的信息。
```
$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp8s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 5c:b9:01:f6:3b:18 brd ff:ff:ff:ff:ff:ff
```

以上两步发现没有无线网卡的设备信息。猜测有两个：

1. 无线网卡设备没有开启，（在笔记本上）手动开启
2. 无线网卡坏了

自己尝试在[hp上开启网线网卡](http://support.hp.com/cn-zh/document/c02762471)发现没用。**这是我有点慌，难道网卡坏了？宝宝刚申请的笔记本就坏了无线网卡，蓝瘦香菇！**

安装archlinux就像堆积木，许多东西都需要自己适配，并且这时我发现了[archlinuxcn-bbs找不到无线网卡[已解决]](https://bbs.archlinuxcn.org/viewtopic.php?id=3187)，参考对方的解决方案感觉是自己没安装对固件(firmware)。

最后定位了自己的问题：**内核中无线网卡固件和无线网卡硬件不一致**。找到问题后参考wiki中的链接正确安装固件后重启笔记本解决问题。

## 有用的参考
- [Wireless network configuration (简体中文)](https://wiki.archlinux.org/index.php/Wireless_network_configuration_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
- [archlinuxcn-bbs找不到无线网卡[已解决]](https://bbs.archlinuxcn.org/viewtopic.php?id=3187)
- [Linux wireless b43 firmware-archive](http://linuxwireless.sipsolutions.net/en/users/Drivers/b43/) 这个链接**详细介绍**在Linux上安装和硬件版本一致的firmware的过程。
- [Linux wireless b43 firmware](https://wireless.wiki.kernel.org/en/users/drivers/b43)