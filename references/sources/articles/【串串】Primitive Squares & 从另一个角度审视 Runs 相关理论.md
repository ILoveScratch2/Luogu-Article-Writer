观前提示：

读者**需要掌握** [字符串基础](https://oiwiki.com/string/basic/)、[字符串哈希](https://oiwiki.com/string/hash/)、少量周期理论（[Periodicity Lemma](https://www.baidu.com/s?ch=3&ie=utf-8&wd=Periodicity%20Lemma)），以及若干复杂度分析能力。

如果读者只希望学习或者见识一下其它可能的做法，不需要与成熟理论进行对照，则不需要掌握任何 Lyndon 和 Runs 相关理论。

**update**

2026.04.20：在例题中给出了该算法的代码。

## 摘要

本文提出了一种**动态加入字符**，并求出后缀 **Primitive Squares**（本原重串）的算法。单次修改**均摊** $O(\log^2n)$。

同时，该算法可以较为简单地（？）在离线情况下以 $O(n\log n)$ 的复杂度求出所有 **Primitive Squares**（该复杂度已达理论上界）。

遗憾的是，使用该算法计算 Runs 的复杂度依然为 [$O(n\log n)$](https://www.luogu.com.cn/record/257207021)，不够完美。

且该算法和 Runs 类似，偏向科技，应用空间不是很大。

## 前言

我说摘要一定要写在最上面不然没人看了。

**如果这篇文章提到的算法已经被发明过，请将文章发给我，然后我将标题改为学习笔记。**

笔者行文尚不熟练，若有格式或逻辑问题欢迎指出；笔者的 Lyndon 和 Runs 理论较学习较浅，不确定 Runs 是否支持**动态在末尾添加字符**同时计算出所有后缀**Primitive Squares**（目前我不会，如果你会了欢迎交流）。

## 正文

### 约定与记号

关于字符串部分的基础详见 [OI Wiki 字符串](https://oiwiki.com/string/)，这里略提。

$abc$ 表示拼接 $a,b,c$ 三个字符串。

$a^n$ 表示 $n$ 个 $a$ 拼接在一起。

**若无特殊定义，下文的字符串 $s$ 都是从 $1$ 开始。**

$|s|$ 表示 $s$ 的长度，$s[i,j]$ 表示 $s$ 的一个子串，第一个字符的标号为 $i$，最后一个字符的标号为 $j$。

### 定义

+ **Squares**：能表示成 $x^2$ 的串，如  $\texttt{abab}$。
+ **Primitive Squares**：不能再拆的 Squares，即最小周期为自身长度的一半的字符串。

$\texttt{abab}$ 是 **Primitive Squares**，但 $\texttt{aaaa}$ 不是。

### 引理

#### Three Squares Lemma

+ 若对于一个字符串 $S$，有 $S[1,2a],S[1,2b],S[1,2c]$ 均为 **Primitive Squares**（$0<a<b<c\leq \dfrac{|S|}2$），则有 $a+b<c$。

证明留作一个练习。或者看看[这篇文章](https://www.cnblogs.com/zght/p/13443305.html)。笔者仅使用 Periodicity Lemma 完成过证明，但是太长了就不放上来了。

显然，以某个固定位置为结尾的 **Primitive Squares** 子串数量级为 $O(\log n)$。

---

#### Squares Judgment

给定所有以某个位置为结尾的 **Squares**，高效判断哪些是 **Primitive Squares**。

+ 以长度递增顺序判定，设上一个 **Primitive Squares** 长度为 $2x$，当前需要判断的 **Squares** 长度为 $2x'$。若上一个 **Primitive Squares** 往左的最远重复距离为 $k$，则当前 **Squares** 是 **Primitive Squares** 的充要条件为：$\lfloor \dfrac k2 \rfloor\times x<x'$。

往左的最远重复距离的定义为能扩展几个连续的完整循环节，如 $S=\texttt{aabababa}$，其 $T=S[5,8]$ 是一个 **Primitive Squares**，其对应的 $k=3$。

严谨地将 $k$ 定义出来是：

$$
k(l,r) = \max \left\{ t \ge 0 \;\middle|\; S[r - t \cdot \dfrac{r-l+1}2+1,\; r] \text{ 以 } \dfrac{r-l+1}2 \text{ 为周期} \right\}
$$

很遗憾这个名字是我自己起的，所以可能有点格格不入。

证明方式只需要用众所周知的的 Periodicity Lemma（周期引理）即可。

另外判定引理还有一个有用的扩展结论：

+ 对于某个字符串上任意两个长度均为 $2x$  的 **Squares**，若它们在原串上出现位置的交 $\geq x$，则它们一定**同时**是或者不是 **Primitive Squares**。

---

### 算法

考虑如何**在线地**更新出以 $i$ 为结尾的所有 **Primitive Squares**。

我们可以注意到，对于长度为 $2x$ 的 **Squares**，其一定经过恰好两个相邻的、下标是 $x$ 倍数的点 $kx-x$ 与 $kx$（$2\leq k\leq \dfrac{n}x$）。

此技巧在 [NOI2016 优秀的拆分](https://www.luogu.com.cn/problem/P1117)中亦有记载。

对于新加入的字符 $S_i$，我们枚举 $i$ 的因数 $x$，计算 $S[1,i-x]$ 与 $S[1,i]$ 的最长公共后缀 $d$。

+ 若 $d=0$，则接下来 $x$ 个位置一定不会出现长度为 $2x$ 的 **Squares**，直接不管。
+ 否则把 $x$ 挂到 $\max(i,i+x-d)$ 的位置。这是首个可能出现 **Squares** 的位置。

处理在 $i$ 处挂着的 $x$ 时，我们判断一下 $[i-2x+1,i]$ 是否是一个 **Squares**，并将其加入待判断序列。

把所有待判断序列中的长度升序排序，先判断是不是 **Squares**，再用 **Squares Judgment** 判定。不是 **Primitive Squares** 直接删去。

（实现精细可以忽略这一行）特别地，最后需要把满足 $i\equiv x-1\pmod x$ 的所有 **Primitive Squares** 移出待判断序列，它们已经过期了。

这样我们就可以用均摊 $O(\log^2n)$ 的复杂度在线求出了所有以 $i$ 为结尾的 **Primitive Squares**。总复杂度 $O(n\log^2 n)$。

注意：vector 虽然不处于常数瓶颈，但实现时最好手写。

我在例题中提供了该算法的一种实现。

### 例题

给定字符串 $S$，将其划分为若干非空连续子串 $s_1,s_2,\ldots,s_k$，要求：
- 每个 $s_i$ 是非循环的（不存在 $k\geq 2$ 和字符串 $t$ 使得 $s_i=t^k$）；
- 相邻子串不同（$s_i\neq s_{i+1}$）。

求满足条件的划分总数，对 $998244353$ 取模。

需要支持**动态在末尾追加字符**并**在线**输出。

即[【串串划分】](https://qoj.ac/contest/2398/problem/13629) 的加强版。

#### 转化

记字符串 $s$ 的最小循环节长度为 ${\rm mnper}(s)$，最大循环次数为 ${\rm mxrep}(s)$。注意这里有 ${\rm mnper}(s){\rm mxrep}(s)=|s|$。

设 $f_i$ 表示 $S[1,i]$ 的划分数。

**略去部分容斥和数学推导**，可得转移式子：

$$
f_i=(\sum_{j=1}^if_{j-1})-2(\sum_{j=1}^if_{j-1}[{\rm mxrep(s[j,i])}\equiv 0\pmod 2])
$$

注意到最后的转移式子的右半部分，只有 **Primitive Squares** 才可能有贡献，并且可以前缀和维护。

用上述算法在线求出 **Primitive Squares** 即可。复杂度 $O(n\log^2n)$。

[code](https://qoj.ac/submission/2256076)。（自觉强制在线后在原题的提交）

## 参考文献

+ [从 Lyndon Words 到 Three Squares Lemma（博客园）](https://www.cnblogs.com/zght/p/13443305.html) 及其引用的参考文献。

## 致谢

感谢 Milmon、CarroT1212 对于题解版本的初稿进行了初步阅读。

感谢 hxhhxh 在首发前耐心帮我读一遍。