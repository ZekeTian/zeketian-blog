---
layout: post
title: "搭建 PHP 开发环境"
categories: 编程语言
tags: PHP
author: ZekeTian
---

* content
{:toc}


在 Windows 下，大多数人可能直接选择安装 WampServer ，从而将 PHP 开发需要的环境直接搭建好，但是今天这篇文章将介绍 PHP、Apache 的单独安装方法，通过本文你可以学习到一些安装过程遇到问题的解决方法。




## 1. 下载 PHP/Apache

### 1.1 下载 PHP 7

进入 [PHP 官网](https://secure.php.net/downloads.php) 下载 PHP 7，选择 Windows downloads，选择 Thread Safe 版本的 zip 包下载

**注意：**PHP 分为两种版本， 

- Thread Safe，线程安全版本，Apache 服务器选择此版本
-  Non Thread Safe，若是 FastCGI IIS 方式，则采此版本效率更高 

### 1.2 下载 Apache

下载步骤

- 进入 [ Apache 官网](http://httpd.apache.org/) 下载 **HTTP Server**(在 apache 首页搜索 http server 即可找到)
- 选择 Apache HTTP Server 2.4.29 ，点击 **Download** 进入下载选择界面
- 选择 **Files for Microsoft Windows** 
- 选择 **ApacheHaus** 已经编译的版本
- 选择相应的版本点击 **Download** 即可开始下载

## 2. 安装 PHP/Apache

### 2.1 安装 PHP

将 PHP 的 zip 包解压到想要安装的路径即可

### 2.2 安装 Apache

- 将 Apache 的 zip 解压到想要安装的路径，如 <u>D:\Program Files\Apache24</u>
- 进入解压后的目录下的 bin 文件夹中，按住 Shift 同时右键单击鼠标，选择**“在此打开 PowerShell”**（或者“在此打开命令行窗口”），输入 **“*httpd.exe*”** 后回车运行
- 打开浏览器，输入“http://127.0.0.1/” 即可访问 Apache Haus's test page（网页的标题为 “Apache Haus”）



**遇到的问题**

提示 **“ServerRoot must be a valid directory”**，解决方法：

- 第一种：进入安装目录下的 **conf** 文件夹中，打开 httpd.conf，找到**Define SRVROOT "/Apache24"**， 将 "/Apache24" 更改为实际安装路径
- 第二种：直接将 Apache 解压到盘符的根目录下，如：<u>c:\Apache24</u>

提示 **“ make_sock: could not bind to address 0.0.0.0:443”** 解决方法：

​	运行 CMD ，输入 **netstat -ano** 查看端口使用情况，检查是否有 Local Address 为 “0.0.0.0:443”的进程， 如果有则是因为该进程占用了 443 端口，如 VMware 则会占用 443 端口。可采用如下方法解决：

1. 第一种（修改端口号）：
   - 打开 <u>“Apache24\conf\extra”</u> 下的 httpd-ssl.conf 文件，找到 **“Listen 443 ”** ，将其注释或者将 443 更改为其它 xxx 端口。
   - 更改外 httpd-ssl.conf 之后，打开同目录下的 httpd-ahssl.conf  文件，找到 **“Listen 443 https”**，将其注释或者更改为其它 xxx 端口（与前面的一致）。


1. 第二种（关闭 SSL）
   - 打开 <u>“Apache24\conf”</u> 目录下的 httpd.conf
   - 找到 **“LoadModule ssl_module modules/mod_ssl.so”** 以及 **“Include conf/extra/httpd-ahssl.conf”** ，将其注释即可

## 3. 配置 PHP/Apache

### 3.1 配置 PHP

找到 php 解压后的目录下的 **“php.ini-development”** 文件，复制一份后，将文件名更改为 “php.ini”，然后打开，找到如下内容并进行修改

```
1. 设置 PHP 扩展包目录
; On windows:
; extension_dir = "ext"

更改为：
; On windows:
extension_dir = "D:/php7/ext"


2. 开启相应的库
; php 7.1 及以及版本
;extension=php_curl.dll
;extension=php_gd2.dll
;extension=php_mbstring.dll
;extension=php_mysqli.dll
;extension=php_pdo_mysql.dll
;extension=php_xmlrpc.dll

php 7.2 版本 以后没有 php 前缀, 直接是库名, 如下例
;extension=mysqli

找到上面的信息之后, 将 ; 删除，开启库


3. 设置时区
;date.timezone =
 
更改为
date.timezone = Asia/Shanghai
```



### 3.2 配置Apache

- 打开 <u>“Apache24\conf”</u> 目录下的 httpd.conf
-  找到 **“DirectoryIndex index.html”** ，在后面添加 **“index.php index.htm”**，以支持 php
- 添加如下 php 模块信息

```
LoadModule php7_module "D:/php7/php7apache2_4.dll" 
PHPIniDir "D:/php7"   

<IfModule php7_module>
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz
    AddType application/x-httpd-php .php
    AddType application/x-httpd-php .html
    AddType application/pdf .pdf
</IfModule>
```

- 更改 Apache 默认的站点位置

```
在 httpd.conf 目录下找到：
DocumentRoot "${SRVROOT}/htdocs"
<Directory "${SRVROOT}/htdocs">

将 ${SRVROOT}/htdocs 修改为项目目录，如下所示
DocumentRoot "D:/Code/Php/basic"
<Directory "D:/Code/Php/basic">
```

 

## 4. PhpStorm 设置 PHP CLI Interpreter

打开 PhpStorm，按如下步骤进行

1. 点击 **Files** -> 点击 **“Settings”** -> 点击**“Languages & Frameworks”** 下的 **“PHP”**  ->  点击 CLI Interpreter 右侧的  **“...”** 按钮，弹出对话框
2. 点击**“+”**  -> 选择 “Local Path to Interpreter” -> 点击 **PHP executable**  右侧的  **“...”** 按钮设置 **PHP executable** 路径 
3. 在弹出的对话框中找到 PHP 安装目录下的 php.exe 即可

## 5. 测试环境

利用 PhpStorm 编写如下代码

```php
<?php
    echo "hello, php!";
    phpinfo();
?>
```

在浏览器中输入：“http://localhost/test.php” （Apache 的站点路径需要修改为项目路径）， 若在网页中显示 “hello, php!” 以及 PHP 的信息则成功搭建好 PHP 环境，接下来可以愉快地学习 PHP 了！