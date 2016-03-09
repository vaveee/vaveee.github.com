---
layout: post
title: "Ubuntu Server 14.04 install Odoo 9.0"
categories: 开发环境
tags: ubuntu odoo9 odoo
---

http://blog.csdn.net/wangnan537/article/details/48895897

Odoo 9.0版已于2015年10月1日正式发布，相较Odoo 8.0版而言，新版本重写了会计模块，新增了一些功能，改进了用户体验(详见Odoo 9.0特性、Odoo 9.0发行说明)。Ubuntu安装镜像可在其官网页面下载，本文假定你已准备好Ubuntu Server 14.04，主要介绍如何以源码方式安装Odoo 9.0。

本文基于社区文章《在Ubuntu Server 14.04 LTS版上以git方式源码安装Odoo8.0》(@Alan Lord 原创、@郑州-Dean 翻译、@上海-卓忆 补充)整理简化而来。与Odoo8.0的安装相比，主要差别在于多了nodejs、node-less的安装，其他基本相同。

整理：苏州-微尘

1. 更新Ubuntu服务器软件源
 
	sudo apt-get update  #更新软件源  
	sudo apt-get dist-upgrade  #更新软件包，自动查找依赖关系  
	sudo shutdown -r now  #重启服务器，以更新改变的内容  

2. 新建系统用户用于运行Odoo程序
运行如下命令创建系统用户：

	sudo adduser --system --home=/opt/odoo9 --group odoo9  
	#新建系统用户odoo9，指定home目录为/opt/odoo9  

系统用户不能用于登录并且没有shell，但当需要以它的身份进行特定操作时，可以用su命令切换用户：

	sudo su - odoo9 -s /bin/bash  # 将当前终端登录切换到odoo9用户，并使用/bin/bash这个shell  

命令运行后会自动从当前目录切换到odoo9用户的home目录/opt/odoo9。操作完毕后输入exit命令，离开odoo9用户的shell，回到登录所用的用户。
3. 安装和配置数据库服务器PostgreSQL
先运行如下命令查看PostgreSQL数据库的版本：

	psql –version  #查看PostgreSQL版本  

如报错，则表明之前未安装过PostgreSQL，那么可以通过如下命令安装：

	sudo apt-get install postgresql #安装PostgreSQL  

接下来切换到postgres用户，它是PostgreSQL默认的初始用户，以它的身份操作我们就有配置数据库的权限：

	sudo su - postgres  

然后以postgres的身份创建一个新的数据库用户odoo9，Odoo程序将用该用户访问数据库。

	createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo9  

根据系统输入密码，记住你这里设置的密码。最后运行exit退出postgres用户。
4. 安装Python运行库和wkhtmltopdf
运行如下命令安装Odoo 9.0版本依赖的python运行库：

	sudo apt-get install python-dateutil python-docutils python-feedparser python-gdata \  
	python-jinja2 python-ldap python-libxslt1 python-lxml python-mako python-mock python-openid \  
	python-psycopg2 python-psutil python-pybabel python-pychart python-pydot python-pyparsing \  
	python-reportlab python-simplejson python-tz python-unittest2 python-vatnumber python-vobject \  
	python-webdav python-werkzeug python-xlwt python-yaml python-zsi python-pyPdf \  
	python-decorator python-passlib python-requests  

下载安装wkhtmltopdf(Odoo使用wkhtmltopdf来输出pdf):

	sudo wget http://download.gna.org/wkhtmltopdf/0.12/0.12.1/wkhtmltox-0.12.1_linux-trusty-amd64.deb #下载wkhtmltopdf，注意根据操作系统选择相应版本  
	sudo dpkg -i wkhtmltox-0.12.1_linux-trusty-amd64.deb  #安装wkhtmltopdf  
	sudo cp /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf  #安装完成后将可执行文件复制到usr/bin中  
	sudo chown root:root /usr/bin/wkhtmltopdf  #更改所有者为root用户  
	sudo chmod +x /usr/bin/wkhtmltopdf  #并增加可执行属性  
	wkhtmltopdf www.baidu.com ~/baidu.pdf  #打印一个网页到home目录，如果成功生成pdf则表明安装成功  
	sudo apt-get install ttf-wqy-zenhei  #安装中文字体  
	sudo apt-get install ttf-wqy-microhei  #安装中文字体  

5. 安装Odoo服务器代码

	sudo apt-get install git  #安装git软件  
	sudo su - odoo9 -s /bin/bash #切换到odoo9用户  
	git clone -b 9.0 https://github.com/odoo/odoo.git .  #下载Odoo9.0代码  
	exit #退出odoo9用户  

6. 安装nodejs、node-less
这一步不能忘掉，否则将来登录Odoo帐套时，界面中会有报错信息：Could not execute command lessc

	apt-get install -y npm   
	sudo ln -s /usr/bin/nodejs /usr/bin/node  
	npm install -g less less-plugin-clean-css  
	apt-get install node-less  

7. 配置Odoo程序
默认的配置文件openerp-server.conf包括基本的设置，这里需要做一点改动。

	sudo cp /opt/odoo9/debian/openerp-server.conf /etc/odoo9-server.conf  #把文件复制到/etc目录  
	sudo chown odoo9: /etc/odoo9-server.conf #将所有权赋予odoo用户和用户组  
	sudo chmod 640 /etc/odoo9-server.conf #只允许odoo用户和root用户读取  

