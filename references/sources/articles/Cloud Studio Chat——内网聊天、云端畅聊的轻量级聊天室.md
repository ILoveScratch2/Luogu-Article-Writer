# Cloud Studio Chat：内网环境下的轻量级聊天工具

**Update 2026.1.10:** 因为服务器原因删除了公网的聊天室，并且将原文章发布到了 [Piaoztsdy's Blog](https://www.piaoztsdy.cn/)，欢迎去看。

> 宣传：[TouchFish Owner 的博客](https://www.touchfish.xin/blog) & [TouchFish OJ](https://oj.piaoztsdy.cn/) & [Sleeping Cup](https://www.sleepingcup.com)

> 叠甲：本软件只用于娱乐与学术交流目的，也可用于生活日常交流。**请不要使用本软件上课开小差聊天，因为本软件而被教练抓到等后果请自负！**

[Github 代码仓库](https://github.com/pztsdy/Cloud-Studio-Chat)  
（点击右侧 `Release` 部分可以下载本软件）

**如果你使用了 Cloud Studio Chat，而且觉得好的话，欢迎给我的 Github 项目一个 Star，十分感谢！🙇(当然啦我也是来讨Star的)**\
**如果你在使用的时候发现了 Bug 或者想给这个项目添砖加瓦，你可以在 Github 上提交 Issue，或者干脆直接 Fork 项目提交 PR。**

---

## Cloud Studio Chat 的由来

在某省某中学的机房，电脑是无法上除了洛谷和一本通以外的网络的，所以我们需要一个能在**内网中**交流学术的软件，这就是 Cloud Studio Chat 的最初由来。后来因为看到了 [用 Python 写的 TouchFish](https://www.luogu.com.cn/article/z6se69kk)，所以将其开源。

本程序采用宽松的 MIT 协议。任何人都能够使用、分发、更改本程序的副本。所以我有一个同学就~~制作了一个黑服程序，顺便偷偷把服务端的代码给改了，现在被我删了。~~

本项目主要由以下几个程序组成：
-   `linux-server`：这是linux环境下已经预编译的文件，随Release发布。
-   `client.exe`：聊天客户端程序。
-   `server.exe`：聊天服务器程序。

### 项目核心特性分析

Cloud Studio Chat 是一个基于 C++ 实现的轻量级聊天程序，它具备以下核心功能：
-   **实时消息传递**：用户可以发送和接收即时消息，数据传输高效且稳定。
-   **多用户支持**：支持多个用户同时在线聊天，通过服务端进行统一管理。
-   **简洁的用户界面**：提供一个直观易用的聊天界面，方便快速上手。
-   **C++ 实现**：利用 C++ 的高性能特性，使得程序运行速度快、资源占用少。
-   **断外网可用**：可以通过在内网中运行服务端来建立聊天室，不受外网连接限制。
-   **服务端管理**：服务端具备发送管理员消息、查看用户列表以及踢出用户等管理功能。

---

## Cloud Studio Chat 的使用方法

### 1. 构建说明

在开始使用前，你需要确保满足以下先决条件：
-   **操作系统**：Windows 7 及以上版本（在 Linux 系统上也已测试通过，例如 Debian 及其衍生版本 Ubuntu）。
-   **系统架构**：$32/64$ 位（可以自行编译）。
-   **C++ 编译器**：推荐使用 GCC，作者的环境是 MSYS2 GCC。网上有详细的安装教程。
-   **编译问题**：如果有用户反馈无法运行或编译，解决方案已在项目的 Issues 页面置顶的 Issue `#3` 中提供。（$\textrm{\color{red}{如需自行编译必须看}}$）

**具体步骤：**

当然，你也可以直接到项目的 Release 界面直接获取二进制版本。
1.  **克隆仓库**：
    ```bash
    git clone [https://www.github.com/pztsdy/Cloud-Studio-Chat.git](https://www.github.com/pztsdy/Cloud-Studio-Chat.git)
    cd Cloud-Studio-Chat
    ```
2.  **编译文件**：
    你可以直接运行项目目录下的 `compile.bat` 文件，进行一键编译。
3. **对于Linux**：
    目前只有服务端做了 Linux 的支持。请使用 g++ 进行编译。（更建议使用 Release 中的版本）

### 2. 程序功能与使用

编译成功后，你将获得可执行文件。`server.exe` 用于开启聊天室，`client.exe` 用于连接聊天室。

-   **高效交流**：客户端支持置顶功能，方便用户在写代码时也能进行高效的学术交流。
-   **多用户在线**：支持多用户同时连接与聊天。
-   **管理员公告**：管理员可以发送全员消息，方便发布通知。
-   **长文本支持**：程序具有超长的缓冲区，支持传输长文本内容。

### 3. 使用例子

![](https://cdn.luogu.com.cn/upload/image_hosting/aabcg6zb.png)

![](https://cdn.luogu.com.cn/upload/image_hosting/3k9ey3hr.png)

---

## 贡献与许可证

欢迎对本项目进行贡献！如果您有任何建议或发现 Bug，请随时提交 Issue 或 Pull Request。

本项目采用 **MIT 许可证** 发布。

### 最后的内容
1. 希望大佬们能给个 Star 求求啦
2. **现在 Cloud Studio Chat 可能不会再进行更多支持了，欢迎移步 TouchFish 获得更好的体验（我现在主要在进行 TouchFish 的开发）**