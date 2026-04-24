复制洛谷部分页面 Markdown 源代码。

## 使用

点击右上角按钮复制源代码到剪贴板。

特别地，比赛、训练页面按钮位于比赛标题下方。

## 安装

推荐使用 cos 方式进行安装，greasyfork 需要你有稳定的国际网络连接。

- [GitHub Raw](https://github.com/LYkcul/copy-markdown/raw/refs/heads/main/copy_markdown.user.js)

- [TencentCloud COS](https://script-1309350268.cos.ap-beijing.myqcloud.com/copy_markdown.user.js)

- [greasyfork](https://greasyfork.org/zh-CN/scripts/545557-copy-markdown)

## 原理

~~容易发现。洛谷专栏获取到的数据以 JSON 格式存在 `script[id="lentille-context"]` 标签内，我们只需要解析该元素即可。~~

添加 `x-lentille-request: content-only` headers 即可获取 JSON 格式的返回数据。

对于比赛、训练和个人主页，其 Markdown 源代码分别位于 `_feInstance.currentData.contest.description`、`_feInstance.currentData.training.description` 和 `_feInstance.currentData.user.introduction`。

## 更新

[更新日志](https://www.luogu.com.cn/article/h5yujhzu)

#### v1.2.0

- 支持比赛详情、训练详情、个人主页源代码复制。
- 重构部分代码。
- 修复与某 e**g 插件的冲突。

#### v1.1.2

add headers for fetching clean JSON data

#### v1.1.1

fix bugs.

#### v1.1.0

- 修复部分场景下因路径变化导致的 btn 显示问题。
- 添加成功/失败状态显示。

## TODO

- 支持讨论区、工单等页面。
- 更优雅的轮询方式。

![](https://cdn.luogu.com.cn/upload/image_hosting/6lri2otn.png)

![](https://cdn.luogu.com.cn/upload/image_hosting/9kd73q6n.png)