以下 $w=64$ 表示字长。

---

其实是一个有些古老的技巧，算是出自 P8569 吧。不太普及，但是感觉可以放在很多数据结构题上骗分。

对于若干长度为 $n$、值域为 $[0,V]\cap N$ 的序列，以下为说明方便先有两个序列 $\{a\},\{b\}$。

利用这个技巧，我们可以在单次 $\mathcal{O}\left(\dfrac{n \log V}{w}\right)$ 的时间复杂度内做到以下操作：

1. 区间 $a_i \gets a_i+v$。
2. 区间 $a_i \gets \min\{a_i,v\}/\max\{a_i,v\}$。
3. 区间 $a_i \gets a_i \operatorname{xor} v/a_i \operatorname{and} v/a_i \operatorname{or} v$
4. 区间 $a_i \gets a_i+b_i$。与前面配合可以维护历史版本和。
5. 区间 $a_i \gets \min\{a_i,b_i\}/\max\{a_i,b_i\}$。与前面配合可以维护历史最值。
6. 区间除二的幂，即 $a_i \gets \lfloor \dfrac{a_i}{2^k} \rfloor$。
7. 查询区间和$/\max/\min/\operatorname{xor}/\operatorname{and}/\operatorname{or}$。
8. 查询区间有几个数 $\le v$。
9. 查询区间 $\text{popcount}(a_i)$ 之和。

可以说想到什么有什么。常数很小，而且这些操作其实可以共存！实测尤其在许多类似扫描线的题上是比较有用的。



---

为方便讲解，以下我们将 $n \gets w$，对于 $n$ 更大的情况就处理 $\dfrac{n}{w}$ 次即可。

我们将每个 $a_i$ 二进制拆分，写成下图的格式：

