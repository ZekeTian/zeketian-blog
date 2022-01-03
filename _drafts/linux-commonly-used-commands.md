 

## 文件相关

- 查看磁盘空间大小：`df -h`

- 扩大磁盘分区大小：`sudo resize2fs /dev/sda1`

  注意后面的 **/dev/sda1** 是待扩大分区所在的文件系统，可以通过 `df -h ` 或者 `lsblk` 获取，如果想指定扩大的容量，可以在后面指定具体容量的大小，如：`sudo resize2fs /dev/sda1 25G`。

  参考资料：https://askubuntu.com/questions/390769/how-do-i-resize-partitions-using-command-line-without-using-a-gui-on-a-server

- 查看当前目录下**各个文件的大小**：`du -h --max-depth=1`

- 查看当前目录的**总大小**：`du -sh`

- 超大文本文件分割：

  - 第一种用法：`split -d -l 50000 -a 2 --additional-suffix=.txt data.txt split_`

  - 第二种用法：`split -b 1G -a 2 --additional-suffix=.txt data.txt split_`

  命令参数含义：
  **-d** 使用数字作为后缀，如果不加该参数，则默认使用字母作为后缀

  **-l** 用行进行分割 （后面接分割的行数）

  **-b** 按照大小进行分割，值要为整数，单位可选。如果不指定单位，则默认为字节。可选的单位有： K, M, G, T, P, E, Z, Y (powers of 1024) 或 KB, MB, ... (powers of 1000)

  **--additional-suffix** 分割后的文件后缀名
  后面两个参数，一个是原始数据文件，一个是新文件的前缀

- 超大文本文件导出指定行内容

  - 导出文件的前多少行：`head -行数 读入的文本文件名 > 输出的文本文件名`

    示例：`head -10000 ./log.txt > ./tmp.txt `

  - 导出文件的后多少行：`tail -行数 读入的文本文件名 > 输出的文本文件名`

    示例：`tail -10000 ./log.txt > ./tmp.txt `

  - 导出文件的指定行：`sed -n '起始行,结束行p' 读入的文本文件名 > 输出的文本文件名`  

    示例：`sed -n '5000,10000p'./log.txt > ./tmp.txt` 

  **注意：**如果没有后面的 **“>”** 进行重定向，则直接输出到屏幕

- 合并多个文本文件：`cat 文件1 文件2 ... 文件n > 合并的文件名`

  示例：`cat log1.txt log2.txt log3.txt > log.txt`

- 查看文本文件的基本信息：`wc 参数 文件名` 

  可选的参数有：

  **-c** 文件的字节总数

  **-m** 文件的字符总数

  **-l** 文件的总行数

  **-L** 文件最长行的长度

  **-w** 文件的单词总数

- 清除临时文件：`tmpwatch -avf 3 /tmp`

  命令参数含义：

  **-a** 删除任何类型的文件

  **-v** 显示命令执行的过程

  **-f** 强制删除文件或目录

  **3** 三个小时以内没有使用的文件

  **/tmp** 待清理的目录
  
- 修改文件权限：`chmod`

  - 用法一：`chmod [R] 用户 +/- 权限 文件/文件夹`

    示例：`chmod a+w test.txt`，表示修改 test.txt 文件权限为：所有人对 test.txt 文件都有写权限。

    用户符号含义：u（user），表示文件所有者；g（group），表示文件所有者所在的用户组；o（other）表示其他人；a（all）表示所有人。

    **+/-**：“+” 表示添加权限，”-“表示移除权限

    权限符号含义：r，表示读权限；w，表示写权限；x，表示执行权限

  - 用法二：`chmod 权限数字 文件`

    示例：`chomd 755 test  ` ，表示**文件所有者**拥有**可读可写可执行**权限，**文件所有者所在的用户组**拥有**可读可执行**权限，**其他人**所有**可读可写可执行**权限。

  **注意：**如果是修改文件夹，则要加上参数 `-R`

