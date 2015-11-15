---
layout: post
title: "2015-11-04-install-simple-git-server-in-ubuntu"
categories: 开发环境
tags: ubuntu git 
---


git server 搭建 (用Ubuntu linux)

1. 安裝git

	sudo apt-get install git-core

2. 在server端建立放置所有projects的目录
放在 /var/git

	cd /var
	sudo mkdir git

3. 在server端建立project
步驟3,5,6是 每开一个新project就要做一遍 的，假设现在我开了一个叫odoo

	cd /var/git
	sudo mkdir odoo.git

如果是第二次新增project，记得也要改改资料夹的组跟权限，相关步奏参照step 4
	
	cd odoo.git
	sudo git --bare init

4. 建立git的组
新建一个git组，用于git操作

	sudo groupadd git
	sudo usermod -a -G git laval # laval改成你自己的账号。

下面这两行，在每次新增新project时要对新的文件夹重做

	sudo chgrp -R git /var/git
	sudo chmod g+rwx -R /var/git
	
在local端，假设我的project文件夹是/path/to/your/projects/odoo

5. 初始化git
	
	cd /path/to/your/projects/odoo
	git init

6. 把我們遠端的repository加到git remote
	
	git remote add odoo43 git@ip:/var/git/odoo9.git/

查看现在的remote
	
	git remote -v

7. 把本地的git push 到 制定的remote

	git push odoo43 

参考：

[http://toyroom.bruceli.net/tw/2011/02/04/install-git-server-on-ubuntu-linux.html](http://toyroom.bruceli.net/tw/2011/02/04/install-git-server-on-ubuntu-linux.html)

[https://git-scm.com/book/zh/v1/Git-%E5%9F%BA%E7%A1%80-%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93%E7%9A%84%E4%BD%BF%E7%94%A8](https://git-scm.com/book/zh/v1/Git-%E5%9F%BA%E7%A1%80-%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93%E7%9A%84%E4%BD%BF%E7%94%A8)

[https://git-scm.com/book/zh/v1/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF](https://git-scm.com/book/zh/v1/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF)

[http://www.tfan.org/build-a-simple-ssh-git-service-in-five-minutes/](http://www.tfan.org/build-a-simple-ssh-git-service-in-five-minutes/)