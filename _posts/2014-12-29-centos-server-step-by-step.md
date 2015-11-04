---
layout: post
title: "centos 6 服务器配置"
categories:
- 
tags: centos
- 


---

yum update

yum install gcc gcc-c++ make 


yum install subversion 

yum install mysql-server  mysql-devel mysql

service mysqld start

chkconfig --level 3 sshd on

yum install java-1.7.0-openjdk

yum -y install giflib-devel libjpeg-devel freetype-devel t1lib-devel zlib

== pdf2swf==
http://www.andrehonsberg.com/article/install-swftools-on-centos-56

==user==
useradd username

passwd username

usermod -G wheel username

 /etc/sudoers

== ip ==

/etc/sysconfig/iptables

/sbin/service iptables restart

== python ==

centos版本默认是2.4，需要升级到2.7

下载python2.7安装包

./configure --prefix=/usr/local/Python2.7 --enable-shared

make

make install

使用新安装的python:

mv /usr/bin/python /usr/bin/python.bak

ln -s /usr/local/Python2.7/bin/python2.7 /usr/bin/python

运行测试python

报错：

	/usr/local/Python2.7/bin/python2.7: error while loading shared libraries: libpython2.7.so.1.0: cannot open shared object file: No such file or directory
	解决办法,添加包的位置：
	vi /etc/ld.so.conf 
	添加/usr/local/Python2.7/lib
	/sbin/ldconfig
	/sbin/ldconfig -v
	现在可以了 python进去交互环境
	因为centos的yum默认的是2.4的，所以我们要修改其引用脚本为2.4
	修改 /usr/bin/yumpython
	#!/usr/bin/python2.4

setuptools

pip 

==mysql python==
yum search MySQL-python


== mysql 安装==  
yum install mysql in cenos
recover MySQL database server password

== svn 安装==  
	yum install
	wget http://subversion.tigris.org/downloads/subversion-1.6.1.tar.gz
	./configure --prefix=/usr/local/svn/
	configure: error: no suitable apr found
	yum install apr-devel.i686  apr.i686 
	configure: error: no suitable APRUTIL found
	yum install apr-util.i686 apr-util-devel.i686 
	configure: error: Subversion requires SQLite
	yum install sqlite.i686  sqlite-devel.i686
	configure: error: subversion requires zlib
	yum install zlib-devel.i686  zlib.i686 

	make && make install
	echo "export PATH=$PATH:/usr/local/svn/bin/" >> /etc/profile
	[root@centos subversion-1.6.1]#
	[root@centos subversion-1.6.1]# source /etc/profile
	[root@centos subversion-1.6.1]# svn
	[root@centos subversion-1.6.1]# mkdir -p /opt/svn/
	[root@centos subversion-1.6.1]# mkdir -p /opt/svn/pinfla/
	[root@centos subversion-1.6.1]# svnadmin create /opt/svn/pinfla/
	[root@centos subversion-1.6.1]# vi /opt/svn/pinfla/conf/svnserve.conf
	[general]
	anon-access = none
	auth-access = write
	password-db = passwd
	authz-db = authz

	[root@centos subversion-1.6.1]# vi /opt/svn/pinfla/conf/authz
	[pinfla:/]
	lign = rw
	[root@centos subversion-1.6.1]# vi /opt/svn/pinfla/conf/passwd
	lign=passwd
	[root@centos subversion-1.6.1]# svnserve -d -r /opt/svn/
	[root@centos subversion-1.6.1]# netstat -tunlp|grep svn

==uname -a==
Linux centos 2.6.32-279.1.1.el6.i686 #1 SMP Tue Jul 10 12:30:45 UTC 2012 i686 i686 i386 GNU/Linux

== 配置hudson ==

== 集成部署环境 ==

== python 运行环境 ==
PIL http://pkgs.org/centos-6-rhel-6/atrpms-i386/PIL-1.1.6-10.el6.i686.rpm.html

==uwsgi+nginx==