- 修改文件的拥有者：`chown`

  - 用法一：`chown [-R] 用户名 文件/文件夹`
  - 用法二：`chown [-R] 用户名:用户组名称 文件/文件夹`

## 网络相关

- 端口管理相关命令

  - 开放端口
    `firewall-cmd --zone=public --add-port=16677/tcp --permanent`
    
  - 关闭端口
      `firewall-cmd --zone=public --remove-port=16677/tcp --permanent`
  - 使配置立即生效
    `firewall-cmd --reload`
  
  - 查看开放的端口号
    `firewall-cmd --zone=public --list-ports`
  
- 内网间传输数据

  - scp

    从远程服务器下载数据：`scp 用户名@远程服务器内网ip:文件路径 本地保存路径`，如：

    `scp user@192.168.27.1:/home/test.txt /home/`

    向远程服务器上传数据：`scp 本地文件路径 用户名@远程服务器内网ip:文件保存路径 `，如：

    `scp /home/test.txt user@192.168.27.1:/home/`

  - rsync

    断点续传下载：`rsync -avP 远程服务器内网ip:文件路径 本地保存路径`，如：

    `rsync -avP 192.168.27.1:/home/test.txt /home/ ` 

    `rsync -avP 172.26.36.5:/mnt2/tz/LargeRDF/large-rdf.tar.gz /home/tz/`

- 测试网络连接

  `curl -i url地址`



## 用户管理

- 显示当前登录的用户：`w`

  在终端输入 `w` 即可查登录的用户信息，包括用户名、终端号、登录 ip 、登录时间等信息。

- 踢出指定用户：`pkill -9 -t 用户终端号`

  注意：

  - 用户终端号是 `w` 命令中输出信息的 TTY 值。
  
- root 用户可以踢出任意用户，而其它用户只能踢出自己
  
- 查看用户组：`cat /etc/group`

- 查看所有用户列表：`cat /etc/passwd`

- 查看当前用户

  - 简略版（只显示名字）：`whoami`
  - 详细版：`who am i`




## 硬件信息

- 查看 CPU 信息

  - 统计信息：`lscpu`

  - 详细信息：`cat /proc/cpuinfo`

    - 获取物理 CPU 个数 ：`cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l`
    - 查看每个物理 CPU 中的核数：`cat /proc/cpuinfo| grep "cpu cores"| uniq`
    - 查看逻辑 CPU 的个数：`cat /proc/cpuinfo| grep "processor"| wc -l`
    - 查看 CPU 型号：`cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c`

    计算公式：

    ```
    总核数 = 物理CPU个数 X 每颗物理CPU的核数 
    总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数
    ```
    
    一般情况下，使用 `lscpu` 命令即可获得需要的 CPU 信息
  
- 查看内存信息

  - 命令一：`free -h` （单位：GB），`free -m`（单位：MB）
  - 命令二：` cat /proc/meminfo | grep MemTotal`
  - 命令三：`top`

- 查看硬盘信息

  - 查看分区情况：`lsblk`
  
- 查看容量
  
    - 命令一：`df -h`
    - 命令二：`fdisk -l`
    
    
  命令一是查看文件系统中的容量情况；而命令二可以查看所有的分区，通过分区数可以确定出硬盘的数量（通过命令一无法获取硬盘数量）。
  
