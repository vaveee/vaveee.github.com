---
layout: post
title: "在 Ubuntu Server 14.04 LTS 版上安装 Odoo 8"
categories:
- 
tags:
- 


---


<< [英文原文](http://www.theopensourcerer.com/2014/09/how-to-install-openerpodoo-8-on-ubuntu-server-14-04-lts/)

<< [中文原文](http://shine-it.net/index.php?topic=16623.msg29044#msg29044 )


2014 年 9 月19 日 Odoo 官方发布了Odoo 8.0 正式版本，这是一个全新的版本，整合了网站建设等多种功能。
在 Linux 类操作系统中安装 odoo 的常规办法是从 odoo 官方网站的下载库http://nightly.openerp.com 相应版本的目录里面下载一个.deb  安装包 ( 用于 Debian/Ubuntu 类型的linux 系统) 或者一个. rpm安装包 (Redhat/CentOS 类型的linux 系统)，并且运行安装。但这种做法下，安装的配置是按照 odoo 官方的默认配置进行的，个性
化设置空间小，有时候不方便管理。

因此我们采取更手工化的源码安装方式，自己配置安装所需的配置。本文介绍在 Ubuntu Server 14.04 版操作系统中通过 Github 以源代码模式安装odoo 的方法。

这种做法的优点是：当官方的源代码升级时，或者有新的 bug 修正时，如果需要，我们可以在网络连通的情况下仅仅用一个“git pull”命令就能更新本地的代码。请注意，未来在进行 pull 代码的操作时，需要事先做好备份。有些情况下，odoo的数据库也需要更新。

第一步，建立服务器安装Ubuntu Server 14.04
================
 

访问 ubuntu 官方网站服务器版页面http://www.ubuntu.com/server下载安装镜像，服务器版目前已经没有 32 位版本，只有 64 位版。或者从国内的网易源下载镜像:
http://mirrors.163.com/ubuntu-releases/14.04/ubuntu-14.04-server-amd64.iso下载完成后将镜像制成启动光盘或者优盘，按照常规的方法将它安装在电脑上，

记好自己设置的用户名和密码。
本文以设定主机名为 odoo、用户名为 dean 为例进行 Ubuntu 系统的安装，下面各步骤中如果有的命令与主机名和用户名相关，请读者自行改成自己实际的主机名和用户名。
在安装进行到选择预安装的服务的步骤时，选中 postgreSQL，把 odoo 所需的数据服务环境 postgreSQL 数据库一并装好。
系统装好、重启后用自己用户名和密码登录进去，运行

	psql –version
	
命令查看版本，目前随 Ubuntu 14.04 服务器版安装的是 postgresQL 的 9.3.5 版本

更新服务器的软件源信息

	sudo apt-get update
	
并且更新服务器的各个软件包，自动查找依赖关系

	sudo apt-get dist-upgrade

尽管并不总是必须的，但此时最好重启下服务器，以使改变的内容更新。

	sudo shutdown -r now

第二步，创建一个系统用户 odoo，将来让他拥有 odoo 程序的权限并运行它
==================

运行

	sudo adduser --system --home=/opt/odoo --group odoo

这里添加的 odoo 用户是一个系统用户，它是用来拥有并运行各种后台服务的一类用户，而不是用来登录进系统进行各种操作的个人用户。在上述命令中指定并创建了odoo 用户的“home”——家目录为 /opt/odoo，这里
就是我们将要把 odoo 程序代码存放的位置。你可以自由选择代码存放的位置，但请注意下文中的一些配置文件里面的内容是基于上述命令指定的目录而定的，所以当读者自行选择代码存放位置时，下文中的某些设置要自行修改。虽然系统用户被禁止用于登录，并且没有 shell，但是当我们需要以它的身份进行一些特定操作的时候，还是可以用 su 命令切换用户：

	sudo su - odoo -s /bin/bash
这个 su（Shift User）命令将把你目前的终端登录切换到 odoo 用户，并且使用 /bin/bash 这个 shell。这命令运行后会自动把你当前所在的目录切换到 odoo 用户的家目录 /opt/odoo。当 你 以 o d o o 用 户 身 份 操 作 完 毕 后 ，可 以 用

	exit
命令离开 odoo 用户的 shell，回到你登录所用的用户

 

第三步，安装和配置数据库服务器 PostgreSQL
==================

如果你之前在安装 ubuntu 服务器过程中没勾选一并安装 postgreSQL，那么可以现在安装：

	sudo apt-get install postgresql
如果已经安装过了则不必执行这个命令。
下面为 postgreSQL 数据库添加并配置 odoo 用户 :
首先切换到 postgres 用户，它是 postgreSQL 默认的初始用户，以它的身份操作我们就有配置数据库的权限：

	sudo su - postgres
然后以 postgres 的身份创建一个新的数据库用户 odoo，odoo 程序将以它的身份访问 postgreSQL 数据库，来创建和删除数据库文件。（下边的命令第一行末尾的 - 和第二行开头的 s 之间没有空格 )

	createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo
	createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo
系统提示两次输入密码：
	
	Enter password for new role: ********
	Enter it again: ********
记住你这里设置的密码，后文中你会用到它
最后退出 postgres 用户身份：

	exit

第四步，为 Ubuntu 服务器安装 Python 运行库和 wkhtmltopdf
==================

Odoo 8.0 版本依赖的 python 运行库与 OpenERP 7.0 版本所依赖的有些不同。

	sudo apt-get install python-dateutil python-docutils python-feedparser python-gdata \
	python-jinja2 python-ldap python-libxslt1 python-lxml python-mako python-mock python-openid \
	python-psycopg2 python-psutil python-pybabel python-pychart python-pydot python-pyparsing \
	python-reportlab python-simplejson python-tz python-unittest2 python-vatnumber python-vobject \
	python-webdav python-werkzeug python-xlwt python-yaml python-zsi python-pyPdf \
	python-decorator python-passlib python-requests

 
Odoo 8.0 版改用 wkhtmltopdf 来输出 pdf，因此要下载 wkhtmltopdf 并安装。

先下载：（下边的命令第一行末尾的 / 和第二行开头的 w 之间没有空格 )

