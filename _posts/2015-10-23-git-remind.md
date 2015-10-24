---
layout: post
title: "git  常用备忘"
categories:
- 
tags:
- 


---


Git基础配置
================

	ssh-keygen -t rsa -C "hello@163.com"    # 生成ssh keygen
	pbcopy < ~/.ssh/id_rsa.pub                      # 复制到剪贴板

	git config --global user.name "hello"
	git config --global user.email "hello@163.com"

	git config --global core.autocrlf input         # For Mac 换行符
	git config --global core.autocrlf input         # For Windows 换行符
	git config core.safecrlf false                  # 不再提示换行符

	git config http.postBuffer 524288000            # 提高push限制
	git config --list                               # 查看配置

Git编辑常用 
================

	git add  <file>
	git add .               # 将所有修改过的工作文件提交暂存区
	git add -A              # 将所有修改过、删除的、新增的的工作文件提交暂存区

	git checkout -- <file>  # 还没有git add之前，抛弃修改某文件
	git checkout .

	git reset <file>        # git add之后，将某文件从暂存区恢复到工作文件
	git reset -- .  
	git reset --hard        # 将修改过的文件从暂存区恢复到工作文件，本地不会有任何修改，不管是否git add
	git reset --hard HEAD^  # 撤销最近一次的提交记录，最近一次的commit记录将会从git log中删除


	git commit -m "comments"
	git commit -am "comments"   # git add, git rm和git commit等操作都合并在一起做
	git commit --amend          # 修改最后一次提交记录

	git revert <commit id>      # 恢复某次commit记录 恢复动作本身也创建了一次提交记录
	git revert HEAD             # 恢复最后一次提交的状态 恢复动作本身也创建了一次提交记录

Git分支管理
================

	git branch -a
	git branch <new branch>

	git checkout <branch>
	git checkout -b <new branch>             # 创建并切换到新分支
	git checkout -b <new_branch> <branchA>   # 切换到基于branchA创建的新分支

	git branch -d <branch>                   # 删除某个分支
	git branch -D <branch>                   # 强制删除某个分支，未被合并的分支被删除的时候需要强制

	git merge <branch>                       # 将branch分支合并到当前分支
	git rebase <branch>                      # 将branch分支在当前分支重新提交一遍

	    git checkout dev                     # 在dev分支
	    git rebase master                    # 把master分支的修改同步到dev分支 这一步一般会有冲突
	        git add .
	        git rebase --continue            # 解决reabse冲突之后 继续rebase
	        git rebase --abort               # 放弃rebase
	    git checkout master                  # 切到master分支
	    git merge dev                        # 把dev分支合并到master分支

	git tag                                  # 查看全部标签
	git tag -a TAG_NAME -m 'TAG NOTE'        # 打标签
	git checkout TAG_NAME                    # 切换到标签