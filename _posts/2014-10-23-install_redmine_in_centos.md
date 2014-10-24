---
layout: post
title: 'centos6.x 安装nginx+thin+redmine  '
categories: 文档
tags: linux  centos  thin  redmine  nginx
---

  1、先安装系统包：

    yum -y install patch make gcc gcc-c++ gcc-g77 flex* bison file
    yum -y install libtool libtool-libs autoconf kernel-devel
    yum -y install libjpeg libjpeg-devel libpng libpng-devel libpng10 libpng10-devel gd gd-devel
    yum -y install freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel
    yum -y install glib2 glib2-devel bzip2 bzip2-devel libevent libevent-devel
    yum -y install ncurses ncurses-devel curl curl-devel e2fsprogs
    yum -y install e2fsprogs-devel krb5 krb5-devel libidn libidn-devel
    yum -y install openssl openssl-devel vim-minimal nano sendmail
    yum -y install fonts-chinese gettext gettext-devel
    yum -y install ncurses-devel
    yum -y install gmp-devel pspell-devel
    yum -y install unzip
    yum -y install automake libmcrypt* libtool-ltdl-devel*
    yum -y install readline* libxslt* pcre* net-snmp* gmp* libtidy*
    yum -y install ImageMagick* svnversion*

由于redmine是用ruby on rails框架开发的，所以需要安装ruby+rails环境，但由于ruby版本和rails版本需要对应，并且redmine也与相应的rails版本对应，
所以，这里，先按装rvm。可以切换ruby的版本

  执行安装：

    bash < <( curl -L https://get.rvm.io )

  安装成功后，运行下面的命令：

    source /etc/profile
    /usr/local/rvm/bin/rvm reload

  将rvm路径加载到系统路径中，然后直接输入:

    rvm -v

如果出现版本信息，说明rvm已经安装成功

2. 安装ruby

  这里使用最新版本的ruby：

    rvm install ruby

  如果出现类似下面的错误信息：

    There is no checksum for 'https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-6' or 'RPM-GPG-KEY-EPEL-6', it's not possible to validate it.
    This could be because your RVM install's list of versions is out of date. You may want to
    update your list of rubies by running 'rvm get stable' and try again.
    If that does not resolve the issue and you wish to continue with unverified download
    add '--verify-downloads 1' after the command.

    There is no checksum for 'https://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm' or 'epel-release-6-8.noarch.rpm', it's not possible to validate it.
    This could be because your RVM install's list of versions is out of date. You may want to
    update your list of rubies by running 'rvm get stable' and try again.
    If that does not resolve the issue and you wish to continue with unverified download
    add '--verify-downloads 1' after the command.


  运行：

    rpm -ivh https://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm


  再运行：


    rvm install ruby



3. 安装rails 和 相关依赖包


  运行：

    gem install rails
    gem install rake
    gem install mysql2

  如果安装出错，可能是mysql是自己编译安装的，请使用以下命令重新安装，指定mysql_config的位置：

    gem install mysql2 -- --with-mysql-config=/data0/mysql/bin/mysql_config

  4.下载redmine:

    yum install subversion -y
    svn co http://svn.redmine.org/redmine/branches/2.3-stable redmine

  进入redmine命令, 安装依赖包, 运行如下命令：

    bundle install


接下来，修改 database.yml.example 为 database.yml
接下来，修改 configuration.yml.example 为 configuration.yml

    cd redmine/config
    mv database.yml.example database.yml

打开database.yul 设置数据库
修改以下信息，为你对应的信息

    production:
      adapter: mysql2
      database: redmine
      host: localhost
      username: root
      password: ""
      encoding: utf8

回到redmine下，运行以下命令导入数据

    rake config/initializers/secret_token.rb
    rake db:migrate RAILS_ENV="production"

http://110.76.45.201:3000/

注意配置的数据库，需要已经存在，不然会出错

5.运行，并测试

    script/rails server -e production


6.安装thin，使用thin运行

    gem install thin
    thin install

  配置运行

    mv /etc/rc.d/thin /etc/init.d/thin

  开机启动

    chkconfig --level 345 thin on

vi /etc/thinredmine.yml

    environment: production
    address: 0.0.0.0
    port: 3000
    timeout: 30
    log: log/thin.log
    pid: tmp/pids/thin.pid
    max_conns: 1024
    max_persistent_conns: 100
    require: []
    wait: 30
    servers: 1
    daemonize: true
    socket: /tmp/thin.sock

在redmine/Gemfile中的gem "rails"之前，添加如下内容

    gem 'thin'

切换到redmine的安装目录后运行

    /etc/init.d/thin start

访问

    http://ip-domain:3000/

7. nginx + thin
已知nginx已经安装成功
添加一个server段

      upstream thin_cluster {
        server unix:/tmp/thin.0.sock;
      }

      server {
        listen 80;
        server_name ip-domain;

        root /opt/redmine/public;#对应app path,务必要指向public 文件夹为止

        location / {
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header Host $http_host;
          proxy_redirect off;

          if (-f $request_filename/index.html) {
            rewrite (.*) $1/index.html break;
          }
          if (-f $request_filename.html) {
            rewrite (.*) $1.html break;
          }
          if (!-f $request_filename) {
            proxy_pass http://thin_cluster;
            break;
          }
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
          root html;
        }
      }

到这里已经大功告成了，看看日志

tail -f /opt/redmine/log/production.log

如果数据库有乱码，是因为乱码问题了，配置数据库编码 vi /etc/my.cnf ：

    [mysqld]
    default-character-set=utf8
    character_set_server=utf8
    init_connect='SET NAMES utf8'

    [mysql.server]
    default-character-set = utf8

    [client]
    default-character-set = utf8

重启数据库：

    service mysqld restart