64位版本：http://download.gna.org/wkhtmltopdf/0.12/0.12.0/
上传 后  解压缩下载到的文件：

	tar -vxf wkhtmltox-linux-amd64_0.12.0-03c001d.tar.xz

得到一个目录wkhtmltox，把wkhtmltopdf复制到/usr/bin目录，更改所有者，并增加可执行属性
 

	sudo cp wkhtmltox/bin/wkhtmltopdf /usr/bin/
	sudo chown root:root /usr/bin/wkhtmltopdf
	sudo chmod +x /usr/bin/wkhtmltopdf

测 试 一下，打 印 一 个 网 页 到 你 自 己 的 家 目 录 ：

	wkhtmltopdf www.baidu.com ~/baidu.pdf

如果显示成功输出了pdf 那么 wkhtmltopdf 就告安装完成。

安装中文字体：
	
	sudo apt-get install ttf-wqy-zenhei
 	sudo apt-get install ttf-wqy-microhei

这里安装完了之后，所有 Odoo 8.0 运行时依赖的项目都已安装完成。

第五步，安装 Odoo 服务器
==================
先安装 git 软件

	sudo apt-get install git

切换到 odoo 用户 :

	sudo su - odoo -s /bin/bash

用 git 软件从 github 网站的 odoo 8.0 分支下载一套代码 (下边的命令第一行末尾
的 - 和第二行开头的 b 之间没有空格；命令最后一个单词后面的一空格加一个点“.”
，这个点表示“当前目录”，不是个句号 ，提示目录存在的话，可以 先用rm -r删除掉 已存在的目录，注意rm -r 是个危险的命令):

	git clone https://www.github.com/odoo/odoo --branch 8.0 --single-branch .

或者 用这个：

	git clone -b 8.0 https://github.com/odoo/odoo.git .

