### 前前言

本文使用 DeepSeek 辅助生成。我已保证我的贡献不小于 GenAI。

### 前言

WSL，即 Windows Subsystem for Linux，适用于 Linux 的 Windows 子系统。其可以让我们在 Windows 上直接运行 Linux 环境。对于算法竞赛选手而言，WSL 提供了一个绝佳的解决方案：既能使用熟悉的 Windows 操作系统，又能使用 Linux 下的编译工具链，完美兼容/适应竞赛环境的要求。无需安装虚拟机或配置双系统，即可实现高效、便捷的开发体验。

效果示意图：

![](https://cdn.luogu.com.cn/upload/image_hosting/8duq2fb5.png)

可以注意到下方的终端变成了 Ubuntu 终端。

本文将介绍如何在竞赛用途下配置 WSL，主要内容如下：

- 安装 WSL

- 配置 Ubuntu

- 使用 WSL 终端代替命令提示符/PowerShell

不同版本的 Windows 在安装及使用上方法可能略有不同，笔者使用的操作系统为 Windows 10 专业版 22H2。

注意：Windows 10/11 家庭版同样支持安装 WSL 2，请确保系统已更新至较新版本（Windows 10 版本 2004 及更高版或 Windows 11）。

## Part 1：安装 WSL

### 1. 启用虚拟化与 Windows 功能

打开控制面板 - 程序 - 程序与功能，并在左侧找到“启用或关闭 Windows 功能”。

找到“适用于 Linux 的 Windows 子系统”与“虚拟机平台”两个选项，勾选并确定，如下图所示：

![](https://cdn.luogu.com.cn/upload/image_hosting/4d96awg9.png)

设置完毕后，**重启计算机**。

重启后，**以管理员身份**打开 PowerShell 或命令提示符，输入以下命令：

```powershell
wsl --install
```

此命令将安装 WSL 本体。完成后，**再次重启计算机**。

由于网络环境问题，你可能会遇到下载缓慢或失败的情况。如果卡在下载步骤，可能需要科学上网，或者将 DNS 改为 `114.114.114.114`。

### 2. 安装 Ubuntu

打开 Microsoft Store，并搜索 Ubuntu。

你应该能看到前缀名为 Ubuntu 的各种应用，后面的数字代表的是版本号。

![](https://cdn.luogu.com.cn/upload/image_hosting/u8vfwibd.png)

任选其中一个下载即可，笔者选择的是 Ubuntu 24.04.1 LTS。

## Part 2：配置 Ubuntu

### 1. 初始化与基本操作

安装完成后，在开始菜单中找到 Ubuntu 并启动。

![](https://cdn.luogu.com.cn/upload/image_hosting/tp130370.png)

首次启动会等待几分钟进行初始化，然后提示你创建新用户名和密码（**请牢记此密码，`sudo` 命令时需要**）。注意在 Linux 中，输入密码时屏幕上不显示文字属于正常现象。

设置好帐户名和密码后，你应当拥有了一个完整的 Ubuntu 终端，效果如下：

![](https://cdn.luogu.com.cn/upload/image_hosting/tjy7q9cm.png)

为了后续安装软件顺畅，你可以选择更换软件源为清华镜像源，以解决 `apt update` 和 `apt install` 的网络问题。更换软件源的方式可以参考：<https://zhuanlan.zhihu.com/p/664403552>，注意选择与 Ubuntu 版本相对应的软件源。

### 2. 安装 g++ 与 gdb

```bash
# 更新软件包列表
sudo apt update

# 安装 g++ 编译器套件和 gdb 调试器
sudo apt install g++
sudo apt install gdb
```

## Part 3：使用 WSL 终端代替命令提示符/PowerShell

如果你的编码模式是直接新开一个终端来编译，那直接把终端换成 Ubuntu 终端切换到对应目录即可。具体地，ubuntu 下的 `/mnt` 路径就对应着 Windows，如 Windows 下的 `D:\Code` 对应着 Ubuntu 下的 `/mnt/d/Code` 路径。

如果你的编码模式是使用 VS Code 的默认终端进行编译，打开 VS Code 终端，找到终端右侧的一个小箭头，点击并选择“选择默认配置文件”，并在弹出的窗口中选择 WSL 作为默认配置文件，重启 VS Code 即可。

![](https://cdn.luogu.com.cn/upload/image_hosting/k3wibm0g.png)

![](https://cdn.luogu.com.cn/upload/image_hosting/r5a7l6k3.png)

现在，你可以使用 Ubuntu 终端进行编译 (`g++ code.cpp -o code`) 和运行 (`./code`)，所有操作都在纯净的 Linux 环境下进行。

附 1：

笔者在所有看到的关于 WSL 结合 VS Code 使用的文章中，都是下载 VS Code 中的 WSL 扩展，再使用这个扩展连接到 Ubuntu。

理论上来说这种方式有更为纯净的 Linux 环境，整个 VS Code 包括工作区都置于 Linux 环境中（上面的做法只是开了一个 Ubuntu 终端，VS Code 仍然置于 Windows 环境中，同样的工作区也在 Windows 里）。

但是从笔者个人使用经验来看，首先前后者在模拟考场环境上完全没有差别，其次后者内存占用非常大，在笔者的笔记本电脑上**一度到达了 8GB**，有时甚至有明显的卡顿感。相比之下前者只需要 500MB~1GB 不等的内存，使用较为流畅，对算法竞赛选手显然是更为友好的。

附 2：如果你发现无法使用 Ctrl+Shift+C 复制，Ctrl+Shift+V 粘贴，打开 Ubuntu 终端，右键白色框框点击属性，将选项中的“将 Ctrl+Shift+C/V 用作复制/粘贴的快捷键”勾选并确定即可，如下图所示。可能需要重启 Ubuntu 终端/VS Code 才能生效。

![](https://cdn.luogu.com.cn/upload/image_hosting/0mf5mhh9.png)

![](https://cdn.luogu.com.cn/upload/image_hosting/menbf2pi.png)

## 总结

WSL，真的很牛，很好用。

希望这篇文章能帮到大家！