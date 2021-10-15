---
title: 搭建家庭简易DNS服务器
date: 2021-10-24 20:58:13
categories:
  - 消费电子
tags:
  - OpenWrt
  - LEDE
  - Docker
  - 宽带
  - DNS
---
大家好，本文记录了我搭建家庭DNS服务器的思路和手法，假如您需要快速、相对少广告，尽量不被画像（profiling）的互联网，也许这篇文章能帮到您。
假如您不知道DNS服务器是什么，那么这篇文章不适合您。

<!-- more -->

阅读本文需要了解如下基本概念：
- [x] DNS
- [x] Docker

动手操作需要：
- [x] 宽带接入
- [x] 机场
- [x] 一台能~~刷入~~安装OpenWrt(版本20或以上)的设备，ARM或者x86_64都可以
- [ ] 一台有SSH客户端的电脑

**阅读本文代表您同意：不恰当的操作可能会无法访问互联网，引发家庭~~暴力~~矛盾。造成的任何损失由您自己负责。**

### C 缘起
我之前是用[RouterOS](https://mikrotik.com/aboutus)拨号并且提供NAT和DNS/DHCP服务，配合一个[OpenWrt](https://openwrt.org/)上运行的[SmartDNS](https://github.com/pymumu/smartdns) （用来提供DNS服务）和[AdGuardHome](https://github.com/AdguardTeam/AdGuardHome)（下称ADGH，用来去广告/反跟踪）以及$SR+/PA$SWALL提供不可描述服务。

我这种做法需要在RouterOS上维护`分流列表`，这个不支持域名，初期操作还挺繁琐的，前几天还正好被我玩坏了又木有备份设置，干脆关机丢一边专心鼓捣OpenWrt，因为我~~生活在发达的大城市~~这边的互联网建设还算不错，多数时候SmartDNS的加速作用不是很明显，所以我干脆也不要SmartDNS了，用ADGH提供DNS服务。
ADGH基本的原理是它作为一个DNS服务器，维护着亿点点过滤规则，当DNS客户端查询的域名命中时则不予解析（黑名单），否则转发到您指定的上游DNS服务器。

---

### Dm 范围

- [x] 硬件：苹果台式机或笔记本电脑
- [x] 软件：运行Big Sur 11.1

### Em 线索

在终端里输入如下命令检查唤醒事件

```bash
$ pmset -g log | grep DarkWake
```

检查其中最频繁的事件，其中有两种：

第一种类似这样的：

>DarkWake DarkWake from Deep Idle [CDNP] : due to SMC.OutboxNotEmpty smc.70070000 wifibt wlan/ Using AC (Charge:100%) 6 secs

这种是由TCPKeepAlive引起的。

---

第二种类似这样的：

>DarkWake DarkWake from Deep Idle [CDNPB] : due to NUB.SPMISw3IRQ nub-spmi.0x02 rtc/Maintenance Using AC (Charge:92%) 45 secs

这种是由Power Nap引起的。

### F 解决

#### 先升级到Big Sur 11.2

苹果昨天发布了Big Sur 11.2，虽然版本说明里面木有提自动唤醒的事，但是暗中解决了。这个是前提。

#### 关闭相应选项

升级完成后

针对第一种，在终端输入：

```bash
sudo pmset -a tcpkeepalive 0
```
---
针对第二种，可以在系统偏好设置里的节能选项里关闭Power Nap（限Intel电脑）；M1的电脑还是在终端输入：

```bash
sudo pmset -a powernap 0
```

好啦，经过了一番操作之后，您的电脑应该不会再失眠啦。谢谢观看。
