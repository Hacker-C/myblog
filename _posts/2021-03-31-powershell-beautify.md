---
title: Windows Powershell 美化：oh-my-posh 方案
date: 2021-03-31 10:57:35
tags: 
    - 开发技巧
    - Windows Powershell
categories: 开发技巧
layout: post
subtitle: "颜值是第一生产力。"
author: "MurphyChen"
header-img: https://cdn.jsdelivr.net/gh/hacker-c/Picture-Bed@main/pl1.png
---

## 0. 前言

之前写了一个关于 WSL 的配置和使用教程，用着很香。后来有时候需要用到 Powershell，但是 Windows10 自带的 Powershell 太丑了，于是决定美化一下终端。最终效果如下所示：

![Powershell oh-my-posh](https://cdn.jsdelivr.net/gh/hacker-c/Picture-Bed@main/pl1.png)

## 1. 前置准备

1.&nbsp;安装 Windows Terminal 。打开 Windows10 都自带的 Microsoft Store，搜索 Windows Terminal 下载并安装即可。

2.&nbsp;安装特殊编程字体。有些终端的图标和文字显示需要某些特殊的编程字体支持，否则安装好终端主题后显示会出现乱码。
这里推荐 MesloLGM NF 字体，完美支持几乎所有终端主题。

> 下载链接：https://pan.baidu.com/s/1sPgqHyJBGlnU9BiZlVFv_Q 
> 提取码：u0d1 

windows 字体安装方法：直接将字体文件（`.ttf`）拖到 “Windows-设置-字体设置” 中。

3.&nbsp;安装 notepad3。为了方便设置后续主题配置操作，推荐使用 notepad3 代替 自带的 notepad（记事本）。就 1 M 不到大小，非常简洁但是具有高级文本编辑器的大部分功能。安装过程中记得勾选 “代替notepad”，即取代 Windows 自带的记事本。之后打开一些配置文件，默认以 notepad3 打开，更方便我们配置。

notepad3 下载链接：https://www.rizonesoft.com/downloads/notepad3


## 2. 开始配置

1.&nbsp;修改字体。打开 Windows Terminal 的 “设置”，如下所示。如果你已经按照前面的教程安装并设置好了 notepad3，那么设置文件将以 notepad3 方式打开。

![Windows Terminal-设置](https://cdn.jsdelivr.net/gh/hacker-c/Picture-Bed@main/pl3.png)

打开后设置文件后，找到 “Make changes here to the powershell.exe profile.” 这块地方，如下所示。增加 `"fontFace": "MesloLGM NF"` 这行代码，配置之前安装的 “MesloLGM NF” 字体。

![Terminal-设置](https://cdn.jsdelivr.net/gh/hacker-c/Picture-Bed@main/pl4.png)

2.&nbsp;设置权限，修改 Powershell 执行策略为 `RemoteSigned`。右键 **以管理员身份** 打开 Windows Terminal，默认是进入 Powershell 终端，键入以下命令后回车执行：
```bash
Set-ExecutionPolicy RemoteSigned
```
 Windows PowerShell 执行策略允许你确定 Windows PowerShell 加载配置文件并运行脚本的条件。

3.&nbsp;安装 posh-git 和 oh-my-posh 模块，安装起来很简单，就两行命令。

```bash
Install-Module posh-git -Scope CurrentUser 
Install-Module oh-my-posh -Scope CurrentUser
```
4.&nbsp;然后去 Windows 资源文件管理器地址栏输入 `C:\Users\用户名\Documents\WindowsPowerShell`（将用户名改为你自己的），回车即跳转到这个文件夹下。查看是否有 `Microsoft.PowerShell_profile.ps1` 这个文件，若没有则自己创建一个。双击打开（或者以其他文本编辑器打开），在其中增加以下内容：

```bash
Import-Module posh-git 
Import-Module oh-my-posh 
Set-PoshPrompt -Theme Agnoster
```
这个文件是 PowerShell 的配置文件。 其中 `Set-PoshPrompt -Theme Agnoster` 这条命令就是设置你的 PowerShell 的主题。不出意外，你的 PowerShell 就变成了这样：

![PowerShell-Agnoster](https://cdn.jsdelivr.net/gh/hacker-c/Picture-Bed@main/pl2.png)

## 3. 主题修改

我们可以直接在 PowerShell 中使用命令 `Set-PoshPrompt -Theme <主题名>` 来切换主题。

![PowerShell-Agnoster](https://cdn.jsdelivr.net/gh/hacker-c/Picture-Bed@main/pl5.png)

但是这种切换主题的方式不能持持久化修改，重启 PowerShell 主题就变成原来的了。要永久修改，直接在之前的 `Microsoft.PowerShell_profile.ps1` 文件中修改。

![PowerShell-Agnoster](https://cdn.jsdelivr.net/gh/hacker-c/Picture-Bed@main/pl6.png)

 oh-my-posh 给我们内置了很多主题，使用 `Get-PoshThemes` 命令即可查看所有内置主题，然后选择自己喜欢的修改配置文件即可。也可以参考这里：[oh-my-posh themes](https://ohmyposh.dev/docs/themes)

![PowerShell-Agnoster](https://cdn.jsdelivr.net/gh/hacker-c/Picture-Bed@main/pl7.png)

## 4. 升级到oh-my-posh3

目前，oh-my-posh已经升级到v3了，主题也多了很多，推荐升级。

升级只需要下面这一行命令：
```
Update-Module -Name oh-my-posh -Scope CurrentUser
```

若遇到有关 `Scope` 参数的问题，则使用下面的命令：
```
Update-Module -Name oh-my-posh
```

## 5. 主题推荐

在v3中，同样使用 `Get-PoshThemes` 命令可以列出自带的所有主题。
```
Get-PoshThemes
```

![oh-my-posh3](https://cdn.jsdelivr.net/gh/Hacker-C/Picture-Bed@main/blog/oh-my-posh3.6t0bmq1k9m80.png)

## 6. 参考链接

- https://ohmyposh.dev/docs/upgrading
- https://ohmyposh.dev/docs/
- https://ohmyposh.dev/docs/themes
- https://blog.csdn.net/Jeffxu_lib/article/details/84710386