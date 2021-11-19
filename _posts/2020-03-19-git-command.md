---
title: git 常用指令
date: 2020-03-03 16:50:52
layout: post
subtitle: "git常用指令总结"
author: "MurphyChen"
## header-img: https://s3.ax1x.com/2021/02/14/ysagaT.jpg
tags:
  - git
---

对 git 常用命令进行了总结，方便查询。

## git 基本配置

### 用户信息

全局配置/修改个人用户信息和电子邮件地址

```git
git config --global user.name "user_name"
git config --global user.email user_email
```

`--global` 是全局配置，这里指你的所有项目都会默认使用所配置的用户信息
给某个项目特定配置用户信息，请去掉 `--global`

### 查看用户信息

查看所有信息

```git
git config --list
```

查看用户名/邮箱

```git
git config user.name
git config user.email
```

### 生成本地公钥和私钥

生成本地公钥，在github创建 SSH key，从而达到一种身份认证的效果。将公钥复制到所创建的 SSH key中，从而获得了一把钥匙，只有拥有钥匙的人能提交代码到你的仓库。

输入以下命令，会让你输入密码，一般一路回车就行。

```git
ssh-keygen -t rsa
```

之后会在 .ssh 文件夹里面生成一个私钥 id_rsa 和一个公钥 id_rsa.pub。  

在 Windows 系统中，公钥和密钥会在 `C:\Users\<your_computer_name>\.ssh` 中。  
在 Linux 系统中，会生成在 `~/.ssh` 中，使用以下命令即可查看。
```git
cat ~/.ssh/id_rsa.pub
```

### 文本编辑器

git 的默认文本编辑器是 Vim 或 Vi，可以根据偏好重新设置

```git
git config --global core.editor <your_editor>
```

## git 基本操作

### git init

初始化项目，创建一个 git 本地仓库，在该目录下生成一个 `.git` 文件夹。

```git
mkdir projcet
cd project
git init
ls
project/ .git/
```

### git add

将工作区文件的改动添加到暂存/缓存区

- 添加项目中所有修改的、新增的文件

```git
git add .
```

- 添加项目所有修改的、新增的、删除的文件

```git
git add -A
```

- 添加指定文件

```
git add <file1> <file2>
```

### git status

查看项目内文件的提交/缓存状态

```bash
git status
```

加上参数 `-s` 显示状态的细节

```git
$ git status -s
?? file5 ## 新添加的未跟踪文件
A  file3 ## 新添加到暂存区中的文件
MM file1 ## 修改过的文件
 M file2 ## 右边的 M 表示该文件被修改了但是还没放入暂存区
M  file4 ## 左边的 M 表示该文件被修改了并放入了暂存区
```

### git commit

git 每一次提交都将记录用户名和邮箱，所以要先配好信息。
提交到版本库， `-m` 属性后面是本次提交的注释。
Github 里面的日志就是根据 `commit` 计算的。

```git
git commit -m "message"
```

可以用添加 `-a` 属性跳过 `git add`这一步

```git
git commit -a -m "message"
```

### git push

`git push` 将本地库分支推送到远程库分支（github）
命令格式：`git push <远程主机名> <本地分支名> <远程分支名>`
远程主机名为 origin，本地分支为 master，省略远程分支名：（常用）

```git
git push origin master
```

当远程库是空的（首次推送）时候，加上参数 `-u`，就会将本地的 `master` 分支和远程的 `master` 分支关联起来，在以后 `push` 或 `pull` 时就可以简化命令。

```bash
git push -u origi master
```

下次推送或拉取使用简化命令：

```
git pull
git push
```

### git pull

`git pull` 将远程主机的 `master` 分支最新内容拉下来后与当前本地分支直接合并

```git
git pull origin master
```

### git fetch

`git fetch` 将远程主机的最新内容拉到本地，不进行合并

```git
git fetch origin master
```

### git clone

克隆远程库的项目到本地
git clone <project_url> <workspace_folder>

````

### git diff

`git diff` 用于比较两次修改的差异

工作区 VS 暂存区
```git
git diff ## git diff无参数默认情况
````

暂存区 VS 版本库/Git 仓库

```git
git diff --cached
```

工作区 VS 版本库

```git
git diff HEAD
```

显示摘要

```git
git diff --stat
```

### git reset

取消已经缓存到缓存区的内容

```git
git reset HEAD
```

### git rm

从工作区删除

```git
git rm <filename>
```

从缓存区强制删除

```git
git rm -f <filename>
```

从缓存区删除但仍保留在工作区

```git
git rm --cached <filename>
```

通配删除，将会递归删除当前目录下所有子目录和文件

```git
git rm -r *
```

### git mv

重命名一个文件

```git
git mv <from_from> <file_to>
```

### git remote

本地 Git 库和和远程仓库是通过 SSH 加密的，需要提前配置好
查看当前有哪些远程库

```git
git remote
git remote -v
```

新增远程库

```git
git remote add origin <project_url>
git remote add <远程仓库别名> <project_url>
```

删除远程库

git remote rm origin
git remote rm <远程仓库别名>
