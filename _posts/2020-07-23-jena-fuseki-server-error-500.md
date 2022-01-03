---
layout: post
title: "Jena fuseki server 500 错误"
categories: 图数据库
tags: Jena
author: ZekeTian
---



* content
{:toc}
本文主要介绍如何解决 Jena Fuseki Server 的 Web 界面进行 SPARQL 查询时遇到的 500 错误，使用的 Jena Fuseki 版本为 3.6.0。



## 1. 问题描述

启动 Jena Fuseki Server 后，在浏览器中访问 Jena 的 Web 管理界面，然后在查询界面中输入 SPARQL 语句。执行 SPARQL 语句后未返回正确结果，提示 ”Error 500: GC overhedd limit exceeded“ ，如下图所示。同时，在 Jena 的后台运行程序中出现 ”OutOfMemoryError“ 异常。

![错误提示信息](https://zeke-images.oss-cn-hongkong.aliyuncs.com/blog/2020-07-23-jena-fuseki-server-error-500.png)



## 2. 解决办法

上述问题出现的原因是查询的数据量较大，中间查询结果较多，而 Jena Fuseki Server 程序默认分配的内存空间较小，最终内存溢出。解决该问题的方法，就是增大分配给  Jena Fuseki Server 程序的内存空间，具体步骤如下。

### 2.1 Linux 环境

- 进入  Jena Fuseki Server 的安装目录，备份 **”fuseki-server“** 文件
- 打开 **”fuseki-server“** 原始文件，搜索 **“JVM_ARGS”**，找到`JVM_ARGS=${JVM_ARGS:--Xmx1200M}`，增大 **“--Xmx”**  后面的值
- 保存文件后退出，重新启动 Jena Fuseki Server 即可

### 2.2 Windows 环境

- 进入   Jena Fuseki Server 的安装目录，备份 **”fuseki-server.bat“** 文件
- 打开 **”fuseki-server.bat“** 原始文件，搜索 **“-Xmx”**，找到`java -Xmx1200M -jar fuseki-server.jar`，增大 **“--Xmx”**  后面的值
- 保存文件后退出，重新启动 Jena Fuseki Server 即可

**注意：**系统环境不同，修改的脚本文件不一样，请根据实际的系统环境进行修改。