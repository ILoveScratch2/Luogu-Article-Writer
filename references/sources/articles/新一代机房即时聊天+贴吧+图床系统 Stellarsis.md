# 前言

如果您遇到种种问题，请在 Github 上创建 Issue。

由于学业压力，估计修的不快。

如果您感兴趣，欢迎 Fork Me！

## 特性

1. 美观的界面，可以切换主题，移动端友好，~~适合各种同学摸鱼使用~~。

2. 自认为还算健壮的后端，高可用。

3. 使用 `:` 呼出命令面板，快速操作。

4. 低延迟在线聊天，贴吧系统，图片上传，消息引用，BADGE 和名字颜色。

5. Markdown 与 ${\LaTeX}$ 良好支持，使用 highlight.js 实现代码高亮。

6. 功能强大的后端，甚至可以直接操作数据库。

7. 完善的权限划分。

## 图

![](https://cdn.luogu.com.cn/upload/image_hosting/ta3yok5q.png)

![](https://cdn.luogu.com.cn/upload/image_hosting/xgpdm9yo.png)

![](https://cdn.luogu.com.cn/upload/image_hosting/f3r16cuu.png)

![](https://cdn.luogu.com.cn/upload/image_hosting/mgz8rl6o.png)

![](https://cdn.luogu.com.cn/upload/image_hosting/cmmlqxuk.png)

管理面板比较丑。

## 项目地址

https://github.com/w1010tdev/stellarsis

## 如何部署？

```bash
# Download https://github.com/w1010tdev/Stellarsis/archive/refs/heads/main.zip
# PLS Use Python 3.10 - Python 3.12
# Or If U want to compile Pillow by your self
pip install -r requirements.txt
python app.py
```

linux 环境下请自行安装 venv。

如果需要长时间部署，请自行安装 tmux 等。

## 技术栈

后端：Python Flask。

实时通信：Flask-SocketIO。

数据库：SQLite。

前端：HTML/CSS/JavaScript。

实时消息：WebSocket （降级到轮询）。

可以的话在 Github 给一个 Star 吧。

## 致谢

感谢杭州深度求索人工智能基础技术研究有限公司提供的代码支持。

感谢 Alibaba Cloud (Singapore)  （因为国内站机房没手机登不上去）提供算力支持。

感谢微软（中国）有限公司的 Copilot 白嫖支持。但是并不感谢贵司把我的域名地下的邮箱全部标记为垃圾用户。

感谢 Anysphere 公司提供的 Cursor 白嫖支持。

感谢 OpenAI 公司、微软（中国）有限公司、 Anthropic 公司提供的大模型支持。


## 本项目和其他的在线聊天有什么区别？

- 强大的后端，防止可爱的同学打。

- 前端使用 Jinjia2 语法开发，非常适合拓展。

- Sqlite 数据库，在可爱的同学们的刷屏下依旧稳定。

- 原生 Flask 鉴权等，安全性高。

- ~~好多机房没有 Node 但是有 python。~~