---
layout: post
title: "jekyll quick start"
categories:
- 
tags:
- 


---



用Jekyll写博客

快速通过GitHub Pages安装博客并发布。然后在本地运行你的博客并创建你的第一个Post和页面。

安装Jekyll-Bootsrap

安装Jekyll-Bootsrap，如果你还没安装好。Jekyll-Bootsrap是一个博客框架，拥有内建的主题、分析、评论以及Post/Page建立功能。

$ gem install jekyll
如果安装出了问题，请看看原始的Jekyll安装文档。你也可以通过GitHub Issues来创建一个支持issue。

一旦这个gem安装好，你就可以转到你的Jekyll-Bootsrap安装目录。如果你一直跟着首页的说明来做，此目录将是：USERNAME.github.com。进入此目录后，你就可以运行有服务器支持的jekyll：

cd USENAME.github.com
jekyll --server
将USENAME改为你自己的GitHub用户名
新建一篇博文（Post）

你可以轻易地通过一个rake任务来新建博文：

$ rake post title="Hello World"
此rake任务会自动创建一个有着正确的格式化好文件名的文件和一个YAML Front Matter。确保指定一个你自己的标题。缺省日期是当前日期。

写上日期才
$ rake post title="Hi, Jekyll" date="2012-03-04"
此rake任务不会覆盖任何已存在的博文，除非你让它那样做。

新建一个页面（Page）

新建页面也是通过rake任务，同样很容易：

$ rake page name="about.md"
新建一个嵌套页面：

$ rake page name="pages/about.md"
新建一个有着好看路径的页面：

$ rake page name="pages/about"
这会建立一个文件： ./pages/about/index.html
此rake任务会自动创建一个有着正确的格式化好文件名的文件和一个YAML Front Matter，同时包含Jekyll Bootstrap的设置文件。

Jekyll-Bootstrap页面样例

Jekyll-Bootstrap同样提供了许多预配置好页面样例提供用户参考。你可以学习它们的代码然后按照你需要的样子定制。

发布
git add .
git commit -m "jekyll quick start"
git push origin master

参照
http://jekyllbootstrap.com/usage/jekyll-quick-start.html
https://github.com/deercoder/0-tech-notes/blob/master/Reference/Jekyll%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.md

