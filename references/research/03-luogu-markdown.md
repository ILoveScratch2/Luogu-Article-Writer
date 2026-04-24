# Luogu Markdown 特殊语法调研

来源：本地语料（d:\Articles），100% 一手来源。

## 洛谷专属 Callout 块

### 格式
```
:::info[标题]{open}
内容
:::

:::success[标题]
内容（默认折叠）
:::

:::warning[标题]{open}
内容
:::

:::error[标题]{open}
内容
:::
```

### 使用规律
- `:::info`：大块代码、参考实现、附注、信息补充
  - 标题常用：`code`、`Code`、`Miller-Rabin`、`Source Code`、`解释`
- `:::success`：参考代码（AC 代码）
  - 标题常用：`参考代码`
- `:::warning`：重要警告、注意事项
- `:::error`：严重警告、禁止事项
- `{open}` 参数：展开显示（不加则默认折叠）

## 数学公式

### 内联公式
- `$...$`：行内公式
- `$\mathcal{O}(n\log n)$`、`$\pm 1$`、`$\bmod$`、`$\equiv$`

### 展示公式
- `$$...$$`：常用于较长公式、定理陈述

### 伪代码/算法格式
- 使用 `\begin{array}{l}...\end{array}` 格式排版伪代码
- 带有 `\textbf{if}`、`\textbf{return}`、`\textbf{repeat}` 等格式化关键字
- 算法用双列 `\begin{array}{ll}` 加行号

## 代码块

### 普通代码块
```
\`\`\`cpp
// 代码
\`\`\`
```

### 带行号
```
\`\`\`cpp line-numbers
// 代码
\`\`\`
```

### 常见语言标识
- `cpp`、`python`、`bash`、`powershell`、`plaintext`、`racket`、`csharp`、`javascript`、`wasm`

## 图片
- 图片几乎全部来自 `https://cdn.luogu.com.cn/upload/image_hosting/`
- 格式：`![描述](URL)`，描述可省略
- 有时带 `?x-oss-process=image/resize,...` 参数控制尺寸

## 分隔线
- `---`：Markdown 水平线，用于段落间逻辑切换
- 有时连续多个 `---` 分隔多个独立思路段

## 链接
- 洛谷题目：`[Pxxxxx 题目名](https://www.luogu.com.cn/problem/Pxxxxx)`
- 洛谷文章：`[文章名](https://www.luogu.com.cn/article/xxxxx)`
- 外部链接：直接用 `<url>` 或 `[显示名](url)`

## 文章内跳转（引用来源）
- 常在开头引用触发来源
- 格式：`来源：[文章名](链接)` 或直接链接

## 打印分页符（Article2PDF 专用）
- `===pagebreak===`：在新行单独使用，PDF 打印时分页

## 删除线
- `~~文字~~`：用于标注"已废弃"、幽默删除，如 `~~无可救药~~`
