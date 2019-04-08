---
layout: post
title:  怎样用github+jekyll搭建自己的博客网站
date:   2019-04-07 11:00:00 +0800
categories: document
#小类配置
tag: 网站相关
---

* content
{:toc}


### 准备
+ 一台电脑
+ 一个github账号
+ (可选)一个可以正常解析的域名

### 在github上部署自己的网页
登陆github，选择“New repository”来创建一个新的代码仓库。如果你想在GitHub上部署网页，仓库的名称必须为
`username.github.io`

好了，你现在已经有了一个可以用来储存网站的素材和html文件的代码仓库。你将要搭建的网站的入口是index.html, 一般来说你需要在仓库中创建这个文件，按照html格式书写代码并提交(commit)，之后在浏览器访问*username*.github.io即可看到你自己编写的网页了。
然而在github网站在线编辑代码并不是一个好的选择。所以你需要用git来管理你的代码，随时从本地上传代码，建立起本地代码与线上仓库之间的联系。关于git的使用网上有很多教程，我也可能会在之后的博客里总结。

### 用jekyll建立博客
如果你没有很高深的html、css、js语言的功底，写出来的网页总是不能让自己满意，恰巧你只是想要一个自己的博客，你可以跟我一样选择一个现有的博客模板。这里我选择的是jekyll，另一个比较有名的系统是Hexo，不过我不了解，这里就不谈了。可以把jekyll理解成一个网站生成器，按jekyll官网的说法，jekyll的特点就是简单（无需数据库、繁琐更新），静态（只需 Markdown、Liquid、HTML 、CSS）。这里得说明一件很遗憾的事，GitHub上部署的这种网站好像只支持静态网页，所以想要搭建discuz等论坛可能就不行了。下面如何介绍在本地安装jekyll。
+ 下载Ruby installer(这个软件的下载链接好像被墙了，但是网上应该有分流)
+ 下载RubyGems，链接：[RubyGems](https://rubygems.org/pages/download)
+ 解压Gems到任意目录，cmd下：
```
cd '解压路径'
ruby setup.rb
gem install jekyll
```

到这里jekyll已经安装完成了，下面介绍如何在本地创建并运行博客。cmd下cd到任意目录(cd 目录名)，博客文件将在这里创建（博客名字可自定义）：

```
jekyll new blog1
cd blog1
jekyll server
```
+ 在同一台电脑上的浏览器里输入网址http://127.0.0.1:4000/即可访问刚刚创建的博客。
![这时候你的页面看起来是这样的]({{ '/styles/images/testblog.png' | prepend: site.baseurl  }})

### 选择jekyll主题并配置
看得出来这时候的博客还只是一个简单的页面，选择主题可以让它看起来更舒适(比如我现在用的这个)。

在[jekyll主题官网](http://jekyllthemes.org/)选择你中意的主题并下载到本地，(也可以fork到你的github)，在cmd下cd到你下载的主题的根目录，在cmd中运行**jekyll server**，就可以在本地运行网站服务了。

不同主题的jekyll配置方法略有不同，主题的作者大多也会在说明文档里进行说明。下面列出一些可能需要手动配置的文件：
+ _config.yml: 基本的配置文件，详细内容见[这里](https://www.jekyll.com.cn/docs/configuration/)
+ _posts文件夹:将你编写的博客储存在这里，文件名和文件开头都有一定的格式要求，具体见主题作者给的文档。编写博客的语言是markdown。
+ includes文件夹:可能包含了一些主题作者的html文件或其他布局文件，修改它们可以细微的改动一些网页布局和内容。

记得最后将你的整个博客文件整个上传到github，远程域名访问就可以达到你在本地运行时相同的效果。

### 将自己的域名指向GitHub Pages
如果你自己有一个域名，并且这个域名可以正常解析，那么你可以将这个域名指向你在github创建的网页。比如只要在浏览器输入njuszj.com就可以访问我的博客啦。
+ 创建CNAME文件，CNAME是一种解析方式，将域名指向另一个网址。在username.github.io仓库首页Settings界面中，custom domain选项下填入你的域名，github会自动在仓库中创建CNAME文件。
![CNAME]({{ '/styles/images/CNAME.png' | prepend: site.baseurl  }})
+ 在你解析域名的添加解析，以腾讯云为例：
```
添加解析
主机记录：根据域名类型来选，比如我选的是www，解析后就是www.njuszj.com
记录类型：CNAME
线路类型：默认即可
记录值：username.github.io
```
![png]({{ '/styles/images/tengxunyun.png' | prepend: site.baseurl  }})

### 结语
至此，一个博客网站就搭好了，如果发现我写的这篇博客有任何问题，欢迎用邮箱与我交流：njuszj@qq.com。

### 参考资料
+ [如何在github上搭建网站？](https://www.cnblogs.com/camille666/p/how_to_build_website_at_github.html)

+ [Github+Jekyll 搭建个人网站详细教程](https://www.jianshu.com/p/9f71e260925d)






