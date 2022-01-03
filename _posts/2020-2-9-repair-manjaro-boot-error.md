---
layout: post
title: 修复 Manjaro 系统启动
categories: Linux
tags: Linux Manjaro
author: ZekeTian
---



* content
{:toc}
本文主要介绍如何修复 Manjaro 强制关机后无法正常启动的问题。



## 1. 问题描述

在 Manjaro 上运行一个程序时，因 CPU、内存使用率非常高，系统处于卡死状态。无奈之下，长按电源键强制关机。当再次开机时，系统无法正常启动，进入了 **emergency shell** 状态，提示如下信息：

```
mount: /new_root: can't read superblock on /dev/sdb2
You are now being dropped into an emergency shell.
sh: can't access tty; job control turned off
[rootfs ]# 
```

在该状态下，只能使用 **ls、cd** 等非常基本的命令，无法使用 **exit**  命令，只能强制关机。

## 2. 解决办法

上述问题的出现是由于强制关机造成的，因为在 **emergency  shell** 中不能使用 **fsck、e2fsck** 命令对文件系统进行检查。因此，需要刻录一个 Linux 启动盘，从外部 Linux 系统进入，然后再运行 **fsck、e2fsck** 命令。

### 2.1 刻录 Linux 启动盘

制作 U 盘启动盘的主流工具有 **rufus**、**UltraISO**，本文中使用的刻录工具是 **rufus**。因为笔记本安装的是 **"Manjaro 18.1"** 系统，所以刻录的 Linux 启动盘也选择对应的系统。

**注意：**在使用 UltraISO 刻录的时候，需要注意启动方式，默认是：USB-HDD+。如果按照默认的配置直接刻录，会无法正常启动，网上其他博客的一些说法是更改为 ***RAW*** 格式。本人未进行尝试，仅供大家参考。

### 2.2 启动 Linux 启动盘

刻录完 U 盘启动盘后，进入 BIOS 更改启动顺序，从 U 盘的 Linux 系统进入。

进入 U 盘的 Linux 系统后，可以选择安装系统，也可以利用终端工具输入命令解决问题，该系统类似于 Windows 的 PE 系统。

### 2.3 检查文件系统

U 盘的 Linux 系统启动成功后，打开终端工具，输入如下命令，对文件系统进行检查。

```
sudo e2fsck -f -y /dev/sdb2
```

#### 2.3.1 **命令参数解释**

​	**-f** 强制检查，无论文件系统是否出错

​	**-y** 采取非互动方式执行，所有的问题均设置以"yes"回答

#### 2.3.2 **注意事项**

命令中最后面的 **“/dev/sdb2”** 表示文件系统，因为根据问题的描述信息可知，是  **“/dev/sdb2”** 出现问题。在解决问题时，应该根据实际情况进行修改。

如果不知道则使用 **lsblk** 命令列为出所有文件系统，然后逐一尝试。但是要注意，不能对已经挂载的文件系统执行 **e2fsck**。如果要执行，需要先使用 **“sudo umount 文件系统”** 命令卸除文件系统。


## 参考文章

[1]: https://www.jianshu.com/p/eb248826b95c