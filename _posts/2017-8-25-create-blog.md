---
layout: post
title: "创建个人博客"
categories: 经验总结
tags: Blog Jekyll
author: ZekeTian
---

* content
{:toc}


经过两天的折腾，终于利用Jekyll成功搭建了自己的个人博客。现在来记录下自己搭建博客的过程，既是个人的一个总结，也希望对想要搭建个人博客的朋友提供一些帮助。



## 一、准备工作

### 1、选择提供博客托管服务的第三方

提供个人博客托管服务的第三方网站有 GitHub、 Coding 。

GitHub 与 Coding的比较：

（1）访问速度：GitHub 服务器在国外，所以访问速度可能较 Coding 慢

（2）个性域名：两者均支持用户自定义的个性域名

（3）SSL证书：GitHub原生的博客网址支持SSL证书，若用户自定义域名					之后，SLL证书失效（可通过第三方添加SSL证书）。相比，Coding自定义的域名支持SSL证书。

（4）跳转广告：GitHub无跳转广告，Coding在用户自定义域名之后，访问博客之前会显示几秒钟的Coding Pages跳转页，用户若想去掉该跳转页，需要网站上添加“Hosted by Coding Pages”说明

在选定之后，只需要先注册一个用户即可。

### 2、注册个性域名（此步也可省略）

用户在选择第三方之后，可以选择注册一个域名，让别人更容易记住自己的博客地址。提供注册域名服务的有阿里云，腾讯云等。



## 二、环境搭建

### 1、Windows平台安装Ruby

### （1）下载Ruby及DevKit