( 有一百多兆东西要下载，根据你的网速，这里会花上些时间

下载完整的分支到当前odoo目录：

	git clone https://github.com/odoo/odoo.git .

全下载好了之后，退出odoo用户:

	exit

第六步，配置 Odoo 程序
==================

odoo 默认的配置文件 (/opt/odoo/debian/openerp-server.conf) 包括基础的设置
内容，做一点小小改动就可以在我们的系统上很好地运行，这里我们先把这个文件复
制到我们需要的位置 /etc下：

	sudo cp /opt/odoo/debian/openerp-server.conf /etc/odoo-server.conf

更改它的所有权和许可：

	sudo chown odoo: /etc/odoo-server.conf
	sudo chmod 640 /etc/odoo-server.conf

上述命令让这个文件被 odoo 用户和用户组拥有，并且只有 odoo 用户和 root 用
户可以读取。下面用文本编辑器编辑它，初学者建议使用 Ubuntu 自带的 nano 编辑器 , 以它为例 ，运 行 ：

	sudo nano /etc/odoo-server.conf
然后做 3 处改动，

1. 打开这个配置文件后，在文件顶部，找到

	db_password = False

这一行，把等号后面的 False 改为你第三步配置 postgreSQL 时设定的数据库密码。
2. 然后找到
	
	addons_path = /usr/lib/python2.7/dist-packages/openerp/addons

这 一 行，改 成

	addons_path = /opt/odoo/addons

这样 odoo 程序会到我们个性化安装的 opt/odoo/addons 目录里面去读取模块。

3. 我们还要指定 Odoo 往哪里写它的日志文件。在文件的末尾新加一行

	logfile=/var/log/odoo/odoo-server.log

配置文件编辑好后，按 Ctrl+O，然后回车覆盖保存，然后 Ctrl+X 退出 nano 程序。
现在你可以试着启动 odoo 服务器，看它是否正常运行。
先切换到 odoo 用户，

	sudo su – odoo -s /bin/bash

然后运行

	/opt/odoo/openerp-server

如果你得到的界面反馈是几行字，告诉你 “OpenERP is running and waiting for connections.”那么就 OK 了。( 虽然版本升级了，但是在日志里面仍然把这程序叫OpenERP 而不是 Odoo)
如果有错误出现，你就要回头找找看看问题出在哪。一切正常的话，按 Ctrl+C 来停止服务器，然后用

	exit
命令离开odoo用户，回到你自己登陆的shell。

第七步，安装启动脚本
==================

启动、停止 odoo 服务牵扯到许多模块，需要多个步骤的操作，比较繁琐，下边我们安装一个脚本，它将以批处理的方式处理这些步骤，我们只要运行这个脚本一次，它就能以正确的用户身份批处理地运行 odoo 服务器的启动和停止等动作。
odoo 程序提供了一个现成的脚本，是 /opt/odoo/server/install/openerp-server.init这个文件，但需要一点小改动——因为我们不是按 odoo 的默认安装方式装的。
这里有个修改好的脚本文件，可以下载使用：(下边的命令第一行末尾的 / 和第二行开头的 o 之间没有空格 )
wget http://www.theopensourcerer.com/wp-content/uploads/2014/09/odoo-server与配置文件类似，你得把下载到的这个脚本复制到 /etc/init.d/ 并将其命名为odoo-server:

	sudo cp ~/odoo-server /etc/init.d/odoo-server
然后把它改成可执行文件，由 root 用户拥有：

	sudo chmod 755 /etc/init.d/odoo-server
	sudo chown root: /etc/init.d/odoo-server
在第六步我们编辑的配置文件里面指定了odoo 服务器的日志文件存储位置，现
在我们得创建那个目录，这样 odoo 服务器就能往里面写日志了，同时我们还得让这
个位置能够被 odoo 用户读写：

	sudo mkdir /var/log/odoo
	sudo chown odoo:root /var/log/odoo

第八步，测试服务器
==================

要启动Odoo服务器，输入:

	sudo /etc/init.d/odoo-server start
现在你可以查看日志文件，看服务器是否已经启动

	less /var/log/odoo/odoo-server.log

（要退出 less 命令的查看界面，只需按一下 q 键）。
如果启动服务器过程中出现问题，你可以依据日志文件的内容回头查找原因。
如果一切正常，现在就可以用网络浏览器访问 odoo 的 web 页面，地址的格式为：
http://odoo 服务器的 IP 或者域名 :8069
例如 odoo 服务器的 IP 是 192.168.1.10，那么在同一局域网的其它电脑上，打开网
络浏览器（由于 odoo 使用的是较新的 HTML5 标准，所以在 Windows XP 自带的 IE6
上无法正常显示，建议下载安装个新版的 Chrome 或者 Firefox 浏览器），在地址栏输入：
http://192.168.1.10:8069
就应该能看到 odoo 的数据库管理界面，因为是全新安装，一个帐套都没建立过，
所以默认会来到这个界面。
建议读者此时修改 odoo 系统的主密码 master password 并牢牢记住它，这个密码
是用来创建、复制、删除、备份、恢复数据库的，权力很大，最好设个强的密码。默认
的主密码是“admin”，比较不安全。该密码以明文方式写在 /etc/odoo-server.conf 文件
里面。这也是我们为什么把这个文件设成只有 odoo 用户和 root 用户可以读的原因。
当你在 web 界面上修改并且保存了新的主密码。/etc/odoo-server.conf 这个文件
会 被 覆 盖 写 入 ，并 且 会 多 出 一 些 选 项 。
下边检查 odoo 服务器是否可以被恰当地停止：

	sudo /etc/init.d/odoo-server stop
检查下日志文件，确定下服务已经停止，也可以用 top 命令查看 ubuntu 服务器正
在运行的进程表来确认。（退出 top 命令的查看界面也是按 Q 键）

第九步，自动化 Odoo 的启动和关闭
==================

前面的步骤如果都运行正常的话，最后的步骤就是让启动脚本随着 Ubuntu 服务
器 的 开 、关 机 而 自 动 启 动 、关 闭 O d o o 服 务 。

	sudo update-rc.d odoo-server defaults
如果你喜欢的话，你现在就可以重启动你的服务器，当你再登录进来的时候，
Odoo 应该已经在运行了。如果你输入

	ps aux | grep odoo
你将会看到像下面这样的信息：
	
	odoo 1491 0.1 10.6 207132 53596 ? Sl 22:23 0:02 python /opt/odoo/openerp-server -c /
etc/odoo-server.conf

这显示服务器正常运行，当然你也可以检查日志文件或者用网络浏览器访问的方
式来验证。


第九步，其他 
==================


connect to PostgreSQL server: FATAL: no pg_hba.conf entry for host

Add or edit the following line in your postgresql.conf :
/etc/postgresql/9.3/main/postgresql.conf

	listen_addresses = '*'

Add the following line as the first line of pg_hba.conf. It allows access to all databases for all users with an encrypted password:
/etc/postgresql/9.3/main/pg_hba.conf

	# TYPE DATABASE USER CIDR-ADDRESS  METHOD
	host  all  all 0.0.0.0/0 md5
