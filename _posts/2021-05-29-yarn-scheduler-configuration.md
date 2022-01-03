---
layout: post
title: 配置 Yarn 的资源调度器
categories: 分布式
tags: 大数据 Hadoop
author: ZekeTian
---





* content
{:toc}
本文主要讲解如何配置 Hadoop 中的 Yarn 的资源调度器。注意，本文方法的 Hadoop 环境是 **Hadoop 2.7.7**。



## 1. Yarn中的资源调度器

Yarn 中的资源调度器有三种类型，分别为：**FIFIO Scheduler**、**Capacity Scheduler**、**Fair Scheduler**，它们各自的调度策略如下图所示。

![调度器示意图](https://zeke-images.oss-cn-hongkong.aliyuncs.com/blog/2021-05-29-yarn-scheduler-configuration-img1.png)

接下来，分别具体介绍三个调度器各自的调度策略。

- **FIFO Scheuduler**

  FIFO Scheduler 调度器采取**先进先出**调度策略，即先提交的任务先执行，后提交的任务后执行

  优点：简单易用，不需要配置

  缺点：不适合共享集群。因为一个大的应用可能会占用所有集群资源，这就导致其它应用被阻塞。

- **Capacity Scheduler**

  Capacity Scheduler 调度器将集群资源池切分成多个小资源池，每个小资源池是一个任务队列（内部相当于是一个 FIFO Scheduler），各个小资源池所占的比例可以通过配置文件修改（这里的比例即体现了 Capacity 的含义）。

  用户在提交任务时，可以指定将任务提交到哪个任务队列中。

- **Fair Scheduler**

  Fair Scheduler ，顾名思义，其会公平地让每个任务都得到集群资源，从而使得每个任务都可以执行。当第一个大任务提交时，其会独占集群所有的资源，当第二个小任务提交时，Fair Scheduler 会将部分资源分给第二个小任务。

**在 Apache Hadoop 中，默认的调度器是 Capacity Scheduler**，可以在 Yarn 的 Web UI 界面查看调度器类型进行验证。查看调度器具体步骤如下。

- 在浏览器中输入 `http://master机器ip:8088/`，进入 Yarn 的 Web UI 界面，如下图所示

  ![查看调度器类型](https://zeke-images.oss-cn-hongkong.aliyuncs.com/blog/2021-05-29-yarn-scheduler-configuration-img2.png)

- 点击左侧列表中的 **“Scheduler”** 链接，即可查看当前的调度器类型。从图中可以看出，默认调度器类型为 Capacity



## 2. 配置调度器

在日常使用中，我们可以增加 Capacity Scheduler 的任务队列，如：在原来的基础上，添加一个 **offline** 队列用于处理离线任务，添加一个 **online** 队列用于处理在线任务。添加队列的步骤分别如下：

### 2.1 编辑配置文件

进入 `$HADOOP_HOME/etc/hadoop` 路径，编辑配置文件 **capacity-scheduler.xml**

### 2.2 添加任务队列

在 **capacity-scheduler.xml** 中，找到属性 **yarn.scheduler.capacity.root.queues** ，其默认值如下：

```xml
<property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default</value>
    <description>
        The queues at the this level (root is the root queue).
    </description>
</property>
```

将默认值修改成如下内容：

```xml
<property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,online,offline</value>
    <description>
        The queues at the this level (root is the root queue).
    </description>
</property>
```

原来的 value 从 **default** 更改成 **default,online,offline**，从而添加 online、offline 两个任务队列。

**注意事项：**设置多个队列时，队列与队列之间需要用逗号分隔。

### 2.3 指定任务队列所占资源比例

添加 online、offline 两个任务队列后，需要重新为 default、online、offline 三个队列设置所占资源比例（即容量）。

- **设置 default**

  default 在配置文件中有默认值，值如下：

  ```xml
    <property>
      <name>yarn.scheduler.capacity.root.default.capacity</name>
      <value>100</value>
      <description>Default queue target capacity.</description>
    </property>
  ```

  将默认值更改成如下：

  ```xml
    <property>
      <name>yarn.scheduler.capacity.root.default.capacity</name>
      <value>70</value>
      <description>Default queue target capacity.</description>
    </property>
  ```

- 设置 online

  在配置文件中添加如下内容：

  ```xml
    <property>
      <name>yarn.scheduler.capacity.root.online.capacity</name>
      <value>10</value>
      <description>Online queue target capacity.</description>
    </property>
  ```

  注意将 **yarn.scheduler.capacity.root.*default*.capacity** 中的 **defalut** 更改为 **online**

- 设置 offline

  在配置文件中添加如下内容：

  ```xml
    <property>
      <name>yarn.scheduler.capacity.root.offline.capacity</name>
      <value>20</value>
      <description>Offline queue target capacity.</description>
    </property>
  ```

  注意将 **yarn.scheduler.capacity.root.*default*.capacity** 中的 **defalut** 更改为 **offline**

### 2.4 指定任务队列可使用资源的上限

通过上面的步骤可以设置每个队列所占资源比例，但是还需要设置每个队列能够使用的资源上限。

- **设置 default**

  defalut 在配置文件中的默认值如下：

  ```xml
    <property>
      <name>yarn.scheduler.capacity.root.default.maximum-capacity</name>
      <value>100</value>
      <description>
        The maximum capacity of the default queue.
      </description>
    </property>
  ```

  将默认值更改成如下：

  ```xml
    <property>
      <name>yarn.scheduler.capacity.root.default.maximum-capacity</name>
      <value>70</value>
      <description>
        The maximum capacity of the default queue.
      </description>
    </property>
  ```

- 设置 online

  在配置文件中添加如下内容：

  ```xml
    <property>
      <name>yarn.scheduler.capacity.root.online.maximum-capacity</name>
      <value>10</value>
      <description>
        The maximum capacity of the online queue.
      </description>
    </property>
  ```

  注意将 **yarn.scheduler.capacity.root.*default*.maximum-capacity** 中的 **default** 更改为 **online**

- 设置 offline

  在配置文件中添加如下内容：

  ```xml
    <property>
      <name>yarn.scheduler.capacity.root.offline.maximum-capacity</name>
      <value>20</value>
      <description>
        The maximum capacity of the offline queue.
      </description>
    </property>
  ```

  注意将 **yarn.scheduler.capacity.root.*default*.maximum-capacity** 中的 **default** 更改为 **offline**

### 2.5 验证是否配置成功

在浏览器中输入 `http://master机器ip:8088/`，重新进入 Yarn 的 Web UI 界面中，可以看到任务队列中出现了 **offline**、**online** 两个新队列，说明配置成功。

![验证调度器配置结果](https://zeke-images.oss-cn-hongkong.aliyuncs.com/blog/2021-05-29-yarn-scheduler-configuration-img3.png)

## 3. 提交任务指定 Capacity Scheduler 的任务队列

在前面，我们已经为 **Capacity Scheduler** 添加了两个新的任务队列。在提交任务时，我们可以通过参数指定任务提交到哪个任务队列中。

如，将任务提交到 **offline** 任务队列中，则添加运行参数`-Dmapreduce.job.queuename=offine`

