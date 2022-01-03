---
layout: post
title: Jupyter Notebook 的安装与配置
categories: 机器学习
tags: Jupyter VSCode Anaconda
author: ZekeTian
---




* content
{:toc}
本文主要介绍 Jupyter Notebook 的安装、使用，同时还会介绍 Jupyter 和 Visual Studio Code 的关联、 Anaconda 的基本使用等相关问题。



## 1. 介绍

### 1.1. Jupyter Notebook 介绍

Jupyter Notebok 是一个交互式笔记本，支持运行 40 多种编程语言。因为  Jupyter Notebook 相比 PyCharm 而言，内存占用少，能指定运行中间的部分代码、输出中间的结果、避免反复读取数据等。正是因为其具有这些较好的交互性，适合机器学习的初学者使用。

当然，虽然 Jupyter Notebook 具有以上这些优点，但是也存在一些不足。当代码需要调试时，在 Jupyter Notebook 中可以用 Python 的 PDB 设置断点进行调试，但没有 IDE 那么方便。因此当代码出现 Bug 时，依然建议用 IDE 调试。

### 1.2 Anaconda 

在正式介绍 Anaconda 之前先简单介绍下 Conda。

> Conda 是一个开源的软件包管理系统和环境管理系统，用于安装多个版本的软件包及其依赖关系，并在它们之间轻松切换。Conda 支持语言包括 Python，R，Ruby，Lua，Scala，Java，JavaScript，C / C ++，FORTRAN。**Conda 的作用是进行包管理和环境管理**。

Anaconda 是一个开源的 Python 发行版本，其包含了conda、Python等180多个科学包及其依赖项。可以通过 Anaconda 对不同的 Python 环境进行管理、切换，从而避免一些环境冲突问题。

## 2. Jupyter Notebook 安装及启动



Jupyter Notbook 的安装有两种方式，一种是直接通过 Pip 安装，另外一种是通过 Anaconda 安装。对于机器学习的初学者而言，一般建议通过后者安装，因为 Anaconda 会自动安装常见的 numpy、pandas 等常见的 Python 包。

### 2.1 pip 方式



Windows 环境下通过 pip 安装 Jupyter Notebook 时，分为“只安装一种版本的 Python”和“同时安装了 Python2 和 Python3 ”这两种情况。这两种情况下，安装输入的命令略有不同，分别按照如下相应方式的步骤进行即可：

#### 2.1.1 只含有一种版本的 Python

- 升级 pip 

  `python -m pip install --upgrade pip`

- 安装 jupyter

  `pip install jupyter`

#### 2.1.2 同时含有 Python2、Python3

在这里介绍的是为 Python3 安装 Jupyter，Python2 同理。

- 指定升级 python3 的 pip
  
  `py -3 -m pip install --upgrade pip`    
  
- 安装 Jupyter
  
  `py -3 -m pip install jupyter`

### 2.2 Anaconda 方式

