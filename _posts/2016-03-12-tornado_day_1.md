---
layout: post
title: "”tornado_day_1”"
categories:
- 
tags:
- 


---

tornado 笔记 第一天

运行环境 mac 

之前的开发版本是python2.7.6

准备用最新的python版本来使用tornado

Python多版本共存之pyenv 

	http://seisman.info/python-pyenv.html
	https://chloechen727.wordpress.com/2015/07/26/mac-os-x-plus-python-2-7-and-3-4/

创建tornado运行环境 

	virtualenv -p python3 tor_pve

激活运行环境

	source ~/tor_pve/bin/activate

在线安装tornado

	pip install tornado

hello world 代码

	import tornado.ioloop
	import tornado.web

	class MainHandler(tornado.web.RequestHandler):
	    def get(self):
	        self.write("Hello, world")

	def make_app():
	    return tornado.web.Application([
	        (r"/", MainHandler),
	    ])

	if __name__ == "__main__":
	    app = make_app()
	    app.listen(5000)
	    tornado.ioloop.IOLoop.current().start()

启动
	
	python hello.py 

访问
	
	127.0.0.1:5000


