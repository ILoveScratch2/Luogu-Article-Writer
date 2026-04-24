> 叠甲，本软件只用于娱乐与学术交流目的。~~并不是鼓励大家不听正课~~，比如老师讲陈年水题时可以交流其它学术内容。请各位合理使用。

[TouchFish Github 源代码。](https://github.com/2044-space-elevator/TouchFish)

**如果您觉得 TouchFish 好用，请点个 star（~~没错我是来讨 star 的~~）。如果您在使用 TouchFish 的时候遇到 bug，请在本文章下提出，最好是去 Github 上提 Issue，以方便流程化处理。如果您想给 TouchFish 添砖加瓦，请提交 PR，感谢您的支持！**


## TouchFish 能干什么?

你有没有机房断网的不快经历，想和同学交流学术内容怎么办？同一个学校里天涯海角的多个人怎么相连？TouchFish 可以帮助你。

如果你有服务器，那么可以直接在服务器上挂着 TouchFish 的进程，**真正实现聊天室的功能**。

## TouchFish 较于传统软件优点是什么？

TouchFish 的优点有：
- 轻量级，它真的很轻。使用 IP 地址链接，使得不用登录，不用加好友。在同一局域网能就能用。
- 还在维护，且使用 Github Actions 来编写编译脚本，可以直接获取最新代码的可执行文件。
- 兼容性很强，~~只要不是史前系统都能跑~~。
- 多系统，有 Linux 与 Windows 的可执行文件（MacOS 的编译脚本还在写），源代码 Python，MacOS 有了 Python 也可以跑。
- 随关随开。
- 一个聊天室可以有多个人，人数可以限制。

## TouchFish 怎么使用？

macOS 党请使用 python 直接运行 Github 上的源代码，其余操作大同小异。

目前有两种下载方法：
- [Github，可以获得最新程序，但对于不太熟悉的同学还是难上。](https://github.com/2044-space-elevator/TouchFish/tree/main/bin)
- [~~我朋友的网站~~，下载速度快，能上。](https://www.bopid.cn/chat/chat.zip)

有 readme 文件，不过我还是手把手教大家如何使用 TouchFish。下载后，你会发现 TouchFish 有两个文件，chat 和 client。

首先聊天室中必须有一人的电脑作为服务器，下面简称服务端。所有参加聊天的人称为客户端。

### 服务端的操作

打开终端，输入 ipconfig（macOS 与 Linux 是 ifconfig），复制“IPv4 地址”。

服务端打开 chat，Connect to IP 后面输入 IPv4 地址。The maxium numbers of account 后面输入聊天室内人员数目上限。

重要的一步来了，请随便找一个 0~65535 的数，终端内输入 `netstat -an`，查找里面有没有这个数，如果没有，可作为端口使用。

回到 chat，输入你找的这个端口，回车。若没闪退，聊天室即开启成功。并将你的 IP 地址和端口分享给聊天室中的其他人。（一般一台电脑的 IP 地址和空闲端口是恒定不变的，分享一次就够了）

![](https://cdn.luogu.com.cn/upload/image_hosting/5pg3wnlv.png)

如果有人链接聊天室或发送消息，则会有提示。

![](https://cdn.luogu.com.cn/upload/image_hosting/vjpkd1ql.png)

### 客户端操作

打开 client，IP 后面输入服务端分享的 IP，端口后面输入服务端分享的端口，Username 爱填啥填啥。

点确定。

![](https://cdn.luogu.com.cn/upload/image_hosting/zr8oayiv.png)


## 总结

点个赞给个 star 吧大佬们。

## 附录：公网上可用的聊天室

大家可以把自己的服务器用来挂，如果有人挂了的可以和我说。

这是我朋友服务器上的：
- IP：`www.bopid.cn`，Port：`7001`，额定：200 人。（不要乱玩，荷载很小，容易挂）