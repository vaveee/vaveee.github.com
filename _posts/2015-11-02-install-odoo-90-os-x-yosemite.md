---
layout: post
title: "2015-11-02-install-odoo-90-os-x-yosemite"
categories: 开发环境
tags: odoo osx
---

Install Odoo 9.0 OS X Yosemite

安装 brew 支持
	
	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

安装 python postgresql

	brew update
	brew install python
	brew install postgresql
	/usr/local/Cellar/postgresql/9.4.4: 3014 files, 40M

设成开机启动 PostgreSQL

	ln -sfv /usr/local/opt/postgresql/*.plist ~/Library/LaunchAgents
	launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist

创建一个 PostgreSQL 用户

	createuser username -P
	#Enter password for new role:
	#Enter it again:

创建数据库

	createdb dbname -O username -E UTF8 -e

安装nodejs、node-less 这一步不能忘掉，否则将来登录Odoo帐套时，界面中会有报错信息：Could not execute command lessc

	brew install node
	npm install -g less less-plugin-clean-css
	npm install node-sass


安装相关包

	brew install freetype jpeg libpng libtiff webp xz

新建python环境

	pip install virtualenv

	cd ~/Sites
	virtualenv odoo-env

安装python 依赖包

	cd Sites/odoo
	~/Sites/odoo-env/bin/pip install -r requirements.txt
	~/Sites/odoo-env/bin/pip install flanker
	~/Sites/odoo-env/bin/pip install ofxparse

配置文件

	cp debian openerp-server.conf ../openerp-server.conf
	[options]
	; This is the password that allows database operations:
	dmin_passwd = admin
	db_host = localhost
	db_port = 5432
	db_user = odoo9
	db_password = odoo9
	addons_path = /Users/Sites/odoo/addons

启动
	
	python openerp-server -c openerp-server.conf


参考：

获取 Homebrew  [http://brew.sh/index_zh-cn.html](http://brew.sh/index_zh-cn.html)

postgres安装问题 [http://stackoverflow.com/questions/27700596/homebrew-postgres-broken](http://stackoverflow.com/questions/27700596/homebrew-postgres-broken)

[http://hiroz.cn/homebrew-install-versions/](http://hiroz.cn/homebrew-install-versions/)

postgresql教程 [http://dhq.me/mac-postgresql-install-usage](http://dhq.me/mac-postgresql-install-usage)

ubuntu安装odoo [http://vaveee.github.io/ubuntu-server-1404-install-odoo-90/](http://vaveee.github.io/ubuntu-server-1404-install-odoo-90/)


