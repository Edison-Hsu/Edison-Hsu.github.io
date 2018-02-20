---
layout: post
title: Mac Book Pro Mid 2012 款安装 Windows 10 中学到的东西
date: 2018-02-20T22:50:00+08:00
categories: 总结
published: true
---

## 起因

最近纯洁在家里，偶然想起魔力宝贝这个初中玩的游戏，并且拍车牌的自动软件只能在Windows上跑，就心想着装个Windows吧。所以拿出了闲置很久的买的第一款mbp，mid-2012-mbp。

本想着装个系统有啥麻烦的，结果装了3天还没搞完，我也是醉了。

这个MBP是自己升级过，光驱位置改成了SSD，内存升级到了8G。然而原本的HDD还是原来的。

## Install With Boot Camp

首先想到的方式就是使用BCA，我就列举下在BCA上碰到的问题：

1. 下载的win10iso文件太大，5.7G吧，BCA制作的u盘镜像格式是FAT32的，所以单个文件不能超过4G，导致制作失败。

## Install Without Boot Camp

1. 制作 ISO 的过程比较麻烦，尝试在Mac下制作失败，尝试在虚拟机中用utral ISO安装失败（反正就是崩溃了），最后用rufus制作成功
2. 制作好之后使用MBR分区格式，并且使用bios方式启动，安装完成之后重启，电脑没办法自动进入Windows。
3. 重新制作u盘镜像，这次选择EFI启动方式，结果安装的时候发现只能安装在HDD上，不能安装在SDD上（我的目标就是把系统安装到SDD上啊！！！）
4. 接着捣鼓，然后先在HDD上安装了win10，然后再重新用u盘启动，再去sdd上安装了一次，发现这次就可以了
5. 安装万系统安装，安装驱动，发现声卡驱动死活安装不上，硬件都没有识别，然后又是一番好找，最后发现是因为使用EFI启动的方式导致的（我的天啊！！！）

## Install With Boot Camp Again

1. 没办法再用boot camp，想着这是最正规的安装方法嘛，肯定没问题，然后重新下载小点的ISO，然后用boot camp制作upan镜像
2. 发现 GPT 格式和bios方式启动的windows安装进程不兼容，还要把分区安装windows的磁盘分区改成MBR
3. 

## 总结

1. 制作windows镜像，可以使用BCA，也可以使用rufus，注意u盘的分区，如果是2012的mac就需要支持bios方式，其他的可以使用EFI启动
2. mid-2012在EFI方式下，audio驱动会有问题，所以只能使用bios方式启动安装
3. bios方式
3. bios
3. bios

