---
title: 玩转WSL(10)之由 WSL 升级到 WSL2
date: 2021-08-09 13:17:49
layout: post
subtitle: "WSL2 出来了，支持 Docker，但是也有缺点。"
author: "MurphyChen"
header-img: https://s3.ax1x.com/2021/02/14/ysagaT.jpg
tags:
  - WSL
---

##  1. WSL1与WSL2的比较

WSL 和 WSL2各有优劣，大家可以根据自己的需求来决定自己是否需要升级到WSL2。另外，WSL和WSL2之间的转换是很容易的，随时可以切换版本。

微软官方关于两者的区别如下图所示：

![wsl1与wsl2的区别](https://cdn.jsdelivr.net/gh/Hacker-C/Picture-Bed@main/blog/wsl2.7h3hu8h5a4w0.png)

可以看出，WSL2相对WSL1的优势：
- 有了完整的 Linux 内核
- 使用 VM 虚拟机技术，有更加完整更好的 Linux 体验
- 完全的系统调用兼容性
- 在实际使用WSL2的过程中，发现WSL2可以稳定运行使用 Docker，不像在WSL1上会出现各种各样的问题。

但是，WSL2也有一些缺点，不知道是不是BUG，这也是为什么WSL2被很多人吐槽：
- 跨操作系统文件读取的性能降低。也就是说你在 WSL 的文件系统里读取 Windows的文件，效率很低。
- 我本人在使用WSL2的时候，发现运行Hexo相关的命令变得很慢。而且每次后台编辑完文档，博客页面都不能实时更新，需要重新启动Hexo服务才能更新。

综合来讲，是否使用WSL2完全取决于你的需求。如果你想更好的体验 Linux，想在WSL上跑Docker，那就升级WSL2。否则就继续使用WSL1，没必要升WSL2了。

> 更多：[Microsoft Docs 比较WSL1与WSL2](https://docs.microsoft.com/zh-cn/windows/wsl/compare-versions)

##  2. 由 WSL 升级到 WSL2

###  2.1 安装WSL的前提要求

要更新到 WSL2，必须是Windows10，而且版本≥1903。或者内部版本≥18362。若不满足，请升级系统。

###  2.2 启用虚拟机平台可选组件功能

以 **管理员身份** 打开 PowerShell 并执行以下命令：
```PowerShell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
如下所示：
![启动Windows虚拟机功能](https://cdn.jsdelivr.net/gh/Hacker-C/Picture-Bed@main/blog/wsl2-2.3vvuxckdm2e0.png)

启动完成后重启电脑，然后以管理员再次打开 PowerShell，输入 `wsl -l` 查看已经安装的子系统的 Ubuntu 版本。
![查看你的子系统Ubuntu版本](https://cdn.jsdelivr.net/gh/Hacker-C/Picture-Bed@main/blog/wsl2-3.5hja77srnxc0.png)
我这里显示的是Ubuntu，需要根据自己实际显示的名称来进行下一步（例如有的是Ubuntu 20.04）。

###  2.3 将WSL1升级安装为WSL2

同样在 PowerShell 中输入以下命令，注意我的子系统版本号显示就叫 Ubuntu，你需要根据自己系统上显示的进行更改。然后等待安装完成。
```PowerShell
wsl --set-version Ubuntu 2
```
如下所示：
![安装WSL2](https://cdn.jsdelivr.net/gh/Hacker-C/Picture-Bed@main/blog/wsl2-4.vi80vl29b3k.png)

输入 `wsl -l -v`，查看WSL的版本，可以看到已经成功升级到WSL2。
![wsl2-5](https://cdn.jsdelivr.net/gh/Hacker-C/Picture-Bed@main/blog/wsl2-5.59iahrxqrg40.png)

###  2.4 将WSL2设置为默认版本

在 PowerShell 输入以下命令，使得之后安装的Linux子系统都安装到WSL2中。
```PowerShell
wsl --set-default-version 2
```

###  2.5 将WSL2版本回退至WSL1

WSL1 和 WSL2版本之间切换很容易，直接输入以下命令。只需修改版本号 `1` 和 `2` 即可切换。
```PowerShell
wsl --set-version Ubuntu 1
```

##  3. 参考链接

- [知乎-从wsl到wsl2明显是退步，为什么还有人鼓吹wsl2？](https://www.zhihu.com/question/424191615)
- [Microsoft Docs 比较WSL1与WSL2](https://docs.microsoft.com/zh-cn/windows/wsl/compare-versions)
- [Microsoft Doc 在Windows10上升级WSL2](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10)
