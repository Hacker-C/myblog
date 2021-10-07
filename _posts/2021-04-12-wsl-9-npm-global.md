---
title: 玩转WSL(9)之修改 npm 全局安装位置
date: 2021-04-12 17:46:57
layout: post
subtitle: "修改 WSL 下 npm 全局安装的位置。"
author: "MurphyChen"
header-img: https://s3.ax1x.com/2021/02/14/ysagaT.jpg
tags:
  - WSL
---

## 前言

> 本文适用于 WSL 的 oh-my-zsh 终端环境。

在使用 npm 全局安装一个工具 gulp 的时候，遇到了一个报错：

```txt
npm ERR! EACCES permissions errors when installing packages globally
npm ERR! It is likely you do not have the permissions 
to access this file as the current user.
```

大意是说，我没有权限全局安装在那个默认安装全局软件的文件夹下。

谷歌搜索了一番，最终找到了 npm 官方解答:

[Resolving EACCES permissions errors when installing packages globally](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally)

官方推荐了两种方法，一种是通过 npm 结点管理器重新安装 npm，另一种是修改 npm 的全局安装位置。

这里我选择了第二种方法，成功解决，记录一下。

## 修改 npm 全局安装位置

1. 在用户根目录 `~` 新建 `.npm-global` 文件夹（名字可以自取），然后进行修改。

    ```txt
    mkdir ~/.npm-global
    ```

2. 设置 npm 全局安装位置为刚才所创建的文件夹 `.npm-global`

    ```txt
    npm config set prefix '~/.npm-global'
    ```

3. 修改环境变量以保证可以正常使用所安装的软件。
    使用 vim 或者其他编辑器 打开编辑 `~/.profile`（若没有这个文件，则创建一个，同名）

    ```txt
    vim ~/.profile
    ```

    在最后加上这一行：
    ```txt
    export PATH=~/.npm-global/bin:$PATH
    ```
4. 刷新配置文件，让配置生效。

    ```txt
    source ~/.profile
    ```
5. 重新使用 npm 全局安装之前失败的软件，查看是否解决问题。

## 新的问题

通过以上设置后，我发现每次重新打开 WSL，都需要手动使用 `source` 命令刷新配置文件，配置才会生效。于是想到，每次打开 zsh 系统都会自动运行一遍 `~/.zshrc` 文件，所以我就把 `source ~/.profile` 这个命令写到 `~/.zshrc` 中了。

编辑 `~/.zshrc`，添加下面这一行：

```txt
source ~/.profile
```

至此，问题解决。

## 参考

- [NPM Docs Resolving EACCES permissions errors when installing packages globally](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally)
- [https://blog.csdn.net/science_Lee/article/details/79214127](https://blog.csdn.net/science_Lee/article/details/79214127)