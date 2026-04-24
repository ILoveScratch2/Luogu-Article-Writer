::::info
**本文有部分内容引用了 [OI Wiki](https://oi-wiki.org)，遵照原作者的要求，本文使用 CC-BY-NC-SA 2.0 协议共享**[^1]
::::

[^1]: [OI Wiki](https://oi-wiki.org): 提供 GCC 和 Git 安装引导

## 开始

在开始配置环境之前，请检查你的系统。

- 如果你使用 **Windows**，请在接下来的步骤中使用 Windows 部分。
- 如果你使用 **macOS**/**Linux**，请在接下来的步骤中使用 **\*nix** 部分。

请注意，如果你的机器上装有自动还原，你可能需要使用便携版的软件，因为你无法在某些步骤正常重启。

## 编辑器或 IDE

你可以按照你的喜好配置编辑器或 IDE，以下是一些常用编辑器和 IDE：

| 名称                                | 简介                         | Windows 兼容性        | macOS 兼容性               | Linux 兼容性                 | 许可状态                              |
| ----------------------------------- | ---------------------------- | --------------------- | -------------------------- | ---------------------------- | ------------------------------------- |
| VS Code                             | Microsoft 开发的现代编辑器   | Windows 10/11         | macOS Big Sur 及以上 (11+) | 基于 Xorg/Wayland 的桌面环境 | 开源，但包含闭源组件                  |
| Vim                                 | 终端编辑器                   | Any                   | Any，Xcode CL Tools 自带   | tty 或桌面环境               | 开源                                  |
| Emacs                               | 终端编辑器，多修饰键式快捷键 | ^                     | Any                        | ^                            | ^                                     |
| Dev-C++                             | 用于 C/C++ 开发的专用 IDE    | Windows 7/8/8.1/10/11 | 不兼容                     | <                            | 开源，多版本                          |
| JetBrains IDEA 或其他 IDEA 系列 IDE | 为特定编程语言开发的 IDE     | Windows 10/11         | macOS Ventura 及以上 (13+) | 基于 Xorg/Wayland 的桌面环境 | 有些 IDE 具有开源的社区版，专业版收费 |
| Sublime Text                        | 可扩展编辑器，现代 UI        | Windows 7             | macOS X 及以上 (10+)       | ^                            | 收费，但提供永久试用                  |

:::warning[关于 Notepad++]
Notepad++ 因为一些原因，现在的版本非常复杂（Np++/Np--/...），不建议使用，建议换用 Sublime Text 试用版等。
:::

### VS Code

VS Code 分为官方版和社区版（VSCodium 等），官方版提供完整的扩展和功能，社区版只能使用开源扩展，但移除了遥测和自动更新。

使用 [交互式安装](https://icc.gt.tc/vscode)

可以使用此安装器来安装 VS Code，此向导包含随后的配置。

或者在 [此链接](http://code.visualstudio.com) 下载官方版。

如果使用 *nix，建议使用包管理器，比如 Arch Pacman：（这个安装的是社区版 Code - OSS）

```bash
pacman -S vscode
```

安装后下载需要的本地化和语言扩展（如 C/C++，社区版可能需要寻找替代）。

如果你安装了本地化语言包，请按下 `Ctrl-Shift-P` 打开命令面板，然后输入 `display`，找到 `Configue Display Language` 来更改语言。

具体的 VS Code 参见：[VS Code 配置教程](https://www.luogu.com.cn/article/a594wn3y)

### Vim

如果你使用 *nix，那么在安装时可能自带了 Vim，macOS 的 Xcode Developer Tools 也自带 Vim。

在 macOS 下使用以下命令安装 XCLT：

```bash
xcode-select --install
```

在终端输入 `vim [filename]` 打开，指定 filename 来打开文件。

Vim 有三种模式，分别为：

- **命令模式**：打开文件后的默认模式，按下的字母键会识别为命令
- **编辑模式**：编辑文件的模式，按下 Esc 返回命令模式，以下是模式说明
  - **插入**：命令 `i` 进入，底栏 `[INSERT]`，正常编辑文本 
  - **替换**：命令 `r` 进入，底栏 `[REPLACE]`，光标下的文本会被替换 
  - **可视**：在命令模式选择文本，底栏 `[VISUAL]`，用于复制等
- **行末模式**：命令 `:` 进入，输入文件命令和其他复杂指令

打开文件后可以使用命令 `h/j/k/l` 用作方向键来导航，使用 `:{number}` 来跳转，如 `:3`。

使用 `:w` 来保存文件，`:q` 关闭编辑器，使用 `!` 后缀来强制执行。

e.g. `:wq` 保存并退出，`:q!` 不询问是否保存就退出

### Emacs

Emacs 的发行版配置比较复杂，已经有大量的介绍，在此不多介绍，参见 [litjohn](https://www.luogu.com.cn/user/537934) 的 https://www.luogu.com.cn/article/v1ecwprt。

### Dev-C++

Dev-C++ 作为 C/C++ IDE，会自带一个 MinGW/TDM-GCC 语言环境，不用额外安装。

进入后可以使用 `F9` 编译，`F10` 运行，`F11` 编译并运行。

如果你想要修改编译选项，可以进入 `工具 -> 编译选项`，以下是一些常用功能的路径：

- 优化：代码生成：优化级别 (-Ox)，如 Med (O2)
- 语言标准：代码生成：语言标准 (-std)，如 C++11，ISO 开头的是不带有 GNU 扩展的版本，此内容会在 GCC 编译器一节说明
- 警告：代码警告：显示最多警告信息 (-Wall)
- 调试信息：连接器：产生调试信息

如果你需要修改语法高亮，进入 `工具 -> 编辑器选项 -> 语法` 进行更改。

### JetBrains IDE

JetBrains 需要新建项目才能工作，选择 `新建项目`，按照提示新建，此 IDE 的引导较为完善，不多介绍。

详情参见 [ARIS1_0](https://www.luogu.com.cn/user/846661) 的 https://www.luogu.com.cn/article/2xtkowsn

### Sublime Text

Sublime Text 可以安装扩展管理器，选择 `Tools -> Install Package Manager...` 安装，然后可以安装语言扩展，使用 `Tools -> Build` 来运行代码。

详情参见 [fanjiayu666](https://www.luogu.com.cn/user/1251774) 的 https://www.luogu.com.cn/article/9sa95w8v

## 语言支持（编译器/解释器）

如果你使用 **编译型语言**，如 C++，你需要安装编译器，每次运行代码需要编译成可执行文件。

如果你使用 **解释型语言**，如 Python，你需要安装解释器，然后直接用解释器运行代码。

### C/C++

首先，请在终端输入这行命令以检查编译器状态：

```bash
gcc --version
```

如果没有安装，请 [下载 GCC](https://oi-wiki.org/tools/compiler/#gcc) 或者使用包管理安装。

以下所有的语言都可以（并在 *nix 下推荐）使用包管理器，如下：

```bash
pacman -S gcc
```

或者使用 base-devel：

```bash
pacman -S base-devel
```

如果使用 Windows，你可能需要将 GCC 的路径加入 Path。

### Python

进入 [Python 官网](http://python.org)，点击 `Download`，下载 `Installer` 版本并安装。

如果使用 Windows，请在安装时勾选 `将 PATH 添加到环境变量`。

在 Python 中，可以使用 pip 安装第三方库，如果提示未找到，使用 `/path/to/python -m pip`。（替换为你的实际 Python 路径）

### Java

进入 [JDK 下载](LINKNOTFILLED)，下载 JDK 并安装。

你需要配置 Path 和 `JAVA_HOME` 环境变量来正确读取 Java 安装。

注意，Java 是**半编译型语言**，开发包 JDK 和运行器 JRE 是子集关系（JDK 包含 JRE），JDK 在包管理器中的名称为 `OpenJDK`，直接安装 `java` 包为 JRE，即运行 jar 文件的工具。

### Node.js

进入 [Node.js 官网](http://nodejs.org)，找到安装脚本，复制到终端即可，这一般会安装 NVM。

NVM 是新版 Node 推荐的版本管理器，可以使用如下命令安装 Node：

```bash
nvm install lts # 最新版
nvm install lts/keypton # 某一个 LTS 版本
nvm install 22.22.0 # 版本号
```

如果安装了多个 Node，使用如下命令来切换和修改默认环境：

```bash
nvm ls # 列出环境，绿色和紫色的是已安装的版本
nvm use lts # 使用最新版，命名方式同上
nvm alias default lts # 默认使用最新
```

除了 `default` 别名，你还可以使用其他别名，如：

```bash
nvm alias electron1 22.22.0
nvm use electron1
```

## 实用工具

### VS Code 扩展：CPH（仅官方发行版的 VS Code）

CPH 可以在 VS Code 中一键测试带有时空限制的 OI 题目，并给出结果。

点击左侧活动栏的“扩展”，搜索 `CPH`，找到 `Competitive Programming Helper`，点击 `安装`

**注意安装官方版本，CPH 有大量的第三方适配版，日常只需要官方版就可以**

### 浏览器插件：Competitive Companion

这个插件用来拉取样例数据，配合 cph 使用。

如果使用 Chromium 浏览器（Chrome, Edge, Opera, etc.）：打开 [扩展商店](https://crxsoso.com)，搜索 `Competitive`，安装到浏览器。

如果使用 Firefpx 浏览器（Firefox, Zen, Tor, etc.）：打开“附加组件”页面，搜索 `Competitive`，安装到浏览器。

:::info[如果你使用新版本的 Chromium 浏览器]{open}
在新版的浏览器中，安装可能无法自动进行，打开扩展页面，打开  `开发者模式`，此时在下载目录下将 `crx` 文件拖入即可
:::

在 OJ 中找到一道题，点击扩展栏中绿色加号图标，这会在 VS Code 的当前文件夹创建一个与题目同名的文件。

这时左侧会显示测例，点击 `Run All` 评测。

### 用户脚本：AtCoder Better! / Codeforces Better!

这是一个可以增强 AtCoder / Codeforces 体验的脚本，提供深色模式，翻译和其他功能。

首先，安装一个脚本管理器，可以使用**篡改猴**或**脚本猫**，步骤同上。

然后搜索 `Greasy Fork` 或者 `Greasy Fork 镜像`，安装对应脚本即可。