![](https://cdn.luogu.com.cn/upload/image_hosting/t9en247z.png)

显然，这是一个 $\lfloor\log V\rfloor \times w$ 的 01 矩阵，其中竖着的那一维长度为 $\lfloor\log V\rfloor$。

事实上，我们直接存储 $a_i$ 的值相当于按蓝色框的方向压位。

但是这样的压位方式比较浪费，因为一个整形压位可以最多压 $w$ 位，但是我们只压了 $\lfloor\log V\rfloor$ 位。

不妨按照绿色框的方向压位，这样一定可以压满 $w$ 位。

这样我们就得到了一个长度为 $\lfloor\log V\rfloor$ 的数组 $\{s\}$，$s_i$ 描述了 $a$ 上有哪些位置的第 $i$ 位为 $1$。

然后我会讲几个操作的做法：

-  整体加 $v$。

   将 $v$ 二进制拆分为 $[v_0,v_1,\ldots ,v_{\lfloor\log V\rfloor}]$。若 $v$ 的第 $i$ 位为 $1$，$v_i=2^{w}-1$，也就是全集。否则为 $0$。从低位向高位枚举，同时维护一个整数 $p$，表示每个位置到当前位是否存在进位。

   显然每一位有 $s_i \gets s_i \operatorname{xor} v_i \operatorname{xor} p$，而 $p \gets (s_i \operatorname{and} v_i)\operatorname{or}\big[(s_i \operatorname{xor} v_i)\operatorname{and} p\big ]$。

   可以十分容易的类比到区间加、上面的操作 4（对 $b$ 也做二进制拆分在某些时候是不可接受的）。

   以下是一份维护操作 4 的代码。

```cpp
#define vc vector<int>
inline vc merge(vc x,vc y){
	if(x.size()<y.size())swap(x,y);int p=0,P,i=0;
	for(;i<y.size();p=P,++i)
		P=(x[i]&y[i])|((x[i]^y[i])&p),x[i]^=(y[i]^p);
	for(;i<x.size();p=P,++i)P=x[i]&p,x[i]^=p;
	if(p)x.push_back(p);return x;
}
```

-  整体对 $v$ 取 $\max$。

   将 $v$ 二进制拆分为 $[v_0,v_1,\ldots ,v_{\lfloor\log V\rfloor}]$。若 $v$ 的第 $i$ 位为 $1$，$v_i=2^{w}-1$，也就是全集。否则为 $0$。从高位向低位枚举，维护一个整数 $p$，表示每个位置是否在前面的位中与 $v$ 一样。还有一个整数 $q$ 表示每个位置是否 $< v$。

   显然每一位有 $q \gets q\operatorname{or}\big [(s_i \operatorname{xor} v_i)\operatorname{and} v_i \operatorname{and} p\big ],p \gets \big [*(s_i \operatorname{xor} v_i)\big ] \operatorname{and} p$。其中我们用 $*x$ 表示 $x$ 取反的结果。

   最后 $q$ 为 $1$ 的位 $j$ 上，对应的 $a_j < v$。将它们赋值为 $v$ 即可。

   可以十分容易的类比到上面的操作 5、操作 8。

如果你理解了这两个，其他的操作相信你也会做了。

---

下表是用这个技巧维护区间取 $\max$ 与直接暴力的效率对比（$n,V$ 同阶）：

|   | $n,m=5 \times 10^4$  | $n,m=10^5$  | $^An,m=10^5 $  | $^An,m=2 \times 10^5 $ | $^{AB}n,m=2 \times 10^5 $ |
|:-:|:-:|:-:|:-:|:-:|:-:|
| 代码 1（二进制拆分）  |  50ms  | 170ms | 220ms  | 550ms | 8230ms |
| 代码 2（暴力）  | 260ms | 880ms  | 1770ms  | 6870ms | 7390ms |

$A$：强制选取长度 $>n/2$ 的区间作为修改区间。

$B$：对于操作，$v$ 随时间递增。

无特殊限制则随机选取。

---

可以发现这个东西如果题目可以对着卡其实不太能保证跑过暴力，特别是这种常数很大的操作。

但是如果数据随机，表现则非常的好。作为复杂度 $\mathcal{O}\left(\dfrac{nm \log V}{w}\right)$ 的东西实际上和 $\mathcal{O}(n \log n+m)$ 或 $\mathcal{O}(n +m \log n)$ 的算法差不多快。

代码 1（二进制拆分）：

```cpp
#include<bits/stdc++.h>
#define ull unsigned long long
#define rd read()
#define gc pa == pb && (pb = (pa = buf) + fread(buf, 1, 100000, stdin), pa == pb) ? EOF : *pa++
using namespace std;
static char buf[100000], * pa(buf), * pb(buf);
inline int read(){
	register int x=0,s=gc;while(!isdigit(s))s=gc;
	while(isdigit(s))x=(x<<1)+(x<<3)+(s^48),s=gc;
	return x;
}
const int N=200005,lgV=18;const ull S=-1;
inline void chkmax(int &x,int y){
	x=x<y?y:x;
}
int n,m,a[N],A[N];ull s[lgV][N>>6];
signed main(){
	n=rd,m=rd;
	for(int i=1,l,r,v;i<=m;++i){
		l=rd,r=rd,v=rd;
		if((l>>6)==(r>>6)){
			for(int j=l;j<=r;++j)chkmax(A[j],v);
		}else{
			for(int j=l;j<(((l>>6)+1)<<6);++j)chkmax(A[j],v);
			for(int j=((r>>6)<<6);j<=r;++j)chkmax(A[j],v);
			for(int j=(l>>6)+1;j<(r>>6);++j){
				ull p=-1,q=0;
				for(int k=lgV-1;~k&&p&&q!=S;--k)
					v>>k&1?q|=(S^s[k][j])&p,p=(s[k][j]&p):p=(S^s[k][j])&p;
				if(!q)continue;
				for(int k=0;k<lgV;++k)
					v>>k&1?s[k][j]|=q:s[k][j]&=(q^S);
			}
		}
	}	
	for(int i=0;i<n;chkmax(a[i],A[i]),++i)
		for(int j=0;j<lgV;++j)a[i]+=(1<<j)*(s[j][i>>6]>>(i&63)&1);
	for(int i=0;i<n;++i)cout<<a[i]<<' ';return 0;
}
```

不过你也会发现，如果精细实现空间复杂度是可以 $\mathcal{O}\left(\dfrac{n \log V}{w}\right)$ 的。

代码 2（暴力）：

```cpp
#include<bits/stdc++.h>
#define rd read()
#define gc pa == pb && (pb = (pa = buf) + fread(buf, 1, 100000, stdin), pa == pb) ? EOF : *pa++
using namespace std;
static char buf[100000], * pa(buf), * pb(buf);
inline int read(){
	register int x=0,s=gc;while(!isdigit(s))s=gc;
	while(isdigit(s))x=(x<<1)+(x<<3)+(s^48),s=gc;
	return x;
}
const int N=200005;
inline void chkmax(int &x,int y){
	x=x<y?y:x;
}
int n,m,a[N];
signed main(){
	n=rd,m=rd;
	for(int i=1,l,r,v;i<=m;++i){
		l=rd,r=rd,v=rd;
		for(int j=l;j<=r;++j)chkmax(a[j],v);
	}
	for(int i=0;i<n;++i)cout<<a[i]<<' ';return 0;
}
```


---

[例题 $1$](https://www.luogu.com.cn/problem/P3917)
>给出序列 $a_1,a_2,\ldots,a_n$，求
>$$\sum_{1\le i\le j\le n} a_i\operatorname{xor} a_{i+1}\operatorname{xor}\cdots\operatorname{xor} a_j$$
>的值。
>
>$n \le 10^5,a_i \le 10^9$。

先对 $a$ 求前缀异或数组 $d$，这样 $a_l\operatorname{xor} a_{l+1}\operatorname{xor}\cdots\operatorname{xor} a_r=d_r\operatorname{xor} d_{l-1}$。

这样，答案显然就是 $\sum\limits_{0\le i\le j\le n}d_i\operatorname{xor}d_j$。

拆位考虑贡献，对于一个位，用序列中 $0$ 的数量与 $1$ 的数量相乘即可。

考虑求出 $c_i$ 表示序列 $d$ 中第 $i$ 位为 $1$ 的位置个数。显然答案是 $\sum \limits _{i=0}^{\lfloor\log V\rfloor}2^i(n-c_i)c_i$。

统计 $c$ 的过程则可以简单做到 $\mathcal{O}(n \log V)$。考虑继续优化。

我们将 $d_i$ 拆位后的结果看作一个 01 序列，$c$ 实际上就是这些 01 序列相同位置加起来的结果。

所以合并的过程实际上就是操作 4！直接使用前面讲到的方法维护依次合并就是 $\mathcal{O}(n \log n)$，注意 $c$ 的值域是 $n$。

将依次合并改为分治合并，每次将 $[l,\text{mid}],(\text{mid},r]$ 的结果合并为 $[l,r]$ 的结果。单次合并 $\mathcal{O}(\log n)$，复杂度是 $T(n)=2T(n/2)+\mathcal{O}(\log n)=\mathcal{O}(n)$。

这样我们就在 $\mathcal{O}(n)$ 的时间复杂度内解决了这个问题。

```cpp
#include<bits/stdc++.h>
#define vc vector<int>
#define int long long
#define rd read()
#define gc pa == pb && (pb = (pa = buf) + fread(buf, 1, 100000, stdin), pa == pb) ? EOF : *pa++
using namespace std;
static char buf[100000], * pa(buf), * pb(buf);
inline int read(){
	register int x=0,s=gc;while(!isdigit(s))s=gc;
	while(isdigit(s))x=(x<<1)+(x<<3)+(s^48),s=gc;
	return x;
}
const int N=100005,w=32;
int n,ans,a[N],c[w];
inline vc merge(vc x,vc y){
	if(x.size()<y.size())swap(x,y);int p=0,P,i=0;
	for(;i<y.size();p=P,++i)
		P=(x[i]&y[i])|((x[i]^y[i])&p),x[i]^=(y[i]^p);
	for(;i<x.size();p=P,++i)P=x[i]&p,x[i]^=p;
	if(p)x.push_back(p);return x;
}
inline vc sol(int l,int r){
	if(l==r)return {a[l]};int mid=l+r>>1;
	return merge(sol(l,mid),sol(mid+1,r));
}
signed main(){
	n=rd;for(int i=1;i<=n;++i)a[i]=a[i-1]^rd;
	vc k=sol(1,n);
	for(int i=0;i<k.size();++i)
		for(int j=0;j<w;++j)if(k[i]>>j&1)c[j]+=(1<<i);
	for(int i=0;i<w;++i)ans+=(1<<i)*c[i]*(n-c[i]+1);
	cout<<ans<<'\n';return 0;
}
```
---

[例题 $2$](https://www.luogu.com.cn/problem/P14764)
>有一个长度为 $n$ 的序列 $\{a\}$ ，$q$ 次询问，每次查询 $[l,r]$ 中恰好出现 $k$ 次的最大数。强制在线。
>
>$n,q,a_i \le 5 \times 10^4$。

考虑预处理出 $f_{p,x}$ 表示在前缀 $[1,p]$ 中 $x$ 的出现次数。

那么询问就是找到最大的 $f_{r,x}-f_{l-1,x}=k$ 的位置。

这是两个数列 $f_{l-1},f_r$ 相减的形式，考虑对值域一维压位，相减与相加基本一样，取出 $=k$ 的位置同样也是简单的。

这样我们就轻松地做到了时空复杂度 $\mathcal{O}\left(\dfrac{nm \log n}{w}\right)$。

这个空间复杂度是不可接受的，但我们发现相邻的 $f_i$ 差别不大。可以在 $\mathcal{O}(\log n)$ 的时间复杂度从 $f_{x}$ 变为 $f_{x+1}$。

考虑设置阈值，只存下所有 $x$ 是 $B$ 的倍数的 $f_x$，然后询问时从上一个 $B$ 的倍数递推到 $l-1,r$ 的位置即可。

空间复杂度是 $\mathcal{O}\left(\dfrac{n^2 \log n}{wB}\right)$，时间复杂度 $\mathcal{O}\left(\dfrac{nm \log n}{w}+mB \log n\right)$。

不妨将 $B$ 看作 $\mathcal{O}(\log n)$ 得到一个比较优美的复杂度：空间 $\mathcal{O}\left(\dfrac{n^2}{w}\right)$，时间 $\mathcal{O}\left(\dfrac{nm \log n}{w}\right)$。

实际上 $B$ 取一个很小的数就足够了，参考代码中有 $B=8$。将值离散化还可以得到一个更小的常数。

建议加入尽可能多的有用的剪枝。

```cpp
#include<bits/stdc++.h>
#define vc basic_string<ull>
#define ull unsigned long long
#define rd read()
#define gc pa == pb && (pb = (pa = buf) + fread(buf, 1, 100000, stdin), pa == pb) ? EOF : *pa++
using namespace std;
static char buf[100000], * pa(buf), * pb(buf);
inline int read(){
	register int x=0,s=gc;while(!isdigit(s))s=gc;
	while(isdigit(s))x=(x<<1)+(x<<3)+(s^48),s=gc;
	return x;
}
const int N=50105;const ull S=-1;
int n,m,v,a[N],p[N];vc f[N>>3][N>>6],e[N>>6];
inline vc g(int x,int u){
	vc res=f[x>>3][u];
	for(int i=((x>>3)<<3)+1;i<=x;++i){
		ull p=(1ull<<(a[i]&63)),P;int t=a[i]>>6;
		if(t!=u)continue;
		for(int j=0;j<res.size()&&p;p=P,++j)P=(p&res[j]),res[j]^=p;
		if(p)res+=p;
	}
	return res;
}
signed main(){
	n=rd;for(int i=1;i<=n;++i)p[i]=a[i]=rd;m=rd;
	sort(p+1,p+n+1),v=unique(p+1,p+n+1)-p-1;
	for(int i=1;i<=n;++i)a[i]=lower_bound(p+1,p+v+1,a[i])-p-1;
	for(int i=1;i<=n;++i){
		ull p=(1ull<<(a[i]&63)),P;int t=a[i]>>6;
		for(int j=0;j<e[t].size()&&p;p=P,++j)P=(p&e[t][j]),e[t][j]^=p;
		if(p)e[t]+=p;if(!(i&7))for(int j=0;j<=(v>>6);++j)f[i>>3][j]=e[j];
	}
	for(int i=1,l,r,k,lst=0;i<=m;++i){
		l=rd^lst,r=rd^lst,k=rd^lst,lst=-1;
		for(int j=(v>>6);~j;--j){
			vc o=g(r,j);if(__lg(k)>=o.size())continue;
			int h=0;vc d=g(l-1,j);ull p=0,P=0,is=-1;
			for(;h<d.size()&&is;p=P,++h)
				P=((p&d[h])|((d[h]|p)&(o[h]^S))),
				o[h]^=d[h]^p,is&=(k>>h&1?o[h]:(o[h]^S));
			for(;h<o.size()&&is;p=P,++h)
				P=p&(o[h]^S),o[h]^=p,is&=(k>>h&1?o[h]:(o[h]^S));
			if(is){lst=(j+1<<6)-__builtin_clzll(is);break;}
		}
		cout<<(lst=lst==-1?0:p[lst])<<'\n';
	}
	return 0;
}
```

最大点只有 350ms。

---

[例题 $3$](https://www.luogu.com.cn/problem/P14945)
>给定一个 $n\times n$ 列且值域为 $[1,n]$ 的正整数方阵 $a$ 和 $q$ 个询问。
>
>每个询问给定四个正整数 $l_1,r_1,l_2,r_2$，求 $|\{a_{i,j}\mid l_1\le i\le r_1,\ l_2\le j\le r_2\}|$。
>
>$n \le 2000,q \le 5\times 10^5$。

离线将值域按 $w$ 分块处理比较方便。

记录 $f_{x,y,i}$ 为所有 $x' \le x,y' \le y$ 的 $a_{x',y'}$中 $i$ 的出现次数。

询问可以变为四个点差分相加减得到的 $\{f\}$ 表示每种数在询问矩阵中的出现次数，然后答案就是 $\sum\limits _{i=1}^{n} [f_i > 0]$。这一步可以利用压位优化并行加减到 $\mathcal{O}\left(\dfrac{n \log n}{w}\right)$。

直接预处理所有的 $f$，配合二进制拆分，复杂度是 $\mathcal{O}\left(\dfrac{n^3 \log n}{w}\right)$。

选取一个阈值 $B$，只存储 $x,y$ 中至少一个为 $B$ 的倍数的 $f_{x,y}$。这样的关键点的数量是 $\mathcal{O}\left(\dfrac{n^2}{B}\right)$ 的。计算时使用边上几个关键点差分然后加上中间的 $\mathcal{O}(B)$ 个散点。

查询的时候差分一下，暴力插入中间点，单次询问代价是 $\mathcal{O}(B^2 \log n)$，预处理的代价是 $\mathcal{O}\left(\dfrac{n^3 \log n}{Bw}+n^2B 
\log n\right)$。

![](https://cdn.luogu.com.cn/upload/image_hosting/38w4tte4.png)

直接取 $B=\mathcal{O}\left(\sqrt{\dfrac{n}{w}}\right)$ 发现复杂度完美平衡，于是我们在 $\mathcal{O}\left(\dfrac{(n\sqrt{nw}+q)n \log n}{w} \right)$ 的复杂度内解决了问题。代码没写，题解区有些类似复杂度的过了，所以应该这个也可以。