# 前言

众所周知，Windows 98 是微软在 1998 年发布的一代操作系统，在 2006 年停止技术支持。那么我们能否在 2026 年使用 Windows 98 在洛谷上做题呢？

# 虚拟机配置

- 内存：256 MB
- 处理器：1 核
- 硬盘：10 GB

# 安装

安装过程略。安装完后就能看到以下界面。

![](https://cdn.luogu.com.cn/upload/image_hosting/kzwjdkel.png)

是不是非常 ~~古老~~ 经典。

# 开发环境

C++ 作为信息学竞赛中常用的语言，我这次选择了 Dev-C++ 作为 IDE。Dev-C++ 作为一款从 1998 年开始编写的古老软件，自然有原生支持 Windows 98 的版本。最后支持 Windows 98 的版本应该是 `Bloodshed Dev-C++ 4.9.9.2`。这个版本也很好找，下载后复制到虚拟机中，直接一直下一步就能装上了。

![](https://cdn.luogu.com.cn/upload/image_hosting/hegwukbq.png)

这个界面是不是非常熟悉？没错，几十年过去了，Dev-C++ 的界面几乎没怎么改过。

接下来我们可以新建一个程序试试。编译并运行，一气呵成……欸？怎么报错了？

![](https://cdn.luogu.com.cn/upload/image_hosting/971ttrjp.png)

原来这个版本的 **g++** 并不支持万能头，我们所熟知的万能头要到 **GCC 4.4.0** 及以上版本才被加入。改用标准输入输出流，再试一下，你会发现只见到一个黑框框一闪而过。这个版本的 Dev-C++ 还不会自动在程序结束时暂停！手动加上 `system("pause");` 即可。

![](https://cdn.luogu.com.cn/upload/image_hosting/0zrkvdap.png)

# 浏览器

既然环境配置好了，回归标题，我们该如何在 Windows 98 上浏览洛谷并提交题目呢？要知道，Windows 98 上的浏览器大部分都已经过时，根本无法正确渲染洛谷这样的现代网页。不过不试试怎么知道呢？

## IE 5

![](https://cdn.luogu.com.cn/upload/image_hosting/h4q8vevl.png)

IE 5 作为古董中的古董，根本打不开洛谷网站。

## Opera

经过查询，Windows 98 上能够原生运行的最新浏览器应该是在 2009 年发布的 **Opera 9.64**，我们一样先下载，再复制到虚拟机内安装。

![](https://cdn.luogu.com.cn/upload/image_hosting/5dzh3lwn.png)

洛谷的界面出来了！但是……又没有完全出来，像是现在洛谷打开网页加载了一半的样子，什么都点不了。

## WRP

面对 Windows 98 上的古老浏览器，真的就没有办法了吗？经过我的不断搜索，发现了一个神奇的项目：[Web Rendering Proxy: Use vintage, historical, legacy browsers on modern web](https://github.com/tenox7/wrp/)。既然 Windows 98 上的浏览器渲染不了，那么直接用别的主机渲染总可以了吧！WRP 项目就是做这个的，它会在本机开一个端口作为服务器，然后在其它浏览器中访问这个端口，输入地址，它就会给出一张带点击区域的图片，然后就能像平时浏览网页一样操作。

![](https://cdn.luogu.com.cn/upload/image_hosting/8uj4vwtf.png)

由于它没有 Windows 版本，于是我选择了在宿主机中安装 WSL 来运行。

![](https://cdn.luogu.com.cn/upload/image_hosting/2dn0cfcp.png)

通过端口转发，成功在 Windows 98 的 IE 5 中访问到了洛谷的题目。WRP 中可以直接点击画面，上面的控件从右往左分别是 Rt 换行、Bs 退格、K 输入文本，但是默认输入框区域很小，输入非常麻烦，尤其是代码。经过不懈努力，我还是敲出了一个 A+B。

![](https://cdn.luogu.com.cn/upload/image_hosting/09lyejnb.png)

提交成功！我宣布，全洛谷（或许是）第一个在 Windows 98 上编写和提交的记录出现了！