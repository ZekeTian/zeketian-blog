---
layout: post
title: Manjaro 上设置 Chrome 代理
categories: Linux
tags: Manjaro Chrome Linux
author: ZekeTian
---



* content 
{:toc}
网上有许多文章介绍利用 SwitchyOmega 插件设置 Chrome 代理，而本文主要介绍另外一种简单快速的方法来设置 Linux 环境的 Chrome 代理。本文使用的 Linux 发行版是 Manjaro，其他 Linux 发行版系统可参考此方法。



## 方法步骤

1. 在 **“/usr/share/applications/”** 目录下找到 **“google-chrome.desktop”**

2. 右键单击 **“google-chrome.desktop”**  -->  选择**“属性”** --> 选择窗口中的**“应用程序”**标签项

3. 在 **“命令”**输入框内的最后面输入如下内容，从而指定 Chrome 的代理服务器 

   `--proxy-server="ip proxy server:port", 如：--proxy-server="127.0.0.1:8080"`

   **注意：**输入的内容与 **“%U”** 之间保留一个空格

   ![设置 Chrome 的代理服务器信息](https://zeke-images.oss-cn-hongkong.aliyuncs.com/blog/2019-12-20-chrome-proxy-on-manjaro-img1.png)

4. 修改后，点击 **“确定”** 保存

5. 启动 chrome，测试代理是否设置成功（如果在进行上述步骤前未关闭 chrome，则先关闭所有已打开的 chrome，然后再进行测试）

## 注意事项

1. 如果执行完上面的操作后，chrome 依然不能设置代理成功，右键单击 chrome 的启动图标，查看是否成功修改了 “命令” 的值。
   如果没有修改成功，则将 **“/usr/share/applications/”** 目录下的 **“google-chrome.desktop”** 复制一份到桌面上，然后对桌面的这个 chrome 启动图标进行上述操作（即修改 “命令” 值）。

2. Manjaro 系统中 1、2 两步也可以按照如下步骤进行：
   在 "应用程序面板" 中找到 chrome --> 右键单击 chrome 图标 --> 选择 “编辑应用程序” --> 选择窗口中的 "应用程序 “标签项



## 参考文章

[1] https://askubuntu.com/questions/513840/google-chrome-proxy-settings