Anaconda 方式非常简单，直接在 [Anaconda 官网](https://www.anaconda.com/)下载安装 Anaconda 即可。Windows 系统在安装 Anaconda 后，在 “开始” 菜单中会新增：

- **Anaconda  Navigator** （用于启动 Anaconda 的图形化管理器）
- **Jupyter Notebook** （启动 Jupyter）
- **Anaconda Prompt**（用于启动 Anaconda 的命令行窗口）

### 2.3 启动 Jupyter Notebook

#### 2.3.1 启动一个

启动一个 Jupyter Notebook 比较简单，有如下两种方式。

- 通用启动方式（命令启动）

  如果 Jupyter Notebook 是通过 pip 安装，打开系统的命令行窗口，输入如下命令（此种方式请留意下面的**注意**内容）： 

  `jupyter notebook` 

  如果 Jupyter Notebook 是通过 Anaconda 安装，在 “开始” 菜单中打开 **“Anaconda Prompt”** ，然后输入如下命令 ：

  `jupyter notebook`。

- Anaconda 特有启动方式

  在 “开始” 菜单中打开 **“Jupyter Notebook”**  即可。

**注意：**

通过系统的命令行窗口启动，提示 “**jupyter 不是内部或外部命令，也不是可运行的程序**” 时，则需要设置 Python 的环境变量。设置方法：将 “Python安装路径\Scripts” 添加到系统环境变量中。

#### 2.3.2 启动多个

如果是利用命令行启动 Jupyter ，并且想开启多个 Jupyter 时，可以通过指定端口号来启动。启动的命令如下：

`jupyter notebook --port <port_number> `

Jupyter 默认端口号是 8888，当在命令行中启动多个 Jupyter 时，需要设置不同的端口号。

**注意：**

通过 Anaconda 特有启动方式启动 Jupyter 时，可以直接在 “开始” 菜单中打开多次 **“Jupyter Notebook”**  即可。这种方式下，后面开启的 Jupyter 会自动切换端口。

## 3. 关联 Visual Studio Code

如果觉得在 Jupyter 的浏览器界面中写代码不舒服，可以将 Jupyter 与 Visual Studio Code 进行关联，从而可以方便地使用 Visual Studio Code 的代码提示功能。

### 3.1 Visual Studio Code 插件安装

首先要确保  VS Code 能够支持 Python 语言，所以如果你的  VS Code  未安装 Python 插件，请先安装如下图所示的插件。

![VSCode 安装 Python插件](https://zeke-images.oss-cn-hongkong.aliyuncs.com/blog/2020-3-7-Anconda-jupyter-notebook-install-img1.png)

### 3.2 选择环境

如果使用的 Python 环境不含有 Jupyter，会无法正常使用。因此在安装完插件后，需要将 Python 环境切换到含有 Jupyter 的环境中，具体方式如下图所示。

![image-20200311181405279](https://zeke-images.oss-cn-hongkong.aliyuncs.com/blog/2020-3-7-Anconda-jupyter-notebook-install-img2.png)

**注意：**

如果打开 VS Code 时，提示 “**microsoft python language server 更新**”，后面可能会引发一些问题，详情请参考 [shaooping 博客](https://blog.csdn.net/shaooping/article/details/89391469)。

## 4. Jupyter Notebook 常用魔法命令

- %lsmagic ，查看 Jupyter Notebook 所有的魔法命令，用法如下：

  `%lsmagic`

- %run：加载脚本，用法如下：
  
  `%run script_path`

- %timeit：计算执行时间函数（会重复执行），只能测试一句代码

  **该函数会根据实际情况重复执行代码，然后计算平均时间**。一般默认会执行一个语句100000次，然后给出运行最快3次的平均值，具体用法如下：

   `%timeit 一句代码`

- %%timeit：计算一段代码的执行时间，用法如下：

  ```
  %%timeit
        测试的代码
  ```

- %time：计算执行时间函数（不会重复执行），只能测试一句代码，用法如下：

  `%time 一句代码`


- %%time：计算一段代码的执行时间，用法如下：

  ```
  %%timeit
        测试的代码 
  ```

**关于选择魔法命令来计算代码执行时间的建议：**

- 如果对计算所得时间的准确性要求较高，则尽量使用 timeit，因为 time 只运行一次具有偶然性。

- 如果多次执行时，每次执行的效率不同（如：前面执行结果会缓存，导致后面执行的运行时间大大缩短，这样每次执行的效率不同），则 timeit 所得的平均计算时间不一定准确。在这种情况下，建议使用 time。

## 5. Anaconda 简单使用

在介绍了 Jupyter Notebook 的常用魔法命令后，接下来将简单地介绍 Anaconda 的一些使用。

### 5.1 环境管理

有时候，A 项目需要 Python2 和一些依赖包， B 项目需要 Python3 和其他一些依赖包。为了满足 A、B 两个项目的环境需要，我们通常可以利用 Anaconda 为它们创建不同的环境。因此，接下来介绍的是 Anaconda 的两种环境管理方式。

#### 5.1.1 图形化界面管理

在开始菜单打开 **“Anaconda  Navigator”** ，进入 Anaconda 的图形化管理界面。在该界面中，我们可以进行包管理、环境管理。由于具有图形界面，操作方式直观容易。因此，在这里只是简单介绍存在这种方式，不对其进行详细说明。

#### 5.1.2 命令行管理

此种方式是打开 Anaconda 的命令行窗口，然后输入相应的命令来管理环境，常用命令如下：

- 创建一个环境
  conda create --name 新环境名 python=python版本（如：3.7）
- 复制一个环境
  conda create -n 新环境名 --clone 待克隆环境名
- 删除一个环境
  conda remove -n 环境名 --all
- 查看所有环境
  conda info -e
  （当前环境前面含有 *）
- 激活环境
  Linux，OS X: source activate snowflakes
  Windows：conda activate snowflake
- 退出当前环境
  conda deactivate

### 5.2 更换源

由于 Anaconda 的镜像源是国外的，所以在国内下载依赖包时，下载速度会比较慢。为了加快下载速度，我们可以将镜像源更换成国内的，更换的步骤如下：

- 打开 Anaconda 的命令行窗口

- 输入如下的命令

  ```
  conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  
  conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge 
  
  conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
  
  # 设置搜索时显示通道地址
  conda config --set show_channel_urls yes
  ```

上面的方式是通过命令行换源，实际上也可通过 Anaconda 的图形化界面更换，具体步骤这里不再进行介绍。

在安装完 Jupyter Notebook 之后，我们就可以用它愉快地写代码，开始机器学习之路了。但是为了让它更加方便，我们可以对其进行一些配置，具体的配置内容将在下一篇博客中介绍，敬请期待！

## 参考文章

[1] [VS Code 插件问题](https://blog.csdn.net/shaooping/article/details/89391469)

[2] [常用魔法命令](https://blog.csdn.net/u011213419/article/details/81299282)

[3] [Anaconda 常用命令](https://www.cnblogs.com/testing-BH/p/10286031.html)

[4] [Anaconda 换源](https://blog.csdn.net/kaige2111/article/details/90727476)