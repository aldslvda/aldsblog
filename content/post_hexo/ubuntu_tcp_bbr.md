title: VPS升级内核，开启TCP BBR 实现高效单边加速
date: 2017-04-14 09:47:02+08:00
tags:
- ubuntu
- vps
- tcp-bbr
categories:
- 技术分享
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/google_tcp_bbr.png?raw=true"
toc: true
comment: true
---

## 1. 从锐速到BBR
自从锐速发布以来，这款牛逼的单边加速神器的确为一些线路不太优秀的服务器带来了更优秀的体验。  
但是由于过高的价格和不再低端售卖。导致了我们除了使用这个软件的破解版之外，并没办法得到一个免费好用的单边加速功能。  
而由于破解**并不完美**，使得在使用中会出现一些无法解决的问题，比如随着运行时间变久，加速效果越来越差（体验中的问题就是youtube观看视频从流畅1080p到不流畅480p。。。。）  
但是去年下半年谷歌为我们带来了新的TCP拥塞控制算法*BBR（Bottleneck Bandwidth and RTT)*。 目前在 Linux Kernel 4.9 及之后的版本中加入了该算法，所以我们只要升级内核就可以使用BBR开启单边加速了。

## 2. 升级Ubuntu内核
上面说到 BBR只被 Linux Kernel 4.9 以后的版本支持，而VPS多用一些比较稳定的旧版内核。所以我们要做的第一步就是升级内核:  

- 首先查看自己的系统内核：

```bash
$uname -a
``` 

如果 系统版本是4.9及以后, 升级内核这一步就可以跳过拉~  

- 到ubuntu官方[内核源](http://kernel.ubuntu.com/~kernel-ppa/mainline/)找到最新的内核：   
  比如：
  
  > http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.11-rc6/linux-image-4.11.0-041100rc6-generic_4.11.0-041100rc6.201704091331_amd64.deb
  
- 找到之后下载这个deb包：

```bash
$ wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.11-rc6/linux-image-4.11.0-041100rc6-generic_4.11.0-041100rc6.201704091331_amd64.deb
```
- 安装内核：

```bash
$ dpkg -i linux-image-4.11.0-041100rc6-generic_4.11.0-041100rc6.201704091331_amd64.deb
```
- 查看已安装的内核：

```bash
$ dpkg-l|grep linux-image
```
- （可选）卸载旧版内核：

```bash
$ apt-get purge ${old_linux_image}
```
- 更新 grub 文件并重启

```bash
$ update-grub
$ reboot
```
- 重启之后就是新内核拉~

## 3. 开启 BBR 加速

- 修改系统配置文件 sysctl.config

```bash
$ echo"net.core.default_qdisc=fq">>/etc/sysctl.conf
$ echo"net.ipv4.tcp_congestion_control=bbr">>/etc/sysctl.conf
```

- 保存并应用修改

```bash
$ sysctl-p
```

- 查看是否生效：

```bash
$ sysctl net.ipv4.tcp_available_congestion_control
```
如果输出如下：  

> net.ipv4.tcp*_*available*_*congestion_control=bbr cubic ren
表明开启成功

## 4. 关闭 TCP BBR
执行下面的命令修改系统配置：

```bash
sed-i'/net\.core\.default_qdisc=fq/d'/etc/sysctl.conf
sed-i'/net\.ipv4\.tcp_congestion_control=bbr/d'/etc/sysctl.conf
sysctl-p
reboot
```
执行完上面的代码，即 **修改配置-重启VPS** 之后就可以关闭BBR加速了。


