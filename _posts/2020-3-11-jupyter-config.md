---
layout: post
title: 配置 Jupyter，享受更好的编码体验
categories: 机器学习
tags: Jupyter
author: ZekeTian
---




* content
{:toc}


上一篇博客介绍了 Jupyter 的两种安装方式，而在本文中，我将介绍 Jupyter 的一些常用配置，提升我们的日常编码体验。



## 1. Jupyter Notebook 参数配置

在 Jupyter Notebook 的参数配置中，我将分别介绍“个人电脑”和“服务器”两种环境的一些常用配置。

### 1.1 个人电脑运行 Jupyter 的相关配置

在个人电脑的 Windows 系统上运行 Jupyter 时，默认打开的文件夹是 C 盘的用户目录。然而，我们的代码通常是在另外一个专用目录下，这样就非常不方便我们打开或保存代码文件。因此，我们需要修改 Jupyter Notebook 的默认文件夹路径。

#### 1.1.1 修改 Jupyter Notebook 的默认文件夹路径

- 创建新文件夹
  在系统中创建一个文件夹存储 Jupyter Notebook 的文件夹，复制该文件夹的路径

- 生成默认配置文件

  打开 Anaconda 的命令行窗口（Anaconda 安装）或  系统的命令行窗口（pip 安装），运行如下命令：

  `jupyter notebook --generate-config`

  执行上述命令后，会提示生成 **“C:\Users\用户名\\.jupyter\jupyter_notebook_config.py”** 配置文件

- 修改配置文件

  根据上一步的提示信息，找到生成的配置文件，打开后搜索**“c.NotebookApp.notebook_dir”**，删除注释符号，并将前面创建的文件夹路径复制粘到此处

**注意：**

Windows 系统的文件路径中的 **“\”** 要换成 **“\\\\”** 或 **“/”**，否则会报 **“ValueError: lstat: embedded null character in path”** 错误。

### 1.2 服务器运行 Jupyter 的相关配置

在服务器环境中，为了安全，我们需要为 Jupyter 设置密码，并且在本地进行远程访问。接下来，我将介绍这些相关的设置。

#### 1.2.1 设置密码

当使用服务器的 Jupyter Notebook 编写代码时，需要设置代码，具体步骤如下：

- 生成密码

  打开 Anaconda 的命令行窗口（Anaconda 安装）或  系统的命令行窗口（pip 安装），运行如下命令：

  `jupyter notebook password`

  执行上述命令后，会提示 **“Wrote hashed password to \xxx\yyy\\\.jupyter\jupyter_notebook_config.json ”** 

- 复制生成的哈希密码

  根据上一步的提示信息，找到生成的文件，打开后复制生成的密码（即 json 文件中 “password” 对应的 value 值，以 “sha1” 开头）。

- 修改配置文件

  打开 Jupyter Notebook 的配置文件，搜索 **“c.NotebookApp.password”**，删除注释符号，按如下格式设置：

  `c.NotebookApp.password = u'生成的哈希密码'`

  **注意：**

  密码前面有一个 **“u”**

#### 1.2.2 开启远程访问

由于服务器一般运行的是无界面的 Linux 系统，因此需要在本地浏览器输入服务器的 ip 来远程访问 Jupyter Notebook 。为了能够远程访问，需要对服务器的 Jupyter 进行如下相关配置。以下的设置都是在 Jupyter Notebook 的配置文件上进行修改，所以需要先打开配置文件。

- 关闭浏览器显示

  由于服务器无界面，因此需要关闭浏览器显示，打开配置文件，搜索 **“c.NotebookApp.open_browser”**， 将值设置成如下：

  `c.NotebookApp.open_browser = False`

- 指定端口（为了安全，建议修改）

  `c.NotebookApp.port = portNumber #可自行指定一个端口, 访问时使用该端口`

- 解决 ip 限制

  ```python
  c.NotebookApp.allow_origin = '*'
  # 是否允许远程访问
  c.NotebookApp.allow_remote_access = True
  # Jupyter Notebook Server监听的IP
  c.NotebookApp.ip = '127.0.0.1'
  
  # c.NotebookApp.ip 也可以设置成如下
  c.NotebookApp.ip = '0.0.0.0' # 监听所有 IP ，默认是127.0.0.1或localhost
  ```


## 2. 主题设置

Jupyter Notebook 的默认主题是白色的，而且字体略微有点小。当写了太长时间的代码，眼睛会比较累。这个时候，最好的方式是将主题换成黑色或者深颜色。

然而，不幸的是，Jupyter Notebook 本身是不能直接在设置中更换主题的。为了更换主题，我们需要做一些额外的工作。接下来，跟着下面的步骤，换一个让眼睛更舒服的主题吧！

### 2.1 安装 jupyter-themes

- 打开命令行窗口

  根据自己的 Jupyter Noetbook 的安装方式，选择 Anaconda 的命令行窗口或 Windows 系统的 CMD。

- 输入安装命令

  `pip install jupyterthemes`

- 检查是否安装成功

  输入如下命令：

  `jt -l`

  如果出现如下提示信息，则安装成功。

  ```
  Available Themes:
     chesterish
     grade3
     gruvboxd
     gruvboxl
     monokai
     oceans16
     onedork
     solarizedd
     solarizedl
  ```