去[Ruby](https://rubyinstaller.org/downloads/)官网下载Windows平台的Installer，需要注意的是：

**不仅需要下载RubyInstallers，同时也需要下载下面的DEVELOPMENT KIT**

![下载示例](http://ov8clk5cn.bkt.clouddn.com/17-8-25/13047553.jpg)



### (2)安装Ruby及初始化DevKit

安装Ruby时，会自动将Ruby添加到环境变量中，建议添加，不要修改默认设置。

安装完Ruby之后，打开下载好的DevKit，将其解压，如解压到`C:\RubyDevKit\`，之后在命令行输入`cd C:\RubyDevKit `进入到该目录，执行如下操作安装DevKit。

~~~
ruby dk.rb init
ruby dk.rb install
~~~



### 2、安装Jekyll

检查 gem 是否安装成功

Win + R，输入 CMD 打开命令行工具，输入`gem -v`，检查安装 Ruby 时， gem 是否安装正常。若执行该命令之后，显示了 gem 的版本号，则安装 gem 成功，继续安装Jekyll

安装Jeykll

则执行 `gem install jekyll`  安装jekyll。若速度较慢，可将镜像更换为[Ruby China镜像](https://gems.ruby-china.org/)。更换命令为:

`gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/`

之后，输入`gem sources -l` ，若显示 `https://gems.ruby-china.org` ，则更换成功。



## 三、利用 Jekyll 创建 Blog

 Win + R打开命令行，进入存放 Blog 文件的目录，然后输入如下命令

```
gem install jekyll bundler
jekyll new my-blog
cd my-blog
bundle install
bundle exec jekyll serve
```

之后打开浏览器，输入 `http://localhost:4000`，进入个人的 Blog 界面。

这种方式创建的 Blog 比较原始，需要自己一点一点地修改美化，若想快速建成个人 Blog ，可进入 [Jekyll主题网站](http://jekyllthemes.org/) 或者 GitHub 选择别人做好的主题。

## 四、使用 Jekyll 主题

**注意：**本网站使用的主题是由 [HyG](https://github.com/Gaohaoyang/gaohaoyang.github.io) 设计的，接下来的讲解都是围绕该主题展开。若不想使用本主题，可忽略下面的内容

### 1、下载主题

在 GitHub 上将主题下载到本地，然后解压到存放 Blog 文件的目录下。

### 2、配置主题文件为个人主题

找到配置文件 `_config.yml` ，修改其中的参数。

（1）修改网站头部信息

```
title: ZekeTian # 博客的标题
brief-intro: Computer Science Student # 博客的简短说明
baseurl: "" # the subpath of your site, e.g. /blog
url: "http://www.zeketian.top/" # 博客网址
```



（2）修改底部链接信息

```
#twitter_username: username
#facebook_username: username.water
github_username:  ZekeTian
email: zeke.tian@outlook.com
#weibo_username: username
zhihu_username: zeketian
#linkedIn_username: username
dribbble_username:

description_footer: 记录我技能成长历程! # 底部说明文字
```

:前面的是各个社交网站名，: 后面各个社交网站自己的用户名（注意大小写，有的网站不需要区分大小写）

（3）修改评论链接

```
duoshuo_shortname: # 多说评论
disqus_shortname: www-zeketian-top # disqus 评论
```

两种评论方式选择一种并注册用户，需要注意的是 **disqus_shortname不一定是自己注册的用户名，可能是系统分配的**。

查看 disqus 的shortname 的方式，登录 disqus ，点击右上角的 “admin”， 然后进入管理界面， 按照下面的步骤即可查看自己的 shortname

![查看 disqus shortname](http://ov8clk5cn.bkt.clouddn.com/17-8-25/67431635.jpg)

（4）修改统计信息

```
baidu_tongji_id:  # 百度统计的 id 号
google_analytics_id:  # google 统计的 id
```

如果要使用[百度统计](https://tongji.baidu.com/web/welcome/login)与 [google 统计](https://www.google.com/intl/zh-CN/analytics/)需要注册两个网站的用户，下面以获取百度统计 id 号为例。

* 按照提示注册成为百度统计用户
* 进入个人网站，点击 “管理” ==》 “获取代码” ![](http://ov8clk5cn.bkt.clouddn.com/17-8-25/77349227.jpg)
* 在统计代码获取中找到` hm.src = "https://hm.baidu.com/hm.js?XXXXXXXXX"` ，复制其中的XXXXXXXXX并粘贴到 baidu_tongji_id : 处

### 3、清除原作者的相关信息

（1）替换网站图标

访问 [Icon 在线制作网站](http://cn.faviconico.org/) 制作自己的 Icon ，保存到本地替换掉原来的 `favicon.ico` 文件

（2）删除 `_posts` 目录下的原作者的博客文件

（3）修改 Collections 和 About 信息

找到 `page` 目录下的 `3collections.md` 和 `4about.md` 文件，修改其中的内容

（4）删除或修改 Demo 界面的信息

原 Demo 界面展示是作者的项目，若不想要此界面， 可将 `page` 目录里面的 `3demo.html` 文件删除。如果要修改里面的内容，则要到 `js` 目录下修改 js 文件

### 4、开始写博客

在 `_posts` 目录下新建 `.md` 文件，遵循 Markdown 语法写个人博客即可（文件名建议以时间的形式命名，即 yyyy-mm-dd-filename 形式，这样会自动生成时间）。

**注意:** 因为利用 Markdown 来写博客，故博客中含有图片时较麻烦，需要选择图床，本博客选择的图床是七牛云和极简图床 Chrome 插件 

**博客语法注意事项:**

（1）在博客头部添加如下信息，可以方便将文章分类、加上标签

```
---
layout: post
title:  "文章标题"
date:   2017-08-25 12:00:00 +0800
categories: 目录
tags: 标签
author: 作者名
mathjax: true
---
```

若文件名是以时间的形式命名，则 `date` 可以不写， `mathjax` 是用来显示数学公式， 可选。 `layout` 、`categories` 等信息建议填写。

（2）生成目录 

```
* content
{:toc}
```

添加上面两行代码可以自动生成目录	

（3）添加摘要

摘要是用来在 Home 界面上显示，若想添加摘要，则在摘要与正文之间空四行，即可让空行之前的内容成为摘要。

摘要分割符的设置可在 `_config.yml` 中修改

```
# excerpt
excerpt_separator: "\n\n\n\n"
```



详细配置信息可参考 [HyG](https://github.com/Gaohaoyang/gaohaoyang.github.io) 的博客

### 5、运行

打开命令行工具，路径切换到 Blog 目录下， 运行 `jekyll s` ，若正常运行则会看到 `Server running... press ctrl-c to stop.`。

在浏览器中输入 `http://localhost:4000/` 即可在本地访问，若访问正常，即可利用 Git 将代码部署到 GitHub 上，然后输入网址访问即可。

