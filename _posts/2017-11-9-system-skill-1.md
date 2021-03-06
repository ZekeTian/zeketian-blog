---
layout: post
title: "Windows 使用技巧系列之利用系统自带工具"
categories: 杂记
tags: Windows 技巧
author: ZekeTian

---


* content
{:toc}


大多数人的电脑使用的是 Windows 系统, 但是我们是否想过如何让 Windows 系统使用起来更加方便、高效呢？为此，我将特意整理一些 Windows 使用的技巧，提高我们的效率，本篇博客讲述的是**利用 Windows 自身的功能来提高效率**。



## 1. 快速启动程序

### 1.1 简介

在 Windows 中想要快速启动一个程序，常见的方式是双击应用程序的快捷方式，但是如果安装的程序过多，从桌面上众多的图标中寻找想要打开的程序是一件苦恼的事情。那么如何避免寻找快捷方式，快速打开一个程序呢？

实际上想要解决这个问题，我们可以直接利用 Windows 系统自带的功能来实现，而且实现的方式有两种，分别为：

1. 利用快捷键（建议**高频软件**使用）
2. 利用系统的“运行”程序来打开指定程序（建议**中频软件**使用）


接下来，就让我们来看看设置的步骤吧。

### 1.2 设置快捷键

**步骤：**

找到需要设置的软件的快捷方式  ==》右键单击快捷方式图标 ==》选择**“属性”** ==》在弹出的属性对话框中选择 **“快捷方式”**（默认选中） ==》 在**“快捷键”**一栏中输入你想自定义的快捷键（比如同时按 Ctrl, Shift, Q 即可将 “Ctrl + Shift + Q” 设置成快捷键）

**示例**

![效果图](http://ov8clk5cn.bkt.clouddn.com/17-11-19/87438459.jpg)

**注意：**在设置的时候，要避免快捷键冲突。可以在设置之前先使用想自定义的快捷键，看该快捷键是否已经被使用，若未被其它程序占用，即可设置。

经过上面的设置步骤之后，如果快捷键没有冲突，你现在就可以愉快的使用快捷键来启动程序了。



### 1.3 利用“运行”来打开指定程序

有软件开发经验的朋友对于这种方式，相信一定不会陌生，现在就让我们来看看具体的步骤吧!

**步骤：**

在 C 盘或 其它盘新建一个文件夹（如 winrun） ==》将需要设置的软件的快捷方式复制到创建的文件夹内 ==》 将快捷方式重命名为一个简短的名字（尽量使用你能记住的最简单的拼音或英文单词） ==》添加环境变量

添加环境变量的步骤此处演示的是 **Win10 系统环境下**：

右键单击**“此电脑”** ==》  在弹出的对话框中选择左边的 **“高级系统设置”** ==》单击**“环境变量”** ==》在弹出的对话框中找到**“系统变量”**下的**“Path”**，选中 ==》点击**“编辑”** ==》将之前创建的文件夹的路径添加进去（如“D:\winrun”） ==》 点击确定即可

示例：

![设置环境变量](http://ov8clk5cn.bkt.clouddn.com/17-11-19/75124097.jpg)

**注意：**也可在**“用户变量”**中添加  **“Path”** 之后，将文件路径添加进去。

同时按 **“Win + R”** 启动 **“运行”**程序，输入之前重命名的快捷方式的名字（如将“腾讯QQ”重命名为“QQ”，则只要输入“QQ”即可），便可直接打开程序。

**附常用的系统自带软件启动**：

|   程序   |      命令      |
| :----: | :----------: |
| dos 窗口 |     cmd      |
|   画图   |   mspaint    |
|  计算器   |     calc     |
|  截图工具  | snippingtool |

 **“Win + R”** 启动 **“运行”**程序之后，输入上面的命令即可打开系统自带的软件。

## 2. 快速打开文件

如果我们想快速访问一些文件夹，我们可以选择将文件夹或者文件夹快捷方式放在桌面，但实际上还有一种方法可以快速打开文件夹。

先让我们看下效果图吧！

![效果图](http://ov8clk5cn.bkt.clouddn.com/17-11-19/97824388.jpg)

如果我们需要经常打开某一文件夹里面的内容，采用这种方式就非常适合。具体的步骤如下：

在任务栏右键单击 ==》 选择**“工具栏”**  ==》 选择**“新建工具栏”** ==》 在弹出的对话框中选择需要设置的文件夹

**注意：**在任务栏中显示的是文件夹的名称，可以将文件夹的名称以缩写命名。

## 3. 快速定位文件

如果你是一位以英文或拼音来命名文件夹的用户，那么你可以用这种方法快速定位到某一文件，避免在众多文件夹中寻找。

![效果图2](http://ov8clk5cn.bkt.clouddn.com/17-11-19/40442264.jpg)

如上图所示，在众多的英文文件夹中，你只需要输入 “m” 即可快速定位到以 “m” 开头的文件夹（当然如果以 m 开头的文件夹较多的话，可能还是需要花费一些时间，但是如果在不能查找的情况下，使用此种方法可以快速定位范围，减少时间）。

**注意：**要在**英文状态下**或者**大写状态下**输入文件夹名的**首字母**

## 4. 快捷键的使用

节约时间、提高效率的一个重要途径就是使用快捷键，使用快捷键可以将多步操作简化为一步，起到化繁为简的效果。在这里我将介绍一些常用的快捷键给大家，提高大家的效率。当然对于 Ctrl + C ， Ctrl + V 这种最简单的基本快捷键，在这里我就不再赘述。

|        快捷键        |    功能     |         描述          |
| :---------------: | :-------: | :-----------------: |
|        F2         |    重命名    |                     |
|    PrintScreen    |    截图     |  系统自带的截图功能，截取整个屏幕   |
| Alt + PrintScreen |    截图     | 系统自带的截图功能，可截取当前活动窗口 |
|      Delete       |    删除     |        临时删除         |
|   Shift + Delet   |   永久删除    |     慎用，避免导致误删文件     |
|     Ctrl + F      |    查找     |                     |
|     Ctrl + S      |    保存     |   在编写文档时，此快捷键必不可少   |
|     Ctrl + Z      |    撤销     |      可以撤销上一步操作      |
|     Ctrl + Y      |  恢复（反撤销）  |   若撤销过多，可使用此快捷键恢复   |
|     Ctrl + A      |    全选     |                     |
|      Win + D      |   返回到桌面   |                     |
|      Win + E      | 打开文件资源管理器 |                     |
|     Win  + M      |  最小化程序窗口  |                     |
|      Win + ↑      |  最大化程序窗口  |                     |
|     Alt + F4      |   关闭窗口    |                     |
|     Alt + Tab     |  正向任务切换   |  调出任务切换视图，进行正向任务切换  |
| Alt + Shift + Tab |  反向任务切换   |  调出任务切换视图，进行反向任务切换  |
| Ctrl + Shift + N  |   新建文件夹   |                     |
|                   |           |                     |

可能有的人觉得记快捷键是一件麻烦事，觉得浪费时间，但是只要你经常使用这些快捷键，就会慢慢地记住，这就让你变得更加高效。总之一句话，快捷键只有多用才能熟记。