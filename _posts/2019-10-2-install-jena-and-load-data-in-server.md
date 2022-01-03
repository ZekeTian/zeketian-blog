---
layout: post
title:  Linux 服务器上 Jena 的安装及简单使用
category: 图数据库
tags: Jena
author: ZekeTian
---



* content
{:toc}


在本文中首先介绍如何在 Linux 服务器上安装 Jena ，然后简单地介绍如何在 jena 里面创建数据库和导入数据，需要说明的是本文中安装的 Jena 版本为 3.6。



## 1.下载 Jena

在 [Jena 官网](https://jena.apache.org/download/index.cgi) 中下载最新版的 jena 和 jena-fuseki，如果想要下载历史版本可以在 [归档](http://archive.apache.org/dist/jena/) 网站中查找下载。

## 2、安装 Jena

将下载好的 jena、jena-fuseki 安装包上传到服务器上，然后将安装包移到安装位置并解压。

## 3、配置

### 3.1 配置环境变量

输入 `vi /etc/profile` 命令，利用 vim 编辑 profile ，在 profile 文件中的最下面添加如下内容：

```bash
#jena settings
export JENA_HOME=jena 解压目录（到 apache-jena-3.6.0 级即可）
export PATH=$PATH:$JENA_HOME/bin
```

编辑完保存 profile，然后输入 `source /etc/profile` 命令更新环境变量。

### 3.2 验证安装、配置是否正常

为了验证 Jena 和环境变量是否安装、配置好，在任意位置输入 `sparql --version` 命令，如果能正常出现版本信息说明安装成功。

## 4、 Jena fuseki 的运行及安全配置

### 4.1 运行 jena fuseki

进入 jena-fuseki 的解压目录，然后输入 `./fuseki-server ` 命令，当出现“Started 运行时间 on port 3030”字样说明启动 fuseki 成功，然后在浏览器中输入“ip 地址:3030” 访问管理页面。

### 4.2 修改安全配置文件

为了安全性，直接通过服务器的 ip 访问管理页面时点击“add new dataset” 是无反应的。要想在管理页面中可以添加数据库，需要修改 Jena fuseki 的配置文件（**具体是否修改，根据自己系统的安全性考虑**），修改方法如下所述：

进入 “fuseki 解压目录/run” 文件夹，将 **shiro.ini** 文件复制一份，然后输入 `vi shiro.ini` 命令编辑 **shiro.ini** 文件。控制用户的访问有如下两种策略（默认是只能通过 localhost 的方式本地访问）：

* 通过用户名、密码进入（**推荐**）

  ```bash
  [users]
  admin=pw 
  
  [urls]
  ## Control functions open to anyone
  /$/status = anon
  /$/ping   = anon
  
  ## and the rest are restricted to localhost.
  #/$/** = localhostFilter
  /$/** = authcBasic,user[admin]
  # Everything else
  /**=anon
  ```

  在上面修改的内容中，需要将 **“admin=pw”** 中的 **“pw”** 更改为密码（admin 是用户名，根据实际需要修改用户名） 。将默认的 `/$/\** = localhostFilter` 注释掉，然后添加一行  `/$/\** = authcBasic,user[admin]` ，即允许 **“admin”** 用户通过密码登录访问。

* 允许任何人进入（**不推荐**）
  修改 **shiro.ini** 文件中如下内容：

  ```bash
    ## and the rest are restricted to localhost.
    #/$/** = localhostFilter
    /$/** = anon
  ```




在上面修改的内容中，先将默认的 `/$/\** = localhostFilter`  注释掉，然后添加一行  `/$/\** = anon` ，即允许所有人访问管理页面。

关于 fuseki 安全的一些详细配置，可参考 [ jena fuseki 官方文档](http://jena.apache.org/documentation/fuseki2/fuseki-security.html) 。

## 5. 创建数据库

- 点击管理页面上方导航栏中的 “manage dataset”，可以添加数据库和删除已存在的数据库
- 点击 **“add new dataset”** ，然后输入 dataset name（即数据库名），dataset type 有 **“In-memory”** 和 **“Persistent”** 两种类型，一般情况下建议使用  **“Persistent”** 类型

在创建数据库时，如果选择了 **Persistent**  类型，那么会自动创建该数据库对应的 ttl 配置文件，在后面配置全文搜索时会需要修改该文件（该文件一般默认存放在  “ jena-fuseki安装目录/run/configuration”  目录下）。

**注意：** 如果点击 “add new dataset” 时无反应，需要修改 fuseki 的安全配置文件，详情请看 4.2 小节内容。

## 6、导入数据

### 6.1 准备工作（检查数据是否正常有效）

有些数据集在收集整理过程中，其部分数据可能不规范，如 uri 中含有如 **“`{}”** 等非法字符，为了避免中途导入数据出错，建议使用 jean 自带的 **riot** 命令检查数据。在终端中输入如下命令：

```bash
riot --validate 数据存储位置
```

如果数据集中存在无效非法的数据， **riot** 命令会中断并提示相关信息。

**注意：**只有前面的 jena 安装、环境变量配置正常之后，上面的命令才能正常使用。

### 6.2 通过可视化界面导入数据

在 jena fuseki 提供的管理页面中，我们可以点击管理页面上方导航栏中的 “manage dataset”，在需要导入数据的数据库那一行中点击 **“upload data”** ，即可选择本地的数据文件（这些文件最终会导入到服务器数据库中）。支持导入的数据文件包括常见的 RDF 数据，如 **.ttl**、**.nt** 、**.rdf** 等，如果想要查看支持的所有文件类型，请查看 [jena 文档](http://jena.apache.org/documentation/io/)。

### 6.3 通过命令行导入数据

当数据是如下两种情况时，可通过命令行的方式将数据导入到服务器中。

- 数据是存放在服务器上
- 本地的数据比较大（建议先上传到服务器，然后通过命令行的方式导入，速度相对会更快）

使用 tdb 导入数据的命令如下所示：

```bash
tdbloader --loc=数据库存储路径 待导入的数据路径
```

**注意事项：**

- 只有前面的 jena 安装、环境变量配置正常之后，上面的命令才能正常使用。
- 路径不要有空格，如果有空格，使用双引号`""` 包住路径
- 数据库存储路径可以查看相应的 ttl 配置文件中的 **“tdb:location”** 值（路径一般是 “jena-fusekie 安装目录/run/databases/数据库名”）



## 参考文章

[1] http://jena.apache.org/documentation/tools/index.html

[2] https://www.jianshu.com/p/380a768e4aea

[3] http://jena.apache.org/documentation/fuseki2/fuseki-security.html

[4] http://jena.apache.org/documentation/io/



