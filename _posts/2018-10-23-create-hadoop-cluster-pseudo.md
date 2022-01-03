---
layout: post
title: "创建 Hadoop 集群之伪分布式"
categories: 分布式
tags: 大数据 hadoop 伪分布式
author: ZekeTian
---

* content
{:toc}


在本篇博客中将介绍如何在虚拟机中配置伪分布式 Hadoop 集群以及在配置过程需要注意的问题。采用的环境如下：服务器为 Ubuntu 18.04 Server 64 位版本，Hadoop 版本为 2.7.7，JDK 版本为 1.8，编程环境为 Windows 10 + IntelliJ IDEA



## 1. 配置 Linux 系统环境(以 Ubuntu Server 为例)
#### 1.1 创建用户及设置密码

以管理员用户身份运行以下命令，新增用户并为其设置密码

~~~shell
# 新增用户(-d 指定新增用户家目录, -m 自动建立目录)
sudo useradd -d /home/zeke -s /bin/bash -m  zeke

# 为新增用户 zeke 设置密码
sudo passwd zeke

# 切换用户
su zeke
~~~

#### 1.2 使用 XShell 连接 Ubuntu Server

- 前往 [XShell](https://www.netsarang.com/download/main.html) 官网下载 XShell，以便在 Windows 系统中连接 Ubuntu Server 使用。

- 在 Ubuntu 系统中获取 ip 地址

  输入`ifconfig`命令，在显示的信息中找到 `inet 192.168.*.*`信息，其中 `192.168.*.*` 即为虚拟机中的 Linux 系统 ip 地址

- 在 XShell 中新建会话，将上面得到的 ip 地址复制到新建会话对话框中，创建新会话

**注意：**在 Ubuntu 中新增用户默认使用 **/bin/sh** ，不支持 tab 自动补全命令，因此在新增用户时要指定使用 shell 版本。如果不确定自己的 shell 版本，可以使用`echo $SHELL` 来查看。如果已经建好用户，后期可通过管理员帐户使用`sudo usermod -s /bin/bash 用户名`指定。

#### 1.3 配置FTP服务传递文件

- 安装 vsftpd
  使用管理员用户登录 Linux 系统，输入`sudo apt-get install vsftpd` 安装 vsftpd

- 配置 vsftpd

  输入如下命令，编辑 vsftpd.conf 文件

  ~~~shell
  # 备份配置文件
  sudo cp /etc/vsftpd.conf /etc/vsftpd-bak.conf
  
  # 编辑配置文件
  sudo vi /etc/vsftpd.conf
  ~~~

  在 vsftpd.conf 文件中，进行如下修改

  ```shell
  1. 允许写操作（找到如下内容，去掉 write_enable 的前面的注释符）
  # Uncomment this to enable any form of FTP write command.
  write_enable=YES
  
  2. 允许 Ubuntu 用户直接登录（找到如下内容，去掉chroot_local_user、chroot_list_enable、chroot_list_file 前面的注释符）
  # (Warning! chroot'ing can be very dangerous. If using chroot, make sure that
  # the user does not have write access to the top level directory within the
  # chroot)
  chroot_local_user=YES
  chroot_list_enable=YES
  # (default follows)
  chroot_list_file=/etc/vsftpd.chroot_list
  
  3. 设置 ftp 上传下载文件路径（在最后面添加上如下内容）
  local_root=/home/zeke/ftp
  
  ```

- 创建 vsftpd.chroot_list 文件

  输入如下命令：

  ```shell
  # 在 /etc 目录下新建 vsftpd.chroot_lis 文件
  sudo vi /etc/vsftpd.chroot_list
  
  # 创建文件后，输入 ftp 服务登录用户，例如如下：
  zeke
  ```

- 创建 ftp 上传下载目录

  切换到普通用户，输入如下命令

  ```shell
  # 新建目录
  mkdir /home/zeke/ftp/
  
  ```

- 重启 vsftpd 服务

  切换到管理员用户，输入如下命令

  ```shell
  sudo service vsftpd restart
  ```

- 在 windows 下通过 ftp 访问 Ubuntu 中的文件

  第一种方式：在资源管理器中输入 “ftp://Ubuntu系统ip地址”，输入vsftpd.chroot_list 文件中配置的用户及密码。

  第二种方式：下载专用 ftp 软件，如 FileZilla，然后配置 ftp 连接即可连接 Ubuntu (使用 FileZilla 软件若中文乱码，可进行如下设置，文件->站点管理，选择相应站点，在右侧点击“字符集”，设置为“强制UTF-8”)

**注意：**在 vim 的命令模式下，输入 `/`可以查找内容

## 2. 配置 JDK 环境

### 2.1 安装 jdk

- 上传 jdk 压缩包到服务器

  下载好 Linux 版本的 jdk （bin.tar.gz 版本），通过 ftp 上传到 Ubuntu 服务器中，使用`tar -xzvf jdk压缩包名字.tar.gz `解压。

- 创建 soft 目录

  使用如下命令，创建 soft 目录存放安装的软件

  ```shell
  # 切换到管理员用户
  su 管理员用户
  sudo mkdir /soft
  
  # 更改 /soft 文件权限为普通用户
  sudo chown zeke:zeke /soft
  
  # 将 ftp 目录下解压的 jdk 文件移动到 soft 中
  mv /home/zeke/ftp/jdk文件夹名 /soft/ 
  
  # 切换到普通用户
  su zeke(普通用户)
  ```

- 创建软链接

  创建软链接，以便后期使用

  ```shell
  ln -s /soft/jdk1.8.0_191/ /soft/jdk
  ```

- 验证 jdk 是否安装成功

  ```shell
  # 输入如下命令，如果显示 java 版本信息则安装成功
  cd /soft/jdk/bin/
  ./java -version
  ```

### 2.2 配置 java 环境变量

- 编辑 /etc/profile

    ```shell
    # 切换到管理员用户
    su 管理员用户
    
    # 编辑 /etc/profile 文件
    sudo vim /etc/profile
    
    # 在 profile 文件最后添加如下内容
    export JAVA_HOME=/soft/jdk
    export PATH=$PATH:$JAVA_HOME/bin
    ```

- 使环境变更立即生效

    输入如下命令

    ```shell
    # 使环境命令立即生效
    source /etc/profile
    
    # 验证环境变量是否配置成功
    cd ~ # 切换到除 /soft/jdk/bin 外的目录下
    java -version # 如果显示版本信息则配置成功
    ```

## 3. 配置 Hadoop 环境
### 3.1 安装 Hadoop
- 上传 Hadoop 压缩包到服务器
  前往 [Apache 官网下载](https://hadoop.apache.org/releases.html) 下载 Hadoop （二进制 tar.gz 版本），通过 ftp 上传到 Ubuntu 服务器中，使用`tar -xzvf hadoop文件名.tar.gz `解压。
 - 复制到 soft 目录
	```shell
	mv /home/zeke/ftp/解压后的hadoop文件名 /soft/
	```
- 创建软链接 
  创建软链接，以便后期使用 

  ```shell
  ln -s /soft/hadoop文件名/ /soft/hadoop
  ```


- 验证 hadoop 是否安装成功
  ```shell
  # 输入如下命令，如果显示 hadoop 版本信息则安装成功
  cd /soft/hadoop/bin/
  ./hadoop version
  ```

### 3.2 配置 Hadoop 环境变量

- 编辑 /etc/profile
	```shell
  # 切换到管理员用户
  su 管理员用户
      
  # 编辑 /etc/profile 文件
  sudo vim /etc/profile
      
  # 在 profile 文件最后添加如下内容
  export HADOOP_HOME=/soft/hadoop
  export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
	```

- 使环境变更立即生效
  输入如下命令  

  ```shell
  # 使环境命令立即生效
  source /etc/profile
  
  # 验证环境变量是否配置成功
  cd ~ # 切换到除 /soft/hadoop/bin 外的目录下
  hadoop version # 如果显示版本信息则配置成功
  ```

### 3.3 配置 Hadoop 

为方便后期使用软链接在三种模式下互相切换，使用命令 **cp** 命令将 **/soft/hadoop/etc/hadoop** 文件夹各单独复制一份进行配置。    

- 单机版（本地文件系统，不需要启动单独进程）

  不需要做其它事情，完成 jdk、hadoop 的安装及配置步骤即为单机版。

  使用`cp -r /soft/hadoop/etc/hadoop/ /soft/hadoop/etc/local`单独复制一份（也可以达到备份的效果）

- 伪分布式版

  使用`cp -r /soft/hadoop/etc/hadoop/ /soft/hadoop/etc/pseudo`单独复制一份进行配置。

  修改 /soft/hadoop/etc/pseudo 下的如下文件，将文件内容设置为如下所示。

  ```xml
  <?xml version="1.0"?>
  <!-- core-site.xml -->
  <configuration>
  	<property>
          <name>fs.defaultFS</name>
          <value>hdfs://localhost/</value>
  	</property>
      <!-- hadoop 的临时目录 -->
      <property>
          <name>hadoop.tmp.dir</name>
          <value>/home/hadoop/hdpdata</value>
       </property>
  </configuration>
  
  <?xml version="1.0"?>
  <!-- hdfs-site.xml -->
  <configuration>
  	<property>
  		<name>dfs.replication</name>
  		<value>1</value>
  	</property>
      <!-- HDFS 的数据存储路径，如果有多个，用 "," 分割 -->
      <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/data/hadoop</value>
  	</property>
  </configuration>
  
  <?xml version="1.0"?>
  <!-- mapred-site.xml -->
  <configuration>
      <property>
          <name>mapreduce.framework.name</name>
          <value>yarn</value>
      </property>
  </configuration>
  
  <?xml version="1.0"?>
  <!-- yarn-site.xml -->
  <configuration>
      <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>localhost</value>
      </property>
      <property>
        <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
    </property>
  </configuration>
  ```
  
  **注意：**
  
  1）如果**不是在 Linux 系统中编程，而是在 Windows 系统中编程**，core-site.xml 中的 **fs.defaultFS** 的值应该设置为 **hdfs://具体ip地址/**（默认端口为8020）
  
  2） 配置 mapred-site.xml 时，需要使用 `cp mapred-site.xml.template mapred-site.xml` 命令得到 mapred-site.xml 文件。

## 4. 配置 SSH

因为集群中有许多主机，如果利用 SSH 登录需要密码，则非常繁琐，因此需要配置 SSH 无密登录。
### 4.1 检查 SSH 环境是否正常

如果安装的是 Ubuntu Server 版本，则 SSH 环境已经搭建好。如果不确定，则可以输入`apt list --installed  | grep ssh`命令，如果包含openssh-client、openssh-server、openssh-sftp-server、ssh-import-id 则正常（Ubuntu Server 版本默认已经安装好）

**注意：**Centos 环境中，输入`yum list installed | grep ssh` 命令，如果包含openssh-server、openssh-clients 、openssh 则正常。

### 4.2 检查 SSH  进程是否启动

输入命令`ps -ef | grep sshd` ，如果含有 sshd 进程信息则说明已经启动

### 4.3 在 SSH 客户端创建无密 SSH Key

输入如下命令

```shell
# rsa 指定 key 算法， -P '' 设置无密， -f 指定生成路径 
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
```

### 4.4 追加到~/.ssh/authorized_keys文件中

输入如下命令

```shell
cd .ssh/
cat id_rsa.pub >> authorized_keys
```

**注意：**如果是 Centos 则需要使用 `chmod 644 authorized_keys` 命令修改 authorized_kes 文件权限

### 4.5 验证

输入 `ssh localhost`，若配置成功，首次需要输入 yes 和密码，之后不需要输入密码。

如果仍然需要输入密码，则可能因为之前使用 XShell 登录过，自动创建了 ~/.ssh/known_hosts，此时删除 ~/.ssh 文件夹下的所有的文件，重复上面的步骤进行一次。

## 5. 启动伪分布式

### 5.1 切换模式

使用软链接指定当前模式为伪分布式

```shell
cd /soft/hadoop/etc/

# 如果 /soft/hadoop/etc/ 下存在 hadoop 文件夹，则先删除该文件夹或者重命名为 hadoop-bak
mv hadoop hadoop-bak

# 使用软链接，让 hadoop 指向 pseudo
ln -s pseudo hadoop
```

### 5.2 运行 Hadoop

- 格式化文件系统

```shell
  hadoop namenode -format
```

- 指定 hadoop 运行的 java 环境

  ```shell
  # 编辑 hadoop-env.sh
  vim /soft/hadoop/etc/hadoop/hadoop-env.sh
  
  # 找到 export JAVA_HOME=${JAVA_HOME} ，将其修改为如下内容
  export JAVA_HOME=/soft/jdk
  ```

- 运行 hadoop

  ```shell
  # 启动 hadoop
  start-all.sh
  
  # 查看 java 运行的进程
  jps
  
  # 如果看到如下进程则启动成功
  NodeManager
  DataNode
  NameNode
  Jps
  ResourceManager
  SecondaryNameNode
  ```

- 创建 hdfs 目录

  ```shell
   # 查看 hdfs 文件系统，初次查看内容为空
   hadoop dfs -ls /
   
   # 创建目录, -p 是当不存在父目录时创建
   hadoop dfs -mkdir -p /user/zeke/hadoop
  ```

  **注意：** jdk 版本不宜太新，在写此篇博客时，最初使用刚发布的 jdk 11 （hadoop 版本为 2.7.7），使用该命令出错，将 jdk 更改为 jdk8 运行正常。

- 查看 hdfs 目录

  第一种方式是利用 `hadoop dfs -ls` 命令查看

  第二种方式在物理机系统中的浏览器中输入 `http://目标主机ip:50070`，进入  webui 界面查看

- 复制本地文件到 hdfs 文件系统中

  ```shell
  # 创建 hello.txt
  cd ~
  vim hello.txt
  # 在 hello.txt 中输入“hello,hadoop”
  
  # 将本地的 hello.txt 复制到 hdfs 文件系统中
  hadoop dfs -put hello.txt /user/zeke/hadoop
  ```

### 5.3 单节点启动

在 Master 上使用 `start-all.sh` 命令可以启动集群中所有的节点。但是，如果某个节点在 Hadoop 的运行过程中挂掉了，则需要单独启动此节点。具体方法如下：

- ssh 登录挂掉的节点
- 使用命令	`hadoop-daemon.sh start datanode` 启动



## 6. 编写代码测试伪分布式



在 Windows 下利用 idea 编写如下代码：

```java 
public class TestHdfs {

    @Test
    public void testRead() throws IOException {
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(conf);
        Path path = new Path("hdfs://ip地址/user/zeke/hadoop/hello.txt");
        FSDataInputStream fdis = fs.open(path);
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        IOUtils.copyBytes(fdis, baos, 1024);

        System.out.println(new String(baos.toByteArray()));

    }
}

```

**注意：**如果遇到**“java.net.ConnectException: Connection refused: no further information”**问题，可参考[此篇博客](https://blog.csdn.net/s740556472/article/details/81263426?utm_source=blogxgwz4)。在伪分布式配置中，编程环境为 Windows但在 core-site.xml中配置成如下内容，则 Linux 服务器不能对外提供服务，需要将 fs.defaultFS 对应的 value 更改成服务器的 ip 地址。

```xml
<?xml version="1.0"?>
<!-- core-site.xml -->
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://localhost/</value> <!-- 此处需要更改成具体 ip 地址，默认端口为8020，具体内容可看参考博客 -->
</property>
</configuration>
```



## 完全分布式的问题

live node 为 0 ，检查 slave 的日志文件，看是否出现如下警告：

```
WARN org.apache.hadoop.hdfs.server.datanode.DataNode: Problem connecting to server: master:9000
```

- 检查 slave 机器上的 /etc/hosts 文件中是否配置了 master 的 IP
- 检查 master 的端口是否开放

集群中的主机名应当在所有机器的 /etc/hosts 里面都要添加