### 2.2 设置主题

安装完 jupyter-themes 后，不要关闭命令行窗口，因为需要通过输入命令来设置主题。从上面 `jt -l` 命令的提示信息来看，总共有 9 种主题可以选择。我们可通过 `jt -t 主题名` 命令来逐一尝试这些主题，直到找到自己喜欢的。

关于这些主题，喜欢亮色主题的朋友可以尝试下**“grade3”**，喜欢深色的可以尝试下**“chesterish”** 、**“monokai”**、**“onedork ”**。另外，在[蝈蝈](https://zhuanlan.zhihu.com/p/46242116)的这篇文章中，他也介绍了主题的设置方法，并且对 **“chesterish”** 、**“grade3”**、**“gruvboxd”**、**“gruvboxl”**、**“monokai”**、**“onedork ”** 这几种主题进行了截图展示。大家可以先从他的截图中预览一下，快速选出自己喜欢的。 

我个人喜欢深色的主题，因此选择了一个黑色的。我使用的主题设置命令如下，可供大家参考。

```
jt --lineh 140 -f source -t onedork -ofs 14 -nfs 14 -tfs 14 -fs 14 -T -N
```

关于主题设置命令的参数介绍，可以参照下表（详情请看 [jupyter-themes GitHub 官网](https://github.com/dunovank/jupyter-themes)）：

| cl options            | arg     | default |
| --------------------- | ------- | ------- |
| Usage help            | -h      | --      |
| List Themes           | -l      | --      |
| Theme Name to Install | -t      | --      |
| Code Font             | -f      | --      |
| Code Font-Size        | -fs     | 11      |
| Notebook Font         | -nf     | --      |
| Notebook Font Size    | -nfs    | 13      |
| Text/MD Cell Font     | -tf     | --      |
| Text/MD Cell Fontsize | -tfs    | 13      |
| Pandas DF Fontsize    | -dfs    | 9       |
| Output Area Fontsize  | -ofs    | 8.5     |
| Mathjax Fontsize (%)  | -mathfs | 100     |
| Intro Page Margins    | -m      | auto    |
| Cell Width            | -cellw  | 980     |
| Line Height           | -lineh  | 170     |
| Cursor Width          | -cursw  | 2       |
| Cursor Color          | -cursc  | --      |
| Alt Prompt Layout     | -altp   | --      |
| Alt Markdown BG Color | -altmd  | --      |
| Alt Output BG Color   | -altout | --      |
| Style Vim NBExt*      | -vim    | --      |
| Toolbar Visible       | -T      | --      |
| Name & Logo Visible   | -N      | --      |
| Kernel Logo Visible   | -kl     | --      |
| Reset Default Theme   | -r      | --      |
| Force Default Fonts   | -dfonts | --      |

**注意：**

设置了主题后，记得重启 Jupyter Notebook ，从而让主题生效。

## 3. 插件管理

默认情况下，我们在 Jupyter Notebook 中写代码时，可以通过**“Tab”** 键来补全代码。但是，这与 VS Code 中的代码自动补全相比，还是有点不方便。为了在 Jupyter Notebook 中能够使用代码自动补全功能，我们需要安装插件。

### 3.1 安装插件

在安装 Jupyter 插件时，我们首先需要打开命令行窗口。与前面 **2.1** 一样，根据自己的 Jupyter 安装方式，打开对应的命令行窗口，然后输入如下命令：

```python
# 安装 nbextensions
pip install jupyter_contrib_nbextensions

# 安装 JavaScript 和 CSS
jupyter contrib nbextension install --user
```

### 3.2 开启插件

安装完插件后，重新启动 Jupyter ，我们可以发现在 Jupyter 首页的顶部导航栏中多了一项**“Nbextensions”**。开启插件的方法如下：

- 点击**“Nbextensions”**选项卡，进入如下图所示的界面

- 勾选 **“Hinterland”**

  如果不能勾选 “Hinterland”，请检查左上角**“Configurable nbextensions”**下面的复选框是否被选中。如果该复选框被选中，将其禁止掉，然后再勾选“Hinterland”。

![开启插件](https://zeke-images.oss-cn-hongkong.aliyuncs.com/blog/2020-3-12-jupyter-config-img1.png)

上面是通过可视化界面来开启插件，实际也可通过命令行开启，具体用法可参考 [jupyter 插件项目 GitHub 官网](https://github.com/ipython-contrib/jupyter_contrib_nbextensions) ，这里不再进行详细介绍。

## 参考文章

[1] [Jupyter 服务器配置](https://blog.csdn.net/u013687632/article/details/83658079)

[2] [jupyter-themes 项目 GitHub 网址](https://github.com/dunovank/jupyter-themes)

[3] [主题设置](https://zhuanlan.zhihu.com/p/46242116)

[4] [主题设置及插件安装](https://www.cnblogs.com/qiuxirufeng/p/9609031.html)

[5] [jupyter 插件项目 GitHub 官网](https://github.com/ipython-contrib/jupyter_contrib_nbextensions)