- 添加硬盘

  在 Linux 系统中挂载容量小于 2TB 的数据盘，操作步骤如下：

  - 查看系统的数据盘：`fdisk -l`

  - 创建分区

    - 运行 `fdisk -u /dev/sdb` 命令分区数据盘（如果没有 **/dev/sdb**，则需要检查是否挂载了新数据盘）
    - 输入`p`查看数据盘的分区情况
    - 输入`n`创建一个新分区
    - 输入`p`选择分区类型为主分区
    - 输入分区编号并按回车键。在此示例中，仅创建一个分区，输入`1`
    - 输入第一个可用的扇区编号，按回车键采用默认值2048
    - 输入最后一个扇区编号。本示例中，仅创建一个分区，按回车键采用默认值
    - 输入`p`查看该数据盘的分区情况
    - 输入`w`开始分区，并在完成分区后退出

    以上命令的输出详细信息如下：

    ```
    fdisk -u /dev/sdb
    
    Welcome to fdisk (util-linux 2.31.1).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.
    
    Device does not contain a recognized partition table.
    Created a new DOS disklabel with disk identifier 0xe175c123.
    
    Command (m for help): p
    Disk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xe175c123
    
    Command (m for help): n
    Partition type
       p   primary (0 primary, 0 extended, 4 free)
       e   extended (container for logical partitions)
    Select (default p): p
    Partition number (1-4, default 1): 1
    First sector (2048-41943039, default 2048):
    Last sector, +sectors or +size{K,M,G,T,P} (2048-41943039, default 41943039):
    
    Created a new partition 1 of type 'Linux' and of size 20 GiB.
    
    Command (m for help): p
    Disk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xe175c123
    
    Device     Boot Start      End  Sectors Size Id Type
    /dev/sdb1        2048 41943039 41940992  20G 83 Linux
    
    Command (m for help): w
    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Syncing disks.
    ```

    

  - 查看新分区：`fdisk -lu /dev/sdb`。如果出现 **/dev/sdb1** 的相关信息，表示新分区已创建完成

  - 在新分区上创建一个文件系统

    - 创建一个 ext4 文件系统：`mkfs -t ext4 /dev/sdb1`
    - 创建 VFAT 文件系统：`mkfs -t vfat /dev/sdb1`，该文件系统也称为 **FAT32**，可被 Linux、Windows 和 Mac 系统访问

  - 写入新分区信息，启动开机自动挂载分区

    - 运行命令`cp /etc/fstab /etc/fstab.bak`，备份 etc/fstab

    - 运行如下命令，向 /etc/fstab 里写入新分区信息

      ```
      echo `blkid /dev/sdb1 | awk '{print $2}' | sed 's/\"//g'` /mnt ext4 defaults 0 0 >> /etc/fstab
      ```

      注意：

      - **blkid** 命令用于为新分区创建唯一的 UUID。
      - 如果要把数据盘单独挂载到某个文件夹，例如单独用来存放网页，则将命令中`/mnt`替换成所需的挂载点路径。但是需要注意的是：**挂载点路径如果不存在，则需要提前创建好**。

  - 查看新分区的信息：`cat /etc/fstab`

  - 挂载文件系统：`mount /dev/sdb1 /mnt`。注意：此处的 **/mnt** 是之前实际挂载点路径。

  - 检查文件系统是否挂载成功：`df -h`

  

  参考资料：https://help.aliyun.com/document_detail/25426.html?spm=a2c4g.11186623.2.20.29d344e6i4jbxj#concept-jl1-qzd-wdb








## 系统信息

- 查看内核、操作系统信息：`uname -a`
- CentOS7 永久修改主机名：`hostnamectl set-hostname 主机名`
- 查看 CentOS 系统版本：`cat /etc/redhat-release`
- 查看系统运行时长：`uptime`
- 列出**所有**用户最近登录的信息：`lastlog`。读取 **/var/log/lastlog** 文件中的信息。
- 列出**失败尝试**的登录信息（**失败登录**）：`lastb`。和last命令功能完全相同，只不过它默认读取的是 **/var/log/btmp** 文件的信息，所以其等同于`last -f /var/log/btmp`。当 **/var/log/btmp** 文件过大时，会导致 SSH 登录变慢。因此，当 SSH 登录时较慢，可以尝试清空 **/var/log/btmp** 文件的内容。
- 列出当前和曾经**登入系统**的用户信息（**成功登录**）：`last`。默认读取的是 **/var/log/wtmp** 文件的信息。
- 检查系统是否安装 GUI：`ps -A | egrep -i "gnome|kde|mate|cinnamon|lx|xfce|jwm"`
- 列出所有定时任务：`crontab -l`





