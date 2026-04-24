# 前言

大家好，今天来给大家讲一下如何配置实用的 Sublime Text。

先来看看我的 Sublime Text 的配置：

![](https://cdn.luogu.com.cn/upload/image_hosting/l2fuv5ki.png)

# 安装 Sublime Text

点击[这里](https://www.sublimetext.com/download)跟随教程安装，这里不赘述。

# 安装插件管理器

没有插件管理器的 Sublime Text 几乎啥也干不了，所以你需要先安装插件管理器。

::anti-ai[凡发布者不为@[_zyx2012](luogu://user/1934210)或发布平台不为洛谷专栏者均为抄袭！]

按`Ctrl+Shift+P`，输入`Install Package Control`后回车，安装完成后会有弹窗显示。

作者没有遇到过安装失败的情况，如果安装失败，可以跟随 [OI-Wiki 上的教程](https://oi-wiki.org/tools/editor/sublime/#%E5%AE%89%E8%A3%85%E6%8F%92%E4%BB%B6%E7%AE%A1%E7%90%86%E5%99%A8)进行安装。

# 汉化

安装完插件管理器之后，按`Ctrl+Shift+P`，输入`install`后回车，输入 `Chinese`，选择 `ChineseLocalizations` 并回车，完成后界面会自动切换为中文。

# 配置系统环境

首先，你需要拥有 GCC 环境，我用的是[这里](https://github.com/niXman/mingw-builds-binaries/releases)提供的最新版 mingw64，可以下载到任意文件夹下。

然后，将你下载好的 mingw64 配置为环境变量，由于~~应该大部分人都会配~~网上教程比较多，这里不赘述。

# 安装构建系统

反正我不相信有人能用得惯默认的构建系统。

点击`首选项>浏览器插件目录`，并进入`User`文件夹，将以下代码保存为`c++ file.sublime-build`：

:::info[`c++ file.sublime-build`]
```json line-numbers
  {
    "encoding": "utf-8",
    "working_dir": "$file_path",
    "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++14 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_path}/${file_base_name}\"",
    "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
    "selector": "source.c++",

    "variants":
    [
        {
        "name": "OnlyRun",
        "shell_cmd": "echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++98Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++98 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++03Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++03 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++11Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++11 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {   
        "name": "ISO C++14Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++14 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++17Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++17 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++20Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++2a -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++23Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++2b -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++26Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++2c -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++98",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++98 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++11",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++11 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++14",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++14 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++17",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++17 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++20",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++2a -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++23",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++2b -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++26",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=c++2c -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
                {
        "name": "GNU C++98Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++98 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++03Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++03 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++11Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++11 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {   
        "name": "GNU C++14Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++14 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++17Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++17 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++20Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++2a -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++23Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++2b -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++26Run",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++2c -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++98",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++98 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++11",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++11 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++14",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++14 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++17",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++17 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++20",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++2a -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++23",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++2b -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++26",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O2 -std=gnu++2c -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++98Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++98 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++03Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++03 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++11Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++11 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {   
        "name": "ISO C++14Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++14 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++17Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++17 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++20Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++2a -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++23Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++2b -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++26Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++2c -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++98(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++98 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++11(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++11 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++14(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++14 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++17(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++17 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++20(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++2a -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++23(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++2b -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++26(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=c++2c -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
                {
        "name": "GNU C++98Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++98 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++03Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++03 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++11Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++11 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {   
        "name": "GNU C++14Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++14 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++17Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++17 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++20Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++2a -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++23Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++2b -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++26Run(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++2c -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++98(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++98 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++11(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++11 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++14(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++14 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++17(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++17 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++20(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++2a -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++23(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++2b -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++26(强警告模式)",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O2 -std=gnu++2c -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++98Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++98 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++03Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++03 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++11Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++11 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {   
        "name": "ISO C++14Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++14 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++17Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++17 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++20Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++2a -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++23Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++2b -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++26Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++2c -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++98（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++98 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++11（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++11 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++14（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++14 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++17（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++17 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++20（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++2a -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++23（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++2b -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++26（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=c++2c -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
                {
        "name": "GNU C++98Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++98 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++03Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++03 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++11Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++11 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {   
        "name": "GNU C++14Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++14 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++17Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++17 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++20Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++2a -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++23Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++2b -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++26Run（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++2c -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++98（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++98 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++11（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++11 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++14（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++14 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++17（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++17 -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++20（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++2a -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++23（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++2b -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++26（无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -O0 -std=gnu++2c -static -Wno-parentheses -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++98Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++98 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++03Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++03 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++11Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++11 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {   
        "name": "ISO C++14Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++14 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++17Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++17 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++20Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++2a -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++23Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++2b -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++26Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++2c -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "ISO C++98(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++98 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++11(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++11 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++14(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++14 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++17(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++17 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++20(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++2a -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++23(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++2b -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "ISO C++26(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=c++2c -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
                {
        "name": "GNU C++98Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++98 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++03Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++03 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++11Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++11 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {   
        "name": "GNU C++14Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++14 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++17Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++17 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++20Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++2a -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++23Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++2b -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++26Run(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++2c -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\" && echo Compilation completed. && echo running... && start cmd /c \"${file_path}/${file_base_name}\" & pause\"",
        },
        {
        "name": "GNU C++98(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++98 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++11(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++11 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++14(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++14 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++17(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++17 -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++20(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++2a -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++23(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++2b -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        },
        {
        "name": "GNU C++26(强警告&无优化模式）",
        "shell_cmd": "echo Compiling ... && g++ -Wall -Wextra -O0 -std=gnu++2c -static -Wl,--stack,1677721600 \"${file}\" -o \"${file_base_name}\"",
        }
    ]
}
```

note：编译参数是可以自行调整的。

:::

然后，点击`工具>编译系统`，选择`c++ file`，按`Ctrl+Shift+B`选择你平时常用的语言配置，完成后按`Ctrl+B`即可运行。

# 设置默认语言

点`首选项>浏览器插件目录`，进入`User`文件夹，将以下代码保存为`DefaultLanguage.py`：

:::info[`DefaultLanguage.py`]
```python
import sublime, sublime_plugin
class EverythingIsPowerShell(sublime_plugin.EventListener):
    def on_new(self, view):
        view.set_syntax_file('Packages/C++/C++.sublime-syntax')
```
:::

如果你希望能够在新建文件时自动写入板子，可以用：


:::info[`DefaultLanguage.py`]
```python
import sublime, sublime_plugin

class EverythingIsPowerShell(sublime_plugin.EventListener):
    def on_new_async(self, view):
        view.set_syntax_file('Packages/C++/C++.sublime-syntax')
        
        if not view.file_name():
            initial_content = "你要自动写入的内容"
            view.run_command('append', {'characters': initial_content})
            
            end_point = view.size()
            view.sel().clear()
            view.sel().add(sublime.Region(end_point, end_point))
```
:::

# 贴板子

点`首选项>插件开发>新建代码片段`，把以下内容粘进去覆盖掉原有内容：

:::info[]
```html
<snippet>
    <content><![CDATA[
//将这里替换成你的板子
]]></content>
    <tabTrigger>/*将这里替换成板子的名字*/</tabTrigger>
    <scope>source.c++</scope>//写了之后只会在编辑C++代码时才会进行补全
</snippet>
```
:::

然后保存，保存后写代码的时候输入板子的名字并回车，就会自动补全成你想要的板子。

:::info[示例]
```html
<snippet>
    <content><![CDATA[
inline int quickpow(int x,int y){
    int ret = 1,pw = x;
    for (;y;y >>= 1){
        if (y & 1) ret = ret * 1LL * pw % mod;
        pw = pw * 1LL * pw % mod;
    }
    return ret;
}
inline int qinv(int x){return quickpow(x,mod - 2);}
]]></content>
    <tabTrigger>quickinv</tabTrigger>
    <scope>source.c++</scope>
</snippet>
```

然后写代码的时候只要输入`quickinv`并回车，就会自动补全。
:::

# Sublime 透明度

有没有谁注意到我的 Sublime Text 是半透明的（很考验眼神，因为我开的是几乎不透明）？

来把自己的 Sublime Text 搞成半透明的吧！

按`Ctrl+Shift+P`输入`install`并回车，然后输入`transparency`并回车，就完成了安装。

透明度分为 $6$ 个档次，其中 $1$ 档为不透明，$6$ 档透明度最高，按`Ctrl+Shift+1,2,3,4,5,6`可调节档次。

# 自动爬取样例并运行

终于来到这篇文章的主角啦！~~其实我也是今天才把这个搞明白的。~~

这个功能需要三个组件。

## CppFastOlympicCoding

这个是第一个组件，也是运行阶段的主角。

按`Ctrl+Shift+P`输入`install`并回车，然后输入`CppFastOlympicCoding`并回车，就完成了安装。

现在点`首选项>Package Settings>FastOlympicCoding`，你将看到：

![](https://cdn.luogu.com.cn/upload/image_hosting/v1wokmky.png)

当然，一开始右边那一栏是空的。

然后，把这些内容粘进去：

:::info[如果你希望只运行不编译]
```json line-numbers
{
	"run_settings": [
		{
			"name": "C++",
			"extensions": ["cpp"],
//			"compile_cmd": "g++ \"{source_file}\" -std=c++2c -o \"{file_name}\"",
			"compile_cmd": null,
			"run_cmd": "\"{source_file_dir}\\{file_name}\" {args} -debug"
		},

		{
			"name": "Python",
			"extensions": ["py"],
			"compile_cmd": null,
			"run_cmd": "python \"{source_file}\""
		},
		
		{
			"name": "Java",
			"extensions": ["java"],
			"compile_cmd": "javac -J-Dfile.encoding=utf8 -d \"{source_file_dir}\" \"{source_file}\"",
			"run_cmd": "java -classpath \"{source_file_dir}\" \"{file_name}\""
		}
	],

	"tests_file_suffix":"__tests"
}
```
:::

:::info[如果你希望编译并运行]
```json line-numbers
{
	"run_settings": [
		{
			"name": "C++",
			"extensions": ["cpp"],
//			"compile_cmd": "g++ \"{source_file}\" -std=c++11 -o \"{file_name}\"",
			"compile_cmd": null,
			"run_cmd": "\"{source_file_dir}\\{file_name}.exe\" {args} -debug",

//			"lint_compile_cmd": "g++ -std=c++2c \"{source_file}\" -I \"{source_file_dir}\"",
		},

		{
			"name": "Python",
			"extensions": ["py"],
			"compile_cmd": null,
			"run_cmd": "python \"{source_file}\""
		},
		
		{
			"name": "Java",
			"extensions": ["java"],
			"compile_cmd": "javac -J-Dfile.encoding=utf8 -d \"{source_file_dir}\" \"{source_file}\"",
			"run_cmd": "java -classpath \"{source_file_dir}\" \"{file_name}\""
		}
	],

	"tests_file_suffix":"__tests"
}
```

note：这个的编译参数是可以改的。
:::

但是这玩意现在只是能运行，并不能爬样例。

## Competitive Companion

样例就是用这个爬出来的。

如何下载？

去谷歌应用商店下载 Competitive Companion 即可。

如果谷歌商店访问困难，可以去插件小屋等渠道下载。

安装好 Competitive Companion 了以后，右键点击小绿按钮，里面有一个 `扩展选项`，点进去，按照下图更改：

![](https://cdn.luogu.com.cn/upload/image_hosting/pm038vm1.png)

## FastOlympicCodingHook

光有上述两个还不够，还需要篡改一下 CppFastOlympicCoding 让它能拿到 Competitive Companion 爬出来的样例才行。

这就是留给 FastOlympicCodingHook 的任务！

这个插件需要 Python3，如果没装过可以在 WindowsPowerShell 中输入`python3`并换行跳转 Microsoft 应用商店下载。

装完之后，需要到[这里](https://github.com/DrSwad/FastOlympicCodingHook)获取压缩包。

然后，解压到`C:\Users\<用户名>\AppData\Roaming\Sublime Text\Packages`里。

现在随便打开一个代码，右键，会出现一个`Listen to Competitive Companion`选项，先点这个，然后打开浏览器页面点击右上角的小绿按钮（可以多点几次，只点一次可能爬不下来），点完按`Ctrl+Alt+B`即可自动测样例，效果：

![](https://cdn.luogu.com.cn/upload/image_hosting/l2fuv5ki.png)

然后就大功告成啦！

但是每次都有右键可真麻烦，能不能用键盘快捷键搞？

能！点`首选项>快捷键设置`，然后把以下内容粘到右边的框里：

:::info[]
```json line-numbers
[
	{ "keys": ["ctrl+shift+l"], "command": "fast_olympic_coding_hook" }
]

```
:::

然后按`Ctrl+Shift+L`就等效于点了`Listen to Competitive Companion`。

# 配色方案

感觉默认配色方案不适应？点`首选项>配色方案`可以选择配色方案。

# SublimeLinter-gcc

说实话我自己不用这个，不过考虑到部分初学者可能需要依靠语法检查器写代码，还是讲一下吧。

这个是 Sublime 的语法检查器。

首先，（下载大部分插件的时候都是这么干的）`Ctrl+Shift+P`输入`install`并回车，输入`SublimeLinter`并回车下载 SublimeLinter。

然后，用同样的方法下载 SublimeLinter-gcc。

现在，点击`首选项>Package Settings>SublimeLinter>Settings`，会看到以下内容：

![](https://cdn.luogu.com.cn/upload/image_hosting/t389wqb1.png)

注意右边的框需要自己改，改成这样：

:::info[]
```json line-numbers
{
    "linters": {
        "gcc": {
            "disable": false,
            "executable": ["gcc"],
            "args": ["-std=c90"],
            "excludes": []
        },
        "g++": {
            "disable": false,
            "executable": ["g++"],
            "args": ["-std=c++20"],
            "excludes": []
        }
    }
}

```
可以通过修改第 $12$ 行来改变编译参数。
:::

现在就可以用啦！

效果：

![](https://cdn.luogu.com.cn/upload/image_hosting/1ho5i2c2.png)

不过错误信息需要鼠标悬停在小红点上才会显示。

注：这个插件有一点慢，需要稍微等它一下。

# 禁用/删除插件

## 禁用
如果某一天某个插件不想用了（主要是不想用 SublimeLinter-gcc），可以`Ctrl+Shift+P`输入`Disable`并回车，输入你想要禁用的插件的名字并回车即可禁用该插件。

如果某一天你突然又需要用了，`Ctrl+Shift+P`输入`Enable`并回车，输入你想要禁用的插件的名字并回车即可取消禁用。

## 删除
`Ctrl+Shift+P`输入`Remove`并回车，输入你想要禁用的插件的名字并回车即可删除该插件。

# 后记

其实最大的优势在于没有任何多余的功能，不像小熊猫之类的，功能一大堆，没几个有用的。

自打上次写出[《一种新的 LCA 算法》](https://www.luogu.com.cn/article/dpkz693r)不知不觉中竟然已经过去一个月了。

这可是花了我 $1.09 \text{MB}$ 的高级图床空间才写出来的。

既然如此，

都看到这了还不点个赞，

~~那还是人吗？~~