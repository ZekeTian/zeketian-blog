---
layout: post
title: 分析 MPI 段错误
categories: 分布式
tags: C++ MPI
author: ZekeTian
---


* content
{:toc}

在本文中，主要介绍如何分析 MPI 程序的段错误问题（Segmentation Fault），从而快速定位 Bug  所在的代码区域。



## 1. 段错误及相关知识介绍

- 段错误产生的原因

  在 C/C++ 程序中对一段非法内存进行访问，如：**数组下标越界、访问野指针**。

- 核心转储（core dump）

  是操作系统在进程收到某些信号而终止运行时，将此时进程地址空间的内容以及有关进程状态的其他信息写出的一个磁盘文件，可用于调试。

## 2. 方法介绍

本方法主要借助 **core dump** 文件，分析程序出错的地方，具体步骤如下：

- **查看是否开启 core dumping功能：**执行命令 `ulimit -a`，出现如下图所示信息。

  ![ulimit -a](https://zeke-images.oss-cn-hongkong.aliyuncs.com/blog/2021-04-18-segmentation-fault-img1.png)

  图中所示的 **core file size** ，值为 0，则 core dumping 功能未开启，需要先开启该功能。

- **开启 core dumping：**执行命令 `ulimit -c unlimited`

- **编译程序：**为了能够调试**，在编译时，需要添加参数 `-g`**

- **运行程序，得到 core dump 文件**

- **查看 core dump 文件**

  在执行完程序后 ，若产生段错误，一般会生成 `core.xxxx` 文件。之后，执行命令 `gdb 编译后的二进制文件名 core文件名   `，如 `gdb run core.10225` ，进入 gdb 调试环境。

  执行命令 `where`，查看更详细的错误信息，如下图所示。

  ![gdb where](https://zeke-images.oss-cn-hongkong.aliyuncs.com/blog/2021-04-18-segmentation-fault-img2.png)

- **退出 gdb 调试环境：**执行命令 `q`

  