## 其它命令

- 后台执行命令

  SSH 连接到服务器后，如果直接在终端输入命令并执行，则需要保证该 SSH 连接不能断开。一旦因为网络异常导致连接断开，则该任务会异常停止。为了避免这种情况的发生，可以使用 `screen` 工具，让命令在后台执行，从而将 SSH 连接与命令的执行分开。使用 `screen` 工具的步骤如下：

  - 安装

    Centos 系统：`yum install screen`

  - 创建一个窗口

    第一种方式，创建无名窗口：`screen`

    第二种方式，创建有名窗口：`screen -S name`，创建一个名字为 name 的窗口

  - 执行命令

    创建窗口后，可在窗口内执行相关命令

  - 退出窗口

    - 在会话内

      执行命令后，使用组合快捷键 `Ctrl + A + D`，可退出当前窗口。退出当前窗口后，命令会在后台继续执行，此时关闭 SSH 连接并不会影响命令的执行。

    - 不在会话内

      使用`screen -d name` 可以使会话从 **Attached** 状态转换成 **Detached** 状态，从而便于再次进入会话。

  - 恢复窗口

    在普通终端（即非窗口模式下的终端）中输入`screen -r 窗口名或窗口id` 可恢复指定的窗口。窗口恢复后，可查看命令的执行情况。

  - 查看窗口

    在普通终端中输入 `screen -ls` 可列出创建的所有窗口的信息，包括 id、名字、状态。

  - 关闭窗口
  
    - 在会话内
  
      如果想要结束当前窗口，使用组合快捷键 `Ctrl + A + K` 即可，或者输入 `exit`。
  
    - 不在会话内
  
      使用`screen -X -S name quit`
  
  - 查看历史信息
  
    输入 `Ctrl + A + [` 进入 **copy mode**，然后即可像 vi 一样使用，最后输入 `Esc` 即可退出该模式。
  
  - 清除窗口
  
    如果在 `screen -ls` 的输出信息中，某个窗口的状态为 **dead** ，则该窗口已经失效，可使用 `screen -wipe` 命令清除。
  
- 远程执行命令

  在当前机器中，控制远程机器执行某条命令。

  - 方式一：在当前机器中**登录到远程机器**内部，然后执行相应命令

  - 方式二：**不登录到远程机器**，直接使用 **ssh**  完成命令的远程执行

    - 执行不需要交互的命令

      ```shell
      # 执行一条命令
      ssh username@ip "命令"
      # eg: ssh root@xxx.xxx.xxx.xxx "ls ~/"
      
      # 执行多条命令
      ssh username@ip "命令1; 命令2;..."
      # eg: ssh root@xxx.xxx.xxx.xxx "df -h; ls ~"
      ```

      **注意：**命令的内容需要用双引号包起来

    - 输出返回结果

      ```shell
      echo `ssh username@ip 命令`
      ```

      **注意：**这里是反引号 **`**，命令可以不用双引号 **“** 包起来
    
    - 执行需要交互的命令
    
      ```shell
      ssh -t username@ip "命令"
      ```
    
      

- 统计命令执行的时间：`time 需要统计执行时间的命令`，如：`time ls`

- 记录命令执行过程：**script**

- 设置定时任务：**at**，使用步骤如下：

  - `at 时间`
  - 输入要执行命令
  - 按 `Ctrl + D` 或者 `Crtl + \` 键退出

- 设置周期任务：**crontab**



Dell 服务器查看物理硬盘步骤

开机，按 F2，然后依次进入如下路径

/Device Settings/Integrated RAID Controller/Physical Disk Management，将状态转换成 Non-RAID （原来是 ready 状态）

[Dell R740服务器设置磁盘直通,不做RAID虚拟磁盘阵列](https://my.oschina.net/u/3585265/blog/3065800)



