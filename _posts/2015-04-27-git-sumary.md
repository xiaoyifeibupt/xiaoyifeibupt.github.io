---
layout: post
title: git使用总结
categories: [git]
---

##安装

在Linux上安装Git，你可以试着输入git，看看系统有没有安装Git：

	$ git
	The program 'git' is currently not installed. You can install it by typing:
	sudo apt-get install git

安装完成后，还需要最后一步设置，在命令行输入：

	$ git config --global user.name "Your Name"
	$ git config --global user.email "email@example.com"

注意git config命令的--global参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。

##创建仓库初始化

初始化一个Git仓库，使用`git init`命令。

添加文件到Git仓库，分两步：

第一步，使用命令`git add <file>`，注意，可反复多次使用，添加多个文件；

第二步，使用命令`git commit`，完成。

##版本控制

要随时掌握工作区的状态，使用`git status`命令。

如果`git status`告诉你有文件被修改过，用`git diff`可以查看修改内容。


###版本回退

`HEAD`指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。

穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。

要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。

###管理修改

每次修改，如果不`add`到暂存区，那就不会加入到`commit`中

###撤销修改

场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD file`，就回到了场景1，第二步按场景1操作。


###删除文件

命令`git rm`用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失最近一次提交后你修改的内容