下面用Ubuntu自带的nano编辑器编辑它, 运行如下命令打开配置文件:

	sudo nano /etc/odoo-server.conf    

然后改动如下，


配置文件设置

	[options]
	; This is the password that allows database operations:
	; admin_passwd = admin
	db_host = False
	db_port = False
	db_user = odoo9
	db_password = False
	addons_path = /opt/odoo9/addons


配置文件编辑好后，按Ctrl+O，然后回车覆盖保存，然后Ctrl+X退出nano程序。
配置文件里指定了日志文件的存储位置，因此要创建这个目录，同时还得让它能被odoo9用户读写：

	sudo mkdir /var/log/odoo9  
	sudo chown odoo9:root /var/log/odoo9  

现在可以尝试启动Odoo服务器：

	sudo su - odoo9 -s /bin/bash  #先切换到odoo9用户，  
	./openerp-server -c /etc/odoo9-server.conf #运行Odoo  

在浏览器输入http://ip地址:8069/，因为是全新安装，还未创建过帐套，所以默认会进入数据库管理界面。



如果一切正常，按 Ctrl+C停止服务器，然后用exit命令离开odoo9用户，回到你自己登陆的shell。如果报错，则需要查看odoo-server.log排查错误。(为方便起见，可以先将配置文件中的logfile一行注释掉，这样就可以直接在控制台看到报错信息)

8. 安装启动脚本

启动、停止Odoo服务需要多个步骤的操作，比较繁琐，可以安装启动脚本以批处理的方式处理这些步骤。

复制配置文件 	sudo cp /opt/odoo/debian/openerp-server.conf /etc/odoo-server.conf

修改一下 

	[options]
	; This is the password that allows database operations:
	; admin_passwd = admin
	db_host = False
	db_port = False
	db_user = odoo9
	db_password = False
	#addons_path = /usr/lib/python2.7/dist-packages/openerp/addons
	addons_path = /opt/odoo9/odoo9/addons
	logfile = /var/log/odoo/odoo-server.log
	xmlrpc_port = 8069

启动脚本 /etc/init.d/odoo-server

	#!/bin/sh
	### BEGIN INIT INFO
	# Provides: odoo-server
	# Required-Start: $remote_fs $syslog
	# Required-Stop: $remote_fs $syslog
	# Should-Start: $network
	# Should-Stop: $network
	# Default-Start: 2 3 4 5
	# Default-Stop: 0 1 6
	# Short-Description: Odoo ERP
	# Description: Odoo is a complete ERP business solution.
	### END INIT INFO

	PATH=/bin:/sbin:/usr/bin
	# Change the Odoo source files location according your needs.
	DAEMON=/opt/odoo9/odoo9/openerp-server
	# Use the name convention of your choice 
	NAME=odoo-server
	DESC=odoo-server

	# Specify the user name (Default: odoo).
	USER=odoo9

	# Specify an alternate config file (Default: /etc/odoo-server.conf).
	CONFIGFILE="/etc/odoo-server.conf"

	# pidfile
	PIDFILE=/var/run/$NAME.pid

	# Additional options that are passed to the Daemon.
	DAEMON_OPTS="-c $CONFIGFILE"

	[ -x $DAEMON ] || exit 0
	[ -f $CONFIGFILE ] || exit 0

	checkpid() {
	[ -f $PIDFILE ] || return 1
	pid=`cat $PIDFILE`
	[ -d /proc/$pid ] && return 0
	return 1
	}

	case "${1}" in
	start)
	echo -n "Starting ${DESC}: "

	start-stop-daemon --start --quiet --pidfile ${PIDFILE} \
	--chuid ${USER} --background --make-pidfile \
	--exec ${DAEMON} -- ${DAEMON_OPTS}

	echo "${NAME}."
	;;

	stop)
	echo -n "Stopping ${DESC}: "

	start-stop-daemon --stop --quiet --pidfile ${PIDFILE} \
	--oknodo

	echo "${NAME}."
	;;

	restart|force-reload)
	echo -n "Restarting ${DESC}: "

	start-stop-daemon --stop --quiet --pidfile ${PIDFILE} \
	--oknodo

	sleep 1

	start-stop-daemon --start --quiet --pidfile ${PIDFILE} \
	--chuid ${USER} --background --make-pidfile \
	--exec ${DAEMON} -- ${DAEMON_OPTS}

	echo "${NAME}."
	;;

	*)
	N=/etc/init.d/${NAME}
	echo "Usage: ${NAME} {start|stop|restart|force-reload}" >&2
	exit 1
	;;
	esac

	exit 0


设定脚本权限

	sudo chmod 755 /etc/init.d/odoo-server
	sudo chown root: /etc/init.d/odoo-server


	sudo chown -R odoo: /opt/odoo/

	sudo chown odoo:root /var/log/odoo

	sudo chown odoo: /etc/odoo-server.conf
	sudo chmod 640 /etc/odoo-server.conf

启动

	sudo /etc/init.d/odoo-server start

查看日志

	tail -f /var/log/odoo/odoo-server.log

关闭

	sudo /etc/init.d/odoo-server stop

查看

	http://IP:8069



参考：

1. install-odoo-9-erp-on-ubuntu-14-04 [https://www.linode.com/docs/websites/cms/install-odoo-9-erp-on-ubuntu-14-04](https://www.linode.com/docs/websites/cms/install-odoo-9-erp-on-ubuntu-14-04)



