---
layout: post
title: "百度地图的导入"
categories:  Android
tags: 百度地图
author: ZekeTian
---

* content
{:toc}


在本篇博客中将讲解百度地图 SDK 的导入及使用的一些细节问题。








# 1. 申请百度地图 Key

## 1.1 创建应用

前往[百度地图应用控制台](http://lbsyun.baidu.com/apiconsole/key) 创建应用从而申请到百度地图的 key ，因为是开发 Android 应用，所以这里选择的应用类型为 Android SDK。 

![百度地图创建应用](http://ov8clk5cn.bkt.clouddn.com/17-10-4/60745811.jpg)

其中发布版的 SHA1 值是自己用于发布上线的应用签名，开发版 SHA1 值是 debug.keystore ，debug.keysotre 在 Windows 系统下默认是在 `C:\Users\Username\.android`路径下。

## 1.2 获得签名的 SHA1 值

- 获取 debug.keystore 签名的 SHA1 值

  `Win + R` 输入 CMD 打开命令行窗口，然后 cd 进入 `C:\Users\Username\.android` （注意 **Username 应该为你的用户名**）路径。

  输入`keytool -list -v -keystore debug.keystore ` 命令，然后输入密匙库口令`android`，之后便可在证书指纹中找到 SHA1 值

- 获取正式签名的 SHA1 值

  如果之前已经生成过签名，则直接按上面的步骤即可获得 SHA1 值，如若没有签名，请先利用 Android Studio 生成签名，生成方法：

  ![生成签名](http://ov8clk5cn.bkt.clouddn.com/17-10-4/22512029.jpg)

  **切记：注意保存好创建的签名，同时设定的密码一定要记住， 否则此签名后期不能用于应用的签名**


## 1.3 获取包名

打开项目的 AndroidManifest.xml 文件，找到 package 的属性，如下图所示：

![package](http://ov8clk5cn.bkt.clouddn.com/17-10-4/166389.jpg)







# 2. SDK 的下载 、导入及配置

## 2.1 下载

前往百度地图 [SDK下载](http://lbsyun.baidu.com/sdk/download?selected=mapsdk_basicmap,mapsdk_searchfunction,mapsdk_lbscloudsearch,mapsdk_calculationtool,mapsdk_radar)	网站下载，可以根据自己项目的需要自定义下载一些功能组件。下载完成并解压，可以发现百度地图共提供了 arm64-v8a, armeabi, armeabi-v7a, x86, x86_64 五种 ABI 的 so 文件，根据需要选择特定版本的文件。

```
armeabi: 是针对旧的 arm cpu
armeabi-v7a: 是针对具有浮点运算或高级扩展功能的 cpu
arm64-v8a: 是针对 64 位的 arm cpu
```

**注意：**arm64-v8a ， armeabi-v7a 均可以向下兼容， 目前这两种 cpu 为主流， 当然为了尽量兼容古董机子，也可以加上 armeabi 。

## 2.2 导入与配置

**so 文件的配置：**

选择好 so 文件之后，将其拷贝到项目路径下的`app/libs`目录下。

在 Android Studio 中打开项目，打开 Module 的 build.gradle 文件， 在 android {} 中添加如下代码从而配置 so 文件。

```
 // 配置 so 文件
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
```



**AndroidManifest.xml 文件配置：**

添加地图需要的权限:

```java
   <!-- 这个权限用于进行网络定位-->
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"></uses-permission>
    <!-- 这个权限用于访问GPS定位-->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"></uses-permission>
    <!-- 用于访问wifi网络信息，wifi信息会用于进行网络定位-->
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"></uses-permission>
    <!-- 获取运营商信息，用于支持提供运营商信息相关的接口-->
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"></uses-permission>
    <!-- 这个权限用于获取wifi的获取权限，wifi信息会用来进行网络定位-->
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"></uses-permission>
    <!-- 用于读取手机当前的状态-->
    <uses-permission android:name="android.permission.READ_PHONE_STATE"></uses-permission>
    <!-- 写入扩展存储，向扩展卡写入数据，用于写入离线定位数据-->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"></uses-permission>
    <!-- 访问网络，网络定位需要上网-->
    <uses-permission android:name="android.permission.INTERNET"/>
    <!-- SD卡读取权限，用户写入离线定位数据-->
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"></uses-permission>
```

设置百度地图的 Key 值：

```java
	  <!-- 配置百度地图的 SDK 的 key 值 -->
        <meta-data
            android:name="com.baidu.lbsapi.API_KEY"
            android:value="AK"/>	
```

定位需要的服务：

```java
 <!-- 百度地图定位需要的服务, 如果不配置, 定位功能将不能正常使用 -->
        <service
            android:name="com.baidu.location.f"
            android:enabled="true"
            android:process=":remote">
        </service>
```

百度地图 SDK 初始化：

```java
SDKInitializer.initialize(getApplicationContext());
```
