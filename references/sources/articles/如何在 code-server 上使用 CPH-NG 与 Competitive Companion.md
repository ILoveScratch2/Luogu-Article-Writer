### 前言

**本文使用环境：本机 Windows 11 专业版，code-server 服务器 Ubuntu 20.04.6 LTS。**

作为 OIER，我经常会切换电脑，用移动盘存储携带不方便，正好手里有一台 Ubuntu 的服务器，我就用上了 code-server，web 版 VScode。[如何安装 code-server](https://blog.csdn.net/qq_35534279/article/details/138277885)

最开始我用的也是 CPH，但是发现 CPH 还要手动复制样例，大数据复制很卡，确实体验不好，在网上一翻，找到了 [CPH-NG](https://www.luogu.com.cn/article/8ix5zp4w)，有更强大的功能，可以通过 [Competitive Companion](https://github.com/jmerle/competitive-companion) 导入数据。[Chrome 安装](https://chromewebstore.google.com/detail/competitive-companion/cjnmckjndlpiamhfimnnjmnckgghkjbl) [FireFox 安装](https://addons.mozilla.org/en-US/firefox/addon/competitive-companion/) （官方没有给出 Edge 的安装链接）

### 本地操作

只不过 [Competitive Companion](https://github.com/jmerle/competitive-companion) 插件似乎只能导入到本地 VScode，去看了一下实现原理，实际上是通过端口发送题目信息的 `json` 到本地来实现快速导入，于是我们想到了端口转发。这里我们以 `21257` 作为示例。

这就需要本地有一个转发程序了。

我们首先需要在浏览器中设置 [Competitive Companion](https://github.com/jmerle/competitive-companion) 的转发端口，点击扩展右侧的三个点，点击选项，`Custom ports` 那里输入你自定的端口，只不过一定要先看一看这个端口是否有被占用。

在本机打开 PowerShell，输入：
```bat
netstat -ano | findstr 21257
```
如果显示 `>>` 那就再按一次回车。
看显示内容，如果显示类似：
```
TCP    0.0.0.0:21257          0.0.0.0:0              LISTENING       [PID（一串数字）]
```
那就说明当前的端口有程序正在占用，如果确定不需要该程序，可以运行下面的代码关闭进程：
```bat
taskkill /PID [PID（刚刚那串数字）] /F
```
如果不确定该程序是否需要，建议切换端口。

确定好端口之后，我们要在本地创建一个端口转发的脚本，这里以 c++ 为例，编译器 g++。

如果本机和 code-server 服务器在同一个内网中，你需要获取到 code-server 的内网 ip，可以在服务器终端输入：

```bash
ip addr
```
会看到类似：
```bash
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.105/24 brd 192.168.1.255 scope global eth0
```
的内容，其中 `192.168.1.105` 服务器的内网 IP。

在你的转发脚本中定义：
```cpp
#define TARGET_IP "server-ip"
#define TARGET_PORT POST // 你需要的端口
```

并定义转发函数：
```cpp
void forward_json(const std::string& json) {
    SOCKET s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);

    sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(TARGET_PORT);
    addr.sin_addr.s_addr = inet_addr(TARGET_IP);
    // 如果这里报错，可以将这一行替换为下面的代码
    // inet_pton(AF_INET, TARGET_IP, &addr.sin_addr);

    if (connect(s, (sockaddr*)&addr, sizeof(addr)) != 0) {
        std::cerr << "Forward failed\n";
        closesocket(s);
        return;
    }

    std::string req =
        "POST / HTTP/1.1\r\n"
        "Host: " TARGET_IP "\r\n"
        "Content-Type: application/json\r\n"
        "Content-Length: " + std::to_string(json.size()) + "\r\n\r\n" +
        json;

    send(s, req.c_str(), req.size(), 0);
    closesocket(s);

    std::cout << "Forwarded to " TARGET_IP << ":" << TARGET_PORT << "\n";
}
```

如果本机和 code-server 服务器不在同一个内网中，你需要服务器的外网 IP（既然能用上 code-server，相信还是有外网 ip 的吧，或者 cloudflared 隧道转发也可以），我这里用域名代替了。

要在脚本中定义：
```cpp
#define TARGET_HOST "your server ip or domain"
#define TARGET_PORT "POST" // 注意这里的两项都需要打引号
```

并添加函数：
```cpp
void forward_json(const std::string& json) {
    addrinfo hints{}, *res = nullptr;

    hints.ai_family = AF_INET;        // IPv4
    hints.ai_socktype = SOCK_STREAM;  // TCP

    int ret = getaddrinfo(
        TARGET_HOST,
        TARGET_PORT,
        &hints,
        &res
    );

    if (ret != 0 || !res) {
        std::cerr << "DNS resolve failed: " << gai_strerrorA(ret) << "\n";
        return;
    }

    SOCKET s = socket(
        res->ai_family,
        res->ai_socktype,
        res->ai_protocol
    );

    if (connect(s, res->ai_addr, (int)res->ai_addrlen) != 0) {
        std::cerr << "Connect failed\n";
        freeaddrinfo(res);
        closesocket(s);
        return;
    }

    std::string req =
        "POST / HTTP/1.1\r\n"
        "Host: " TARGET_HOST "\r\n"
        "Content-Type: application/json\r\n"
        "Content-Length: " + std::to_string(json.size()) + "\r\n\r\n" +
        json;

    send(s, req.c_str(), (int)req.size(), 0);

    freeaddrinfo(res);
    closesocket(s);

    std::cout << "Forwarded to " TARGET_HOST ":" << TARGET_PORT << "\n";
}
```

接下来，无论使用了内网 ip 还是公网 ip（域名），都是一样的了，引入头文件和 lib 库。

```cpp
#include <winsock2.h>
#include <ws2tcpip.h>
#include <iostream>
#include <string>
#pragma comment(lib, "ws2_32.lib")
```

主函数：
```cpp
int main() {
    WSADATA wsa;
    WSAStartup(MAKEWORD(2, 2), &wsa);

    SOCKET server = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);

    sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(PORT);
    addr.sin_addr.s_addr = INADDR_ANY;

    bind(server, (sockaddr*)&addr, sizeof(addr));
    listen(server, 5);

    std::cout << "Listening on " << TARGET_POST << "...\n";

    while (true) {
        SOCKET client = accept(server, nullptr, nullptr);
        std::cout << "Client connected\n";

        std::string data;
        char buf[4096];
        int len;

        while ((len = recv(client, buf, sizeof(buf), 0)) > 0) {
            data.append(buf, len);
        }

        // 提取 HTTP Body（JSON）
        size_t pos = data.find("\r\n\r\n");
        if (pos != std::string::npos) {
            std::string json = data.substr(pos + 4);
            std::cout << "===== JSON =====\n";
            std::cout << json << "\n";
            std::cout << "===============\n";

            forward_json(json);  
        }


        // 必须回应 200，否则插件会重试
        const char* ok =
            "HTTP/1.1 200 OK\r\n"
            "Content-Length: 0\r\n\r\n";
        send(client, ok, strlen(ok), 0);

        closesocket(client);
    }

    WSACleanup();
    return 0;
}
```

注意：需要添加 `-lws2_32` 编译参数。

如果编译运行后显示 `Listening on [POST]...`，那么说明前面的步骤成功了。

### 本地测试

![](https://cdn.luogu.com.cn/upload/image_hosting/zee62fr4.png)

打开一道题目，在扩展列表中点击 Competitive Companion，等待页面蓝色加载条完成，查看本地运行脚本的终端，如果出现类似：
```json
===== JSON =====
{"name":"A - XXX","group":"Virtual Judge - XXX性","url":"https://vjudge.net/contest/XXX#problem/A","interactive":false,"memoryLimit":1024,"timeLimit":1000,"tests":[{"input":"5\n5 1 2 3 4\n0\n6\n2 1 3\n5 4 6 2\n0\n0\n","output":"1\n2\n"}],"testType":"single","input":{"type":"stdin"},"output":{"type":"stdout"},"languages":{"java":{"mainClass":"Main","taskClass":"XXX"}},"batch":{"id":"03112856-30ee-4e26-XXX-1721f38320fb","size":1}}
===============
```
格式的内容，说明我们的脚本可以成功接收消息了。

如果运行终端继续显示
```bat
Forwarded to [HOST]:[POST]
```
就说明成功转发了。

此时本地操作已经完成，测试成功。

### 服务器操作

服务器操作就没那么复杂了，只不过还是需要先判断是否端口被占用。（这里服务器使用了 Ubuntu，如果是 Windows，参见上面本地操作中端口占用情况的部分）

输入：
```bash
lsof -i:[POST]
```

如果有一行类似：
```bash
COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
node    [PID] USER   34u  IPv6 XXXXX      0t0  TCP *:[POST] (LISTEN)
```

不用杀，一般是 CPH-NG 插件的监听，如果有其他，建议关闭（`kill -9 [PID]`）或者换端口。

接下来在 code-server 中找到 `Cph-ng › Companion: Listen Port` 这项参数，将端口填成你转换到服务器的那个端口，最好重启一下插件（说一个方便的方法，把上面 `lsof -i` 命令拿到的 CPH-NG 的进程 `PID` 杀了，code-server 会自动重启插件），保证本地的转发一直在线，便可以使用啦。

### 后记

实际上 Competitive Companion 直接自动创建的文件不一定在我们想要的位置，文件名也不会因为 Contest 而改变，最好的办法是自定义（既然我们都拿到 json 了），只不过篇幅问题，而且改内容不在本文主要内容范围内，敬请期待下一篇博客。

使用过程中如遇问题，欢迎邮件联系 [1000ttank@gmail.com](mailto:1000ttank@gmail.com)。
**不一定每封都能回复，但会尽量看**。