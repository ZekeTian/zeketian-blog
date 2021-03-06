---
layout: post
title: "计算机组成原理知识点总结（一）"
categories:  计算机课程
tags: 计算机组成原理 计算机系统概念
author: ZekeTian
---

* content
{:toc}

计算机组成原理知识点总结系列之“计算机系统概念”。在本篇博客中，主要介绍计算机系统的一些基本知识，如计算机的发展历程、计算机的软硬件概念。





## 1.1 计算机发展历程

|  代数  |     时间      |   硬件技术   | 速度 / (次/秒)  |
| :--: | :---------: | :------: | :---------: |
|  一   |  1946-1957  |   电子管    |   40 000    |
|  二   |  1958-1964  |   晶体管    |   200 000   |
|  三   | 1965 - 1971 | 中小规模集成电路 |  1 000 000  |
|  四   | 1972 - 1977 | 大规模集成电路  | 10 000 000  |
|  五   |  1978 - 现在  | 超大规模集成电路 | 100 000 000 |

## 1.2 计算机系统层次结构

### 1.2.1 计算机的软硬件概念

​	**计算机系统组成**

- 硬件（计算机的实体，如主机、外设）
- 软件（由具有特殊功能的信息、程序组成）

  软件可以细分为 **系统软件** 和**应用软件**

  ```
  系统软件常见的有：操作系统，编译程序，数据库管理系统
  ```



​	

​	**编程语言的发展**

​	机器语言（面向机器，由二进制代码组成，速度快）

    ↓

​	汇编语言（面向机器，符号化的机器语言，经汇编后运行）

    ↓	

​	高级语言（面向问题，经编译链接后运行，可细分为编译型和解释型）

​	

​	**计算机系统的层次结构**

​	虚拟机器M4	  （用编译程序翻译成汇编语言，对应高级语言）

    ↓

​	虚拟机器M3	  （和汇编语言翻译成机器语言，对应汇编语言）

    ↓

​	虚拟机器M2	  （用机器语言解释操作系统）

    ↓

​	实际机器M1	  （用微指令解释机器语言，对应机器语言）

    ↓

​	微程序机器M0（由硬件直接执行机器语言）

​	

​	**计算机体系结构与计算机组成的区别与理解**

​	**计算机体系结构**是程序员所看到的计算机系统的属性，<u>只关系计算机是否含有某项属性（或功能特性），而不关心实现这个属性的具体细节</u>。

​	`在计算机体系结构中，关心的是“有无乘法指令”`

​	**计算机组成**则强调<u>计算机体系结构中表现的属性的实现</u>

​	`在计算机组成中，关心“如何实现乘法指令”`	

