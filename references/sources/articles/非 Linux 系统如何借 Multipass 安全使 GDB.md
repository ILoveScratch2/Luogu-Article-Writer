# 0x-1 前言
本人用 2017 破 macbook，想装个 GDB，调试进程被 SIP[^1] 直接杀死，关了一次 SIP 又觉不妥。虽然 MacOS 有更加强大的 LLDB，但是考场用不了啊！

正好笔者前些年用 Multipass 试图部署 StableDiffusionWebUI 无果，现重拾此物，使其为 OI 效力。
# 0x00 下载、安装 Multipass
Multipass 是 Ubuntu 旗下的一个命令行虚拟机软件，可以在**主机终端**运行 Ubuntu 虚拟机。

访问[官网](//multipass.run)，点击下载，静待即可。必要可食用七根棍。
## 0x00.5 官方 Windows 安装教程
### [下载适用于 Windows 的 Multipass](https://canonical.com/multipass/download/windows)
需要 **Windows 10 专业版/企业版/教育版 v1803 或更高版本**，
或者任意 Windows 10 搭配 VirtualBox。
### 确保网络为专用网络
请确认本地网络被设置为 **专用网络**，
否则 Windows 会阻止 Multipass 启动。
### 运行安装程序
你需要允许安装程序获取 **管理员权限**。
## 0x00.A 官方 MacOS 安装教程
### [下载适用于 macOS 的 Multipass](https://canonical.com/multipass/download/macos)
或者，请参考我们的文档，了解如何使用 **brew** 进行安装。
### 运行安装程序
你需要在具有 **管理员权限** 的账户下运行安装程序。
# 0x01 创建虚拟机
以 MacOS 为例，首先打开应用，会有这样的界面：

![](https://pic1.imgdb.cn/item/68c587d458cb8da5c8a79e55.png)

我们找到最新的 25.04，点击下面中间的齿轮进去：

![](https://pic1.imgdb.cn/item/68c5893958cb8da5c8a7a4bc.png)

设置虚拟机名称，CPU 数量（推荐 2 个），内存（**一般需要 4G，炸了就老实了**），磁盘（16GB 足够）。

然后是关键一步：**设置链接目录**

这样你就可以在主机写代码，交送虚拟机进行运行了。

往下翻，翻到 Mount 一项，点击 Add Mount，前为主机目录，后为在虚拟机的目录，可添加多个。

![](https://pic1.imgdb.cn/item/68c58a2958cb8da5c8a7a68f.png)

接下来，点击启动 Launch。

![](https://pic1.imgdb.cn/item/68c58a6958cb8da5c8a7a772.png)

（新版的比以前好用多了！）

等右下角出现绿色提示，就可以点击中间的 Open Shell 按钮打开命令行。

![](https://pic1.imgdb.cn/item/68c5916458cb8da5c8a7c358.png)

现在发现一个问题，就是**它无法访问挂载的文件夹中的文件，原因是权限不够**。这样，我们打开系统设置 $\to$ 隐私与安全 $\to$ **完全磁盘访问权限**，勾选 Multipassd 这一项：

![](https://pic1.imgdb.cn/item/68c60cf18d8069e937bca19d.png)

但你会问了，这样我怎样高效使用呢？别急，我来教你使用**主机命令行**访问虚拟机。
# 0x02 主机命令行访问虚拟机
关闭刚刚的窗口，打开终端，依次输入 
```sh
multipass start <你的虚拟机名>
multipass shell <你的虚拟机名>
```
![](https://pic1.imgdb.cn/item/68c593b058cb8da5c8a7cb11.png)

即可进入虚拟机终端。
# 0x03 实战
也可使用 vscode 等。
## 0x03.3 配置环境
我们的虚拟机默认是没有 g++、gdb 的，所以我们要安装，方法也很简单，运行
```sh
sudo apt install g++
sudo apt install gdb
```
## 0x03.7 在 Xcode 中编写程序，虚拟机中运行
因为 Apple clang 比较的神奇，会有一些 [bug](https://www.luogu.com.cn/discuss/1152011)。

那我们就在主机写程序，放到虚拟机去运行。

**确保你挂载了 Xcode 目录**。

![](https://pic1.imgdb.cn/item/68c60fad8d8069e937bca65f.png)

打开虚拟机，进入终端，访问 Xcode 项目目录，例如我这个只需要 `cd dev` 即可。

然后 g++ 编译、gdb 调试即可：

![](https://pic1.imgdb.cn/item/68c612698d8069e937bcaa9b.png)

也可 time 查看时间
## 0x03.B 在 Vscode 中编写程序，虚拟机运行
与上面一样，不过终端可以换成 Vscode 带的。

![](https://pic1.imgdb.cn/item/68c618b18d8069e937bcaf28.png)
# 0x~1 结语
本文介绍了如何使用 Multipass 来在非 Linux 平台上获得 Linux 环境（如 gdb 等），希望对大家有所帮助！

（顺便提一嘴，如果要 debug，g++ 编译命令中 `-g` 要在 `-o` 前面否则编译不成）

完结撒花，求赞！
# 0x~0 解释说明
[^1]: 「SIP」系统保护进程，类似杀毒软件的存在，但是关闭启用必须在恢复模式（就像安装 Ubuntu 时的那个界面）下进行。