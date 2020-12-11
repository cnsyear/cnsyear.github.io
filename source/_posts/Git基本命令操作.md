---
title: Git基本命令操作
toc: true
abbrlink: ce227c93
date: 2020-12-11 10:34:17
tags: Git
categories: 版本控制
---

#### 一、Git工作流程图

我们先来理解下 Git 工作区、暂存区和版本库概念：

- 工作区：就是你在电脑里能看到的目录。
- 暂存区：英文叫 stage 或 index。一般存放在 .git 目录下的 index 文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。
- 版本库：工作区有一个隐藏目录 .git，这个不算工作区，而是 Git 的版本库。

<!-- more -->

![image](/static/img/2.png)

图中左侧为工作区，右侧为版本库。在版本库中标记为 "index" 的区域是暂存区（stage/index），标记为 "master" 的是 master 分支所代表的目录树。

图中我们可以看出此时 "HEAD" 实际是指向 master 分支的一个"游标"。所以图示的命令中出现 HEAD 的地方可以用 master 来替换。

图中的 objects 标识的区域为 Git 的对象库，实际位于 ".git/objects" 目录下，里面包含了创建的各种对象及内容。

当对工作区修改（或新增）的文件执行 git add 命令时，暂存区的目录树被更新，同时工作区修改（或新增）的文件内容被写入到对象库中的一个新的对象中，而该对象的ID被记录在暂存区的文件索引中。

当执行提交操作（git commit）时，暂存区的目录树写到版本库（对象库）中，master 分支会做相应的更新。即 master 指向的目录树就是提交时暂存区的目录树。

当执行 git reset HEAD 命令时，暂存区的目录树会被重写，被 master 分支指向的目录树所替换，但是工作区不受影响。

当执行 git rm --cached <file> 命令时，会直接从暂存区删除文件，工作区则不做出改变。

当执行 git checkout . 或者 git checkout -- <file> 命令时，会用暂存区全部或指定的文件替换工作区的文件。这个操作很危险，会清除工作区中未添加到暂存区的改动。

#### 二、常用命令

下面只记录常用的命令，对于不常用的请查阅[Git官方命令手册](https://git-scm.com/docs)。

##### 1、创建仓库

```
#使用当前目录作为Git仓库
git init 
git init <directory>

#克隆远程仓库
git clone <repo>
git clone <repo> <directory>

#配置提交代码时的用户信息
git config --global user.name "zhaojie"
git config --global user.email test@qq.com
```

##### 2、基本操作

```
#添加文件到仓库
git add [file1] [file2] ..
git add [dir]
git add .  # 添加当前目录下的所有文件到暂存区

#查看仓库当前的状态，显示有变更的文件
git status

#比较文件的不同，即暂存区和工作区的差异
git diff [file]

#显示暂存区和上一次提交(commit)的差异:
git diff --cached [file]

显示两次提交之间的差异:
git diff [first-branch]...[second-branch]

#提交暂存区到本地仓库
git commit -m [message]
git commit [file1] [file2] ... -m [message]
git commit -a -m [message]

#回退版本
git reset [--soft | --mixed | --hard] [HEAD]

--mixed 为默认，可以不用带该参数，用于重置暂存区的文件与上一次的提交(commit)保持一致，工作区文件内容保持不变
git reset HEAD^ # 回退所有内容到上一个版本  
git reset 052exxxx # 回退到指定版本
git reset --hard origin/master # 将本地的状态回退到和远程的一样 

--soft 参数用于回退到某个版本：
git reset --soft HEAD

--hard 参数撤销工作区中所有未提交的修改内容，将暂存区与工作区都回到上一次版本，并删除之前的所有信息提交
git reset –hard HEAD~3  # 回退上上上一个版本  
git reset –hard bae128  # 回退到某个版本回退点之前的所有信息。 
git reset --hard origin/master # 将本地的状态回退到和远程的一样 

#删除工作区文件
git rm <file> #将文件从暂存区和工作区中删除
git rm --cached <file> #将文件从暂存区域移除，但仍保留在当前工作目录中

#移动或重命名工作区文件
git mv [file] [newfile]
git mv -f [file] [newfile] #如果新但文件名已经存在，但还是要重命名它，可以使用 -f 参数
```

##### 3、提交日志

```
#查看历史提交记录
git log
git log --oneline #查看历史记录的简洁的版本。

#以列表形式查看指定文件的历史修改记录
git blame <file>

```

##### 4、远程操作

```
#远程仓库操作
git remote	
git remote add [shortname] [url] #添加远程版本库
git remote add origin https://xxx.git

#从远程获取代码库
git fetch	

#下载远程代码并合并
git pull <远程主机名> <远程分支名>:<本地分支名>
git pull origin master:brantest #将远程主机 origin 的 master 分支拉取过来，与本地的 brantest 分支合并。

#上传远程代码并合并
git push <远程主机名> <本地分支名>:<远程分支名>
git push origin master:master #将本地的 master 分支推送到 origin 主机的 master 分支。
git push --force origin master #本地版本与远程版本有差异，强制推送可以使用 --force 参数
git push origin --delete master #删除主机但分支可以使用 --delete 参数，以下命令表示删除 origin 主机的 master 分支
```

##### 5、分支管理

```
#创建分支命令：
git branch (branchname)

#切换分支命令:
git checkout (branchname)

#列出分支基本命令：
git branch

#删除分支命令：
git branch -d (branchname)

#分支合并 将任何分支合并到当前分支中去
git merge
```

##### 6、标签管理

```
#查看所有标签：
git tag

#创建标签
git tag tagName
```