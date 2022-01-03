---
layout: post
title: 非 root 用户离线安装 gcc
categories: 编程语言
tags: C++ gcc
author: ZekeTian
---

* content
{:toc}
在本文中，介绍了**非 root 用户**如何在**离线状态**下通过编译源码的方式安装 **gcc 4.9**。介绍的安装方式有两种，第一种是网上大多数博客中使用的，第二种是本文最终真正安装成功的方式。



## 1. 下载 gcc 源码

在 [gcc 官方](https://gcc.gnu.org/mirrors.html)提供的镜像网站中选择一个，然后进入镜像网站中下载 gcc 源码，本文下载的是 [gcc 4.9](https://bigsearcher.com/mirrors/gcc/releases/gcc-4.9.4/) 。



## 2. 查看 gcc 所需依赖包并下载

解压 gcc 源码，然后进入**解压目录**，执行命令 `cat ./contrib/download_prerequisites`，查看当前 **gcc** 版本所需要的依赖包。

``` shell
MPFR=mpfr-2.4.2
GMP=gmp-4.3.2
MPC=mpc-0.8.1

wget ftp://gcc.gnu.org/pub/gcc/infrastructure/$MPFR.tar.bz2 || exit 1
tar xjf $MPFR.tar.bz2 || exit 1
ln -sf $MPFR mpfr || exit 1

wget ftp://gcc.gnu.org/pub/gcc/infrastructure/$GMP.tar.bz2 || exit 1
tar xjf $GMP.tar.bz2  || exit 1
ln -sf $GMP gmp || exit 1

wget ftp://gcc.gnu.org/pub/gcc/infrastructure/$MPC.tar.gz || exit 1
tar xzf $MPC.tar.gz || exit 1
ln -sf $MPC mpc || exit 1

# Necessary to build GCC with the Graphite loop optimizations.
if [ "$GRAPHITE_LOOP_OPT" = "yes" ] ; then
  ISL=isl-0.12.2
  CLOOG=cloog-0.18.1

  wget ftp://gcc.gnu.org/pub/gcc/infrastructure/isl-0.12.2.tar.bz2 || exit 1
  tar xjf isl-0.12.2.tar.bz2  || exit 1
  ln -sf isl-0.12.2 isl || exit 1

  wget ftp://gcc.gnu.org/pub/gcc/infrastructure/cloog-0.18.1.tar.gz || exit 1
  tar xzf cloog-0.18.1.tar.gz || exit 1
  ln -sf cloog-0.18.1 cloog || exit 1
fi
```

在能够连接外网的 Linux 系统中执行上述脚本的 5 个 `wget` 命令，从而下载得到需要的依赖包：**gmp**、**mpfr**、**mpc**、**isl**、**cloog**，然后上传到需要安装 gcc 的 Linux 系统中。

**注意事项：**在下载 **gmp**、**mpfr**、**mpc** 时，需要将脚本中 ftp 链接的变量名更换成实际文件名。



## 3. 方式一（逐个编译安装依赖包）

网上大多数博客介绍的方法是，逐个编译安装依赖包，然后再安装  gcc。本人曾尝试过该方法，但是未能成功安装，在此仅将该过程记录下来。

### 3.1 安装 gmp

- 解压 gmp：`tar xjf gmp-4.3.2.tar.bz2`

- 创建临时文件夹，用于编译：`mkdir tmp`

- 配置编译参数：

  ```shell
  cd tmp
  ../configure --prefix=~/soft/gcc49/gmp
  ```

- 编译并安装：

  ```shell
  make
  make install
  ```

### 3.2 安装 mpfr

- 解压 mpfr：`tar xjf mpfr-2.4.2.tar.bz2`

- 创建临时文件夹，用于编译：`mkdir tmp`

- 配置编译参数：

  ```shell
  cd tmp
  ../configure --prefix=~/soft/gcc49/mpfr --with-gmp=~/soft/gcc49/gmp
  ```

- 编译并安装：

  ```shell
  make
  make install
  ```

### 3.3 安装 mpc

- 解压 mpc：`tar xzf mpc-0.8.1.tar.gz`

- 创建临时文件夹，用于编译：`mkdir tmp`

- 配置编译参数：

  ```shell
  cd tmp
  ../configure --prefix=~/soft/gcc49/mpc --with-gmp=~/soft/gcc49/gmp --with-mpfr=~/soft/gcc49/mpfr
  ```

- 编译并安装：

  ```shell
  make
  make install
  ```

### 3.4 安装 gcc

安装完所需的依赖包后，执行如下命令,安装 gcc：

- 在 gcc 的解压目录下，创建临时文件夹，用于编译：`mkdir build`

- 配置编译参数：

  ```shell
  cd build
  
  ../configure \
  --prefix=~/soft/gcc49/gcc \
  --enable-threads=posix \
  --disable-checking \
  --enable--long-long \
  --disable-multilib \
  --enable-languages=c,c++ \
  --with-gmp=~/soft/gcc49/gmp/lib \
  --with-mpfr=~/soft/gcc49/mpfr/lib \
  --with-mpc=~/soft/gcc49/mpc/lib
  ```

- 编译并安装：

  ```shell
  # 编译
  make -j线程数量
  
  # 安装
  make install
  ```

**注意事项：** `build` 文件夹是在 gcc 的解压目录下创建的，即：`gcc解压目录/build`

采用方式一安装时，虽然下载了所需版本的 gmp、mpfr、mpc ，并且在 configure 里面指定了所在的路径，但还是提示无法找到这些依赖。具体错误信息如下 ：

```shell
checking for the correct vresion of gmp.h... yes
checking for the correct version of mpfr.h... no
configure: error: Building GCC requires GMP 4.2+, MPFR 2.4.0+ and MPC 0.8.0+.
Try the --with-gmp, --with-mpfr and/or --with-mpc options to specify
their locations. Source code for these libraries can be found at
their respective hosting sites as well as at
ftp://gcc.gnu.org/pub/gcc/infrastructure/.
```

因此，如果有遇到同样问题的，可尝试使用方式二安装。



## 4. 方式二（不编译安装依赖包）

本人在使用方式一安装  gcc 时，未能成功安装，因此本人想到采用方式二安装。需要说明的是，下方提到的 **gcc 的解压目录下**全部是`~/ftp/gcc4.9`。

### 4.1 解压并链接 gmp

- 解压 **gmp**，然后将解压后的文件夹移到 **gcc 的解压目录下**（`~/ftp/gcc4.9`）

  ```shell
  tar xjf gmp-4.3.2.tar.bz2
  mv gmp-4.3.2 ~/ftp/gcc4.9/
  ```

- 进入 gcc 的解压目录，创建链接

  ```shell
  cd ~/ftp/gcc4.9/
  ln -sf gmp-4.3.2 gmp
  ```

### 4.2 解压并链接 mpfr

- 解压 **mpfr**，然后将解压后的文件夹移到 gcc 的解压目录下

  ```shell
  tar xjf mpfr-2.4.2.tar.bz2
  mv mpfr-2.4.2 ~/ftp/gcc4.9/
  ```

- 进入 gcc 的解压目录，创建链接

  ```shell
  cd ~/ftp/gcc4.9/
  ln -sf mpfr-2.4.2 mpfr
  ```

### 4.3 解压并链接 mpc

- 解压 mpc，然后将解压后的文件夹移到 gcc 的解压目录下

  ```shell
  tar xzf mpc-0.8.1.tar.gz
  mv mpc-0.8.1 ~/ftp/gcc4.9/
  ```

- 进入 gcc 的解压目录，创建链接

  ```shell
  cd ~/ftp/gcc4.9/
  ln -sf mpc-0.8.1 mpc
  ```

### 4.4 解压并链接 isl

- 解压 isl，然后将解压后的文件夹移到 gcc 的解压目录下

  ```shell
  tar xjf isl-0.12.2.tar.bz2
  mv isl-0.12.2 ~/ftp/gcc4.9/
  ```

- 进入 gcc 的解压目录，创建链接

  ```shell
  cd ~/ftp/gcc4.9/
  ln -sf isl-0.12.2 isl
  ```

### 4.5 解压并链接 cloog

- 解压 cloog，然后将解压后的文件夹移到 gcc 的解压目录下

  ```shell
  tar xzf cloog-0.18.1.tar.gz
  mv cloog-0.18.1 ~/ftp/gcc4.9/
  ```

- 进入 gcc 的解压目录，创建链接

  ```shell
  cd ~/ftp/gcc4.9/
  ln -sf cloog-0.18.1 cloog
  ```

### 4.6 安装 gcc

- 在 gcc 的解压目录下，创建临时文件夹，用于编译：`mkdir build`

- 配置编译参数：

  ```shell
  cd build
  
  # 配置编译参数
  ../configure \
  --prefix=~/soft/gcc49/gcc \
  --enable-threads=posix \
  --disable-checking \
  --enable--long-long \
  --disable-multilib \
  --enable-languages=c,c++
  ```

- 编译并安装：

  ```shell
  # 编译
  make -j线程数量
  
  # 安装
  make install
  ```

**注意事项：**

- 如果系统中无 32 位头文件，需要在 **configure** 的参数里面指定 `--disable-multilib`
- `build` 文件夹是在 gcc 的解压目录下创建的，即：`~/ftp/gcc4.9/build`
- 在编译时，尽可能使用更多线程去编译，否则会很慢



## 5. 总结

在正常情况下，如果机器能够连接外网，源码安装 gcc 的步骤如下：

- 执行 gcc 解压目录下的 `/contrib/download_prerequisites`，自动下载依赖
- 配置 gcc 的编译参数
- 编译、安装

因为使用的机器无法连接外网，所以自动下载依赖无法完成。而在方式二中，相当于是人为手动地执行了  `/contrib/download_prerequisites` 脚本的内容，从而完成依赖包的下载和软链接的创建。最后，依赖包以及 gcc 的编译、安装全部交给 gcc 的编译脚本自动完成。