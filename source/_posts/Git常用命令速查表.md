
---
title: Git常用命令
date: 2016-04-24 22:43:42
author: wúzguó
avatar: /images/avatar.png
authorLink: https://wzguo.github.io
authorAbout: https://github.com/wzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 资源
tags: 
	- Git
keywords: Git
photos:
	- /images/avatar.png
description: Git常用命令速查表
---


##Git常用命令速查表


1. 创建版本库
git clone ＜url>　　　　#克隆远程的版本库
git init　　　　　　#初始化本地版本库
git checkout -b  本地分支名  远程分支名　	　			#获取远程分支的代码

***
2. 修改和提交
git status　　　　　　#查看状态
git diff　　　　　　#查看变更内容
git add　　　　　　#跟踪所有改动过的文件
git add ＜file>　　　#跟踪指定的文件
git mv ＜old> ＜new>	　　　#文件改名
git rm ＜file>　　　　#删除文件
git rm --cached ＜file>　　　　#停止跟踪文件但不删除
git commit -m "commit message"　　　　#提交所有更新过的文件
git commit --amend　　　　#修改最后一次提交

***
3. 查看提交历史
git log　　　　#查看提交的历史
git log -p ＜file>　　　#查看指定文件的提交历史
git blame ＜file>　　　#以列表方式查看指定文件的提交历史

***
4. 撤消 
git reset --hard HEAD　　　　　#撤消工作目录中所有未提交文件的修改内容
git checkout HEAD ＜file>　　　　　#撤消指定的未提交文件的修改内容
git revert ＜commit>　　　　　#撤消指定的提交
git reset --hard HEAD　　　　　#放弃暂存区B和工作目录C的所有修改，恢复最近一次提交状态 （A ->C 所有文件)
git checkout -- ＜file_name>　　　　#恢复某文件到最近一次提交状态，放弃checkout后的修改 (A -> C 指定文件)
git revert HEAD　　　　　#撤消最近的一个提交 （针对已经的commit，跟B和C无关）
git revert HEAD^　　　　　# 撤消上次”(next-to-last)的提交 （^ 个数可以递增）

*** 

5. 分支与标签
git branch　　　　　　　#显示所有本地分支
git branch ＜branchName>　　　　　#创建新分支
git checkout ＜branchName/tagName>　　　　　#切换到指定分支或标签
git checkout -b ＜branchName>　　　　#创建新分支并切换到该分支
git branch -d ＜branchName>　　　　#删除本地分支
git branch -D ＜branchName>　　　　　#强制删除本地分支
git tag　　　　　　　#列出所有本地标签
git tag ＜tagname>　　　　　　#基于最新提交创建标签
git tag -d ＜tagname>　　　　　#删除标签
git checkout ［options] [＜branchName>] -- ＜file>...　　　 #复制文件到指定分支
git diff ＜branch> --stat　　　　　　#比较branch分支与当前分支差异

*** 
6. 合并与衍合
git merge ＜branch>　　　　　#合并指定分支到当前分支
git rebase ＜branch>　　　　　#衍合指定分支到当前分支

*** 
7. 远程操作
git remote -v　　　　　　#查看远程版本库信息
git remote show ＜remote>　　　　　#查看指定远程版本库信息
git remote add ＜remote> ＜url>　　　　　#添加远程版本库
git fetch ＜remote>　　　　　　#从远程库获取代码
git pull ＜remote> ＜branchName>　　　　　　　#下载代码及快速合并
git push ＜remote> ＜branchName>　　　　　　#上传代码及快速合并
git push ＜remote> : ＜branchName/tag-name>　　　#删除远程分支或标签
git push --tag　　　　　　　　#上传所有标签
git branch -r　　　　　　　　#查看远程分支

***
8. 删除操作
git rm --cached readme.txt　　　　　　#文件放弃跟踪，但是还是保留在工作目录中。 
　　　　　　　　　　　　　　　　　　　没有--cached 选项，就完全删除文件

git rm filename　　　　　　　#直接删除文件

通过“git rm --cached README.txt”命令，可以将文件状态还原为未暂存状态，即回到“Untracked files”文件状态。
通过“git add README.txt”命令将已修改文件更新到暂存区域中，如果想撤销修改。

可以使用“git checkout -- README.txt”命令。

通过git reset HEAD ＜file>取消暂存

***
9. 总结
先git add你修改过的文件，再git diff并git status查看确认，然后git commit提交，然后输入你的开发日志，最后git log再次确认。

