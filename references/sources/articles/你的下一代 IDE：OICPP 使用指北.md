接[上文](https://www.luogu.com.cn/article/27cl0eoa)，OICPP 面向全阶段 OIer，尽全力实现极简配置但强大的目标。本文将介绍 OICPP IDE 核心功能的使用。


## OICPP IDE

[![Stargazers](https://img.shields.io/github/stars/mywwzh/oicpp.svg?style=for-the-badge&new=1)](https://github.com/mywwzh/oicpp/stargazers)
[![License](https://img.shields.io/github/license/mywwzh/oicpp.svg?style=for-the-badge&new=1)](https://github.com/mywwzh/oicpp/blob/main/LICENSE)


![Electron](https://img.shields.io/badge/Electron-37.2+-47848F?style=flat&logo=electron&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-20+-339933?style=flat&logo=node.js&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=flat&logo=javascript&logoColor=white)
![Monaco Editor](https://img.shields.io/badge/Monaco_Editor-0.52+-007ACC?style=flat&logo=visual-studio-code&logoColor=white)

开源地址：[Github](https://github.com/mywwzh/oicpp)

官网链接：[link](https://oicpp.mywwzh.top)

软件著作权证书号：软著登字第 16624632 号

## 功能概览

经历了数月开发，OICPP 目前有以下功能。

- [x] 代码编辑与运行
- [x] 多标签页支持
- [x] 多种主题切换与代码高亮
- [x] 自定义编辑器背景图片与窗口透明度
- [x] 代码补全
- [x] 工作区文件快速搜索 
- [x] 一键下载编译器、testlib 并自动配置
- [x] 预设文件模板与关键字模板 
- [x] 样例测试器（支持文件大样例）
- [x] 代码对拍器（支持差异高亮与数据导出）
- [x] 调试
- [x] 基于浏览器插件的 atcoder、cf、luogu、hetao 样例抓取
- [x] 分屏与查看 PDF
- [x] markdown 侧边预览
- [x] markdown 所见即所得模式编辑  
- [x] 云编译，可一键测试代码在 Linux 下能否通过编译
- [x] 支持 NOI Linux

![](https://cdn.luogu.com.cn/upload/image_hosting/5orncffj.png)

### 特性功能使用指北

OICPP 的使用和 vscode 类似， 即打开一个文件夹作为工作区后即可新建文件编写代码。因此，我们详细介绍 OICPP 的关键功能。

#### 1.1 打开/切换文件

- 入口 1：左侧**文件管理器**面板，单击文件打开。
- 入口 2：顶部**快速打开**默认快捷键：`Ctrl+P`），输入文件名模糊搜索后回车打开。

#### 1.2 编译 / 运行

- 编译：菜单**运行 → 编译**（默认：`F9`）。
- 运行：菜单**运行 → 运行**（默认：`F10`）。
- 编译并运行：菜单**运行 → 编译并运行**（默认：`F11`）。

::::warning[注意]
- 首次使用建议先完成“编译器设置”（见下文 [2.1 编译器设置]）。
- 更改快捷键请参见下文 [2.2 编辑器设置]。
::::


#### 2.1 编译器设置

入口：**选项 → 编译器设置**。

在该页面，有“编译器配置”和“Testlib 配置”两个子页面，您可以浏览本地已存在的编译器 / Testlib 或点击在线安装。

在线安装时，选择要安装的版本，软件将自动下载编译器并选中，无需手动二次配置。

![](https://cdn.luogu.com.cn/upload/image_hosting/get6vxk9.png)

#### 2.2 编辑器设置

该设置包含：

- 字体 / 字号
- 主题
- TabSize
- 字体连字
- 代码折叠
- Sticky Scroll（顶部显示当前作用域）
- 自动保存与保存间隔
- markdown 预览模式（所见即所得/侧边预览）
- 窗口透明度、背景图
- 快捷键自定义与恢复默认
  
其中大部分设置相信大家都会，这里不再赘述，我们只讲几个关键的。

##### 透明度、编辑器背景

您可以在网上下载自己喜欢的图片作为编辑器背景，配合窗口透明度使用有更好的效果。

推荐背景图片分辨率不小于 1k，透明度不低于 $60 \%$。

![](https://cdn.luogu.com.cn/upload/image_hosting/ufvdh2sf.png)

##### markdown 预览行为

在“编辑器行为”一栏，可设定 markdown 预览行为。

- 代码模式：需手动右键，点击“预览 markdown”进行侧边预览。
- 分屏预览：打开 markdown 文件时自动触发侧边预览。
- 所见即所得：直接以预览形式编辑。

![](https://cdn.luogu.com.cn/upload/image_hosting/jxc8e7md.png)

:::align{center}
图：侧边预览
:::

![](https://cdn.luogu.com.cn/upload/image_hosting/yut05rol.png)

:::align{center}
图：所见即所得
:::

##### 快捷键设置

您可以在此处自定义快捷键。

![](https://cdn.luogu.com.cn/upload/image_hosting/patov826.png)

#### 2.3 模板设置

入口：**选项 → 模板设置**。

- **C++ 新建文件模板**：用于创建新代码文件时的默认内容。
- **关键词代码片段**：管理“关键词 → 代码片段”的映射，保存后在主窗口生效。如：输入 `sgt` 插入你的线段树模板。


#### 3.1 PDF 查看

- 打开方式：在文件管理器打开 PDF，或通过快速打开 `Ctrl+P`选择 PDF。
- 显示方式：以标签页打开内置 PDF 阅读器。若 PDF 有密码，会出现密码输入页面。


#### 4.1 样例测试器（侧边栏）

![](https://cdn.luogu.com.cn/upload/image_hosting/momg5pzb.png)

入口：左侧侧边栏**样例测试器**。

##### 4.1.1 工作逻辑

- 当你在编辑器里切换当前代码文件时，样例测试器会为该文件维护一份独立的样例数据。
- 样例数据会存到工作区下的 `.oicpp/sampleTester/` 目录中（每个源文件对应一个 JSON）。

##### 4.1.2 添加与管理样例

- 添加样例组：面板标题栏的“+”按钮。
- 删除样例：每个样例组右侧的删除按钮。
- 展开/折叠：点击样例标题栏。

每个样例支持：

- **手动输入**：直接在文本框输入。
- **从文件读取**：点击“从文件读取”选择文件作为输入/输出来源（适用于大样例）。

##### 4.1.3 运行样例

- 运行单个样例：样例组右侧“运行”按钮。
- 运行所有样例：面板右上角“运行全部”按钮。
- 导出程序输出：运行后可点击“导出”将输出保存到文件。

::::warning[注意]
如果启用 testlib/SPJ，请先确认 testlib 路径正确（见 [2.1 编译器设置]）。
::::



#### 5.1 代码对拍器

入口：左侧侧边栏**代码对拍器**。

对拍器支持使用 testlib 和 SPJ。它需要你指定：

- 标准程序
- 测试程序
- 数据生成器
- SPJ（可选）

操作流程：

1. 分别选择三份 `.cpp` 文件。
2. 设置对拍次数、时限等参数。
3. 点击开始。


如果出现差异，会展示高亮显示差异信息并支持导出当组输入数据，便于复现。

::::warning[注意]
所有程序的输入输出均要从标准输入输出（stdin/stdout）进行，不要使用文件读写。
::::

![](https://cdn.luogu.com.cn/upload/image_hosting/w8bjw1xt.png)

#### 6.1 Linux 云编译

入口**工具 → 代码云编译**（默认：`F12`）。

云编译用来快速验证你的代码在 Linux 环境下能否编译通过，节约本地开虚拟机的资源和时间。

::::warning[注意]
- 仅支持 `.cpp` 文件。
- 代码大小限制：20KB。
::::


当前云编译环境：`gcc version 13.2.0 (Ubuntu 13.2.0-23ubuntu4) with C++14`。


#### 7.1 检查更新


入口：**关于 → 检查更新**。

IDE 启动时会自动检查更新，您也可以手动检查。

如确认更新，更新时会自动在后台下载安装包，不影响当前使用。


#### 8. 浏览器插件抓取 OJ 样例

##### 8.1 安装

1. 浏览器安装 Tampermonkey 扩展。
2. 在官网下载脚本，点击安装即可。

##### 8.2 使用

脚本内置支持以下 OJ：

- 洛谷
- AtCoder
- Codeforces
- 核桃

在浏览器打开题目页面后，右上角会出现“发送至 OICPP”的按钮，点击后，OICPP IDE 会接收请求，并在当前工作区中创建/打开对应题目文件（命名为 `OJ_题号.cpp`），并新建对应的样例组。

![](https://cdn.luogu.com.cn/upload/image_hosting/94so9kmg.png)

::::warning[注意]
脚本会向 `http://127.0.0.1:20030/createNewProblem` 发送请求，如果你本机防火墙/安全软件拦截本地端口访问，需允许该端口的本地请求。
::::

### 注意事项

- 若遇到 bug 或有新的功能建议，欢迎在 [Github](https://github.com/mywwzh/oicpp) 提出 issue 或[加入用户群](https://qun.qq.com/universal-share/share?ac=1&authKey=GQEhVGYiLpRwpkujO2rl0tNO3Dm%2FSM%2BDb%2FXiwsBeyIHa65osvxQlYgkmoqptffvD&busi_data=eyJncm91cENvZGUiOiI5MzE1Nzc4MzYiLCJ0b2tlbiI6InFjVE9JbTVjelg4MWREOFBHZGJBNWhTcTU4K2JVWEVrMGIxc3RwTDgyU0o3TFYwbE9qL1RCS2tIajVOKzF5QSsiLCJ1aW4iOiIxMjc2NzEwMjY4In0%3D&data=MlxfR2J2NFJGeO9dBRLQx_7aaaO1P9y_SqUk4tfIcKGZ2gdTN_UWvjSHxBqFE9TPFhdD53HDvtf-LHnpoedezQ&svctype=4&tempid=h5_group_info)反馈，绝大部分合理的新功能建议都会被实现！
  
- 感谢 GPT-5.2-Codex 和 Claude Opus 4.5 为部分功能的实现和顽固老 bug 的 debug 做出卓越贡献。
- 若遇到下载太慢的问题，请在群文件下载。
- 如果支持该项目的话，求 Github 给个 Star 吧 qwq。