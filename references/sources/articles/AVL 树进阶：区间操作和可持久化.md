本文为 AVL 树的区间操作与可持久化，如果读者不知道 AVL 树是什么，那么请移步[AVL 树基础 - 洛谷专栏](https://www.luogu.com.cn/article/x0a22idn)。

在基础篇中，读者已经可以用 AVL 解决普通平衡树了。下文将讲解文艺平衡树。

# 区间操作

我们发现文艺平衡树就是要区间操作，原来的平衡树满足中序遍历后权值递增，于是我们可以将下标当作权值，这样平衡树就满足中序遍历后为原序列。

## 翻转

先考虑全局翻转，发现只需要交换每个节点的左右儿子即可，于是我们再要翻转时给根打个标记，之后查询时进行下放。

```cpp
il void pushtag(int u){if(!u)return;swap(ls(u),rs(u)),tag[u]^=1;}
il void pushdown(int u){
	if(!tag[u])	return;
	pushtag(ls(u)),pushtag(rs(u)),tag[u]=0; 
}
```

## 分裂合并

现在我们要对区间进行操作，假设我们要操作区间 $[i,n]$ ，那么我们可以把平衡树分成两棵树 $T1$ 和 $T2$，一颗维护 $[1,i-1]$，另一颗维护 $[i,n]$。然后给 $T2$ 打上标记，最后在合并回去，现在问题来到了如何分裂。

### 严格平衡的方法

先给出代码：

```cpp
il void join(int x,int &u,int d){
	if(h[u]-h[ch[x][d]]<2){
		ch[x][!d]=u;
		pushup(u=x);
	}
	else{
		pushdown(u);
		join(x,ch[u][d],d);
		maintain(u);
	}
}
il void merge(int &u){
	int x=u,d=BF(u)>0;
	join(x,u=ch[u][!d],d);
}
il void split(int u,int &x,int &y,int k){
	if(!u)	return x=0,y=0,void();
	pushdown(u);
	if(siz[ls(u)]+1<k){
		x=u;
		split(rs(u),rs(x),y,k-siz[ls(u)]-1);
		merge(x);
	}
	else{
		y=u;
		split(ls(u),x,ls(y),k);
		merge(y);
	}
}
```

这段代码包含三个辅助函数：`join`、`merge` 和 `split`。它们共同完成将一棵 AVL 树按大小（中序排名）拆分成两棵平衡树的功能。下面逐一解释每个函数的作用和原理。

#### 1. `join(int x, int &u, int d)`

**作用**：将一棵子树 `u` 作为节点 `x` 的 `d` 方向的孩子（`d=0` 表示左，`d=1` 表示右）连接起来，并保证整棵树仍满足 AVL 平衡性质。最终 `u` 会被更新为合并后的根节点。

**参数**：

- `x`：已经存在的节点，它将成为父节点。
- `u`：待插入的子树（引用），函数执行后 `u` 变为合并后的树根。
- `d`：方向，表示将 `u` 连接到 `x` 的哪一侧。

**过程**：

1. 检查 `x` 在 `d` 方向的高度与 `u` 的高度差是否小于 2。  
   - 如果 `h[ch[x][d]] - h[u] < 2`（即插入后不会导致 `x` 失衡），则直接将 `u` 设为 `x` 的 `d` 方向孩子，并更新 `u` 为 `x`（因为 `x` 成为新根）。
   - 否则，需要将 `u` 进一步向下调整：先下传 `u` 的懒标记（`pushdown`），然后**递归**地将 `x` 插入到 `u` 的 `d` 方向孩子中（注意参数顺序变为 `join(x, ch[u][d], d)`）。递归返回后，调用 `maintain(u)` 对 `u` 进行平衡调整（可能涉及旋转），使 `u` 恢复 AVL 性质。

这个函数类似于 AVL 树插入节点后的递归调整过程，但这里处理的不是单个节点，而是一整棵子树。

#### 2. `merge(int &u)`

**作用**：当节点 `u` 的某一侧子树被修改后，调用此函数将 `u` 与它的另一侧子树合并，并重新平衡。通常用于递归返回后，将当前节点与已经调整好的子树合并。

**参数**：

- `u`：当前子树的根（引用）。

**过程**：

1. 暂存当前根 `x = u`。
2. 根据 `u` 的平衡因子 `BF(u)` 决定方向 `d`。  
   - 若 `BF(u) > 0`（左子树更高），则 `d = 1`；否则 `d = 0`。这个方向表示需要将 `u` 的**相反方向**的孩子与 `u` 合并，以平衡高度差。
3. 调用 `join(x, u = ch[u][!d], d)`。  
   - 这里 `u` 被更新为 `ch[u][!d]`（即与 `d` 相反方向的孩子），然后调用 `join` 将 `x` 连接到这个孩子的 `d` 方向。实际上，这相当于将 `u` 和它的一个孩子进行合并，使得树恢复平衡。

可以理解为：`merge` 是一种针对单节点的平衡修复函数，它利用 `join` 将当前节点与其较高子树的一侧合并，从而降低高度差。

#### 3. `split(int u, int &x, int &y, int k)`

**作用**：将一棵以 `u` 为根的 AVL 树按中序遍历顺序拆分成两棵平衡树，前 `k` 个节点放入 `x`，其余放入 `y`。

**参数**：

- `u`：原树的根。
- `x, y`：引用返回的左右树根。
- `k`：左边树的大小。

**过程**：

1. 如果 `u` 为空，则直接令 `x = y = 0` 返回。
2. 下传 `u` 的懒标记（`pushdown`）。
3. 计算左子树大小 `siz[ls(u)]`。  
   - 若 `siz[ls(u)] + 1 < k`，说明当前节点属于左边（因为左子树加上当前节点仍不足 k 个）。  
     - 将当前节点赋给 `x`（即 `x = u`）。
     - 递归分裂右子树：`split(rs(u), rs(x), y, k - siz[ls(u)] - 1)`。  
       - 右子树中，前 `k - siz[ls(u)] - 1` 个节点作为 `x` 的新右孩子，其余作为 `y`。
     - 递归返回后，对 `x` 调用 `merge(x)`，因为 `x` 的右子树被修改，可能需要平衡调整。
   - 否则，当前节点属于右边。  
     - 将当前节点赋给 `y`（即 `y = u`）。
     - 递归分裂左子树：`split(ls(u), x, ls(y), k)`。  
       - 左子树中，前 `k` 个节点作为 `x`，其余作为 `y` 的新左孩子。
     - 递归返回后，对 `y` 调用 `merge(y)` 进行平衡调整。

**核心思想**：采用递归分治，将当前节点划归到一边，然后递归处理另一边的子树，最后用 `merge` 修复可能失衡的树。这样保证分裂后的两棵树仍然是 AVL 树。

#### 总结

这三个函数配合实现了 AVL 树的**分裂**操作：

- `join` 负责将两棵树按指定方向合并，同时维持平衡。
- `merge` 是 `join` 的一种特例，用于修复单节点与其孩子的不平衡。
- `split` 通过递归划分节点，并在回溯时调用 `merge` 来保证平衡性。

这种实现方式类似于 Treap 或 Splay 中的分裂，但针对 AVL 树的高度平衡特性，通过 `join` 和 `merge` 动态调整高度差，确保了分裂后的树仍然满足 AVL 条件。

### 非严格平衡的方法（推荐）

虽然以下方法不满足 AVL 树的性质（也许并不能称之为 AVL 树），但是树高为 $O(\log n)$ 级别，常数更小，所以更推荐。具体的，我们采用类似 FHQ 的方法去分裂合并。

```cpp
void split(int &rt1,int &rt2,int p,int x){
	if(!x){
		rt1=0,rt2=p;
		return;
	}
	pushdown(p);
	if(siz[ls(p)]+1<=x){
		rt1=p;
		split(rs(p),rt2,rs(p),x-siz[ls(p)]-1);
		pushup(p);
		return;
	}
	else{
		rt2=p;
		split(rt1,ls(p),ls(p),x);
		pushup(p);
		return;
	}
}
inline int merge(int u,int v){
	if(!u)	return v;
	if(!v)	return u;
	pushdown(u);
	pushdown(v);
	if(h[u]>=h[v]){
		rs(u)=merge(rs(u),v);
		maintain(u);
		return u;
	}
	else{
		ls(v)=merge(u,ls(v));
		maintain(v);
		return v;
	}
}
```

正确性显然，但是分裂后的两颗树不再是 AVL 树，所以可能发生复杂度的变化，以下为复杂度证明：

1. **基本性质**：每个节点存储高度 `h`，通过 `pushup` 更新，`maintain` 函数在插入、删除或合并后检查平衡因子（绝对值不超过 1）并进行旋转，确保任意时刻树的高度与节点数 $n$ 的关系为 $h\le c\log n$（$c$ 为常数）。

2. **分裂操作**：`split`  沿中序路径递归断开，只更新高度而不旋转。由于分裂得到的子树是原树的一部分，其节点数减少，且原树高度为 $O(\log n)$，因此分裂后的子树高度不超过原树高度，仍为 $O(\log n)$。尽管可能暂时不平衡，但高度信息正确。

3. **合并操作**：`merge` 递归地将一棵树插入另一棵，并在返回时调用 `maintain` 修复不平衡。该过程类似于 AVL 树的插入，每次递归深度不超过树高，且最终结果满足 AVL 性质，高度仍为 $O(\log n)$。

综上，所有操作后树的高度始终为 $O(\log n)$。

于是我们就可以实现文艺平衡树了。

::::info[第二种实现]

```cpp
#include<bits/stdc++.h>
#define N 100005
using namespace std;
int n,q,rt;
#define BF(u) (h[ch[u][0]]-h[ch[u][1]])
#define ls(u) (ch[u][0])
#define rs(u) (ch[u][1])
int tot,ch[N][2],siz[N],val[N],h[N],tag[N];
int stk[N],top;
inline int newnode(int x){
	int u=(top?stk[top--]:++tot);
	val[u]=x,siz[u]=h[u]=1,ls(u)=rs(u)=tag[u]=0;
	return u;
}
inline void pushup(int u){
	siz[u]=siz[ls(u)]+siz[rs(u)]+1;
	h[u]=max(h[ls(u)],h[rs(u)])+1;
}
inline void pushdown(int u){
	if(!tag[u])	return;
	swap(ls(u),rs(u));
	if(ls(u))	tag[ls(u)]^=tag[u];
	if(rs(u))	tag[rs(u)]^=tag[u];
	tag[u]=0;
}
inline void rotate(int &u,bool f){
	int v=ch[u][f];
	pushdown(v);
	ch[u][f]=ch[v][!f];
	ch[v][!f]=u;
	pushup(u),pushup(v),u=v;
}
inline void maintain(int &u){
	pushdown(u);
	int chk=BF(u);
	if(chk>1){
		pushdown(ls(u));
		if(BF(ls(u))<=0)	rotate(ls(u),1);
		rotate(u,0);
	}
	else if(chk<-1){
		pushdown(rs(u));
		if(BF(rs(u))>=0)	rotate(rs(u),0);
		rotate(u,1);
	}
	else if(u)	pushup(u);
}
inline void insert(int &u,int k,int w){
	if(!u)	return void(u=newnode(w));
	pushdown(u);
	if(k<=siz[ls(u)])	insert(ls(u),k,w);
	else	insert(rs(u),k-siz[ls(u)]-1,w);
	maintain(u);
}
inline void del(int &u,int k){
	pushdown(u);
	if(k==siz[ls(u)]+1){
		int v=u;
		if(ls(u)&&(v=rs(u))){
			while(pushdown(v),ls(v))	v=ls(v);
			val[u]=val[v],del(rs(u),1);
		}
		else	stk[++top]=u,u=ls(u)?ls(u):rs(u);
	}
	else if(k<=siz[ls(u)])	del(ls(u),k);
	else	del(rs(u),k-siz[ls(u)]-1);
	maintain(u);
}
inline int kth(int u,int x){
	int tmp=0;
	while(u){
		pushdown(u);
		if((tmp=siz[ls(u)]+1)==x)	return val[u];
		else	u=((tmp>x)?ls(u):(x-=tmp,rs(u)));
	}
	return -1;
}
void split(int &rt1,int &rt2,int p,int x){
	if(!x){
		rt1=0,rt2=p;
		return;
	}
	pushdown(p);
	if(siz[ls(p)]+1<=x){
		rt1=p;
		split(rs(p),rt2,rs(p),x-siz[ls(p)]-1);
		pushup(p);
		return;
	}
	else{
		rt2=p;
		split(rt1,ls(p),ls(p),x);
		pushup(p);
		return;
	}
}
inline int merge(int u,int v){
	if(!u)	return v;
	if(!v)	return u;
	pushdown(u);
	pushdown(v);
	if(h[u]>=h[v]){
		rs(u)=merge(rs(u),v);
		maintain(u);
		return u;
	}
	else{
		ls(v)=merge(u,ls(v));
		maintain(v);
		return v;
	}
}
inline void change(int &u,int l,int r){
	int x=0,y=0,z=0,t=0;
	if(r!=n)	split(t,z,u,r);
	else	t=u;
	if(l!=1)	split(x,y,t,l-1);
	else	y=t;
	tag[y]^=1;
	u=merge(merge(x,y),z);
}
int main(){
	ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
	cin>>n>>q;
	for(int i=1;i<=n;i++)	insert(rt,i-1,i);
	for(int i=1;i<=q;i++){
		int x,y;
		cin>>x>>y;
		change(rt,x,y);
	}
	for(int i=1;i<=n;i++)	cout<<kth(rt,i)<<' ';
	return 0;
}
```

::::
::::info[第一种实现]

```cpp
#include<bits/stdc++.h>
#define N 100005
#define il inline
#define ls(u) (ch[u][0])
#define rs(u) (ch[u][1])
#define BF(u) (h[ch[u][0]]-h[ch[u][1]])
using namespace std;
int n,m,rt,a[N],ch[N][2],siz[N],h[N],val[N],tag[N];
il void pushup(int u){
	h[u]=max(h[ls(u)],h[rs(u)])+1;
	siz[u]=siz[ls(u)]+siz[rs(u)]+1;
}
il void pushtag(int u){if(!u)return;swap(ls(u),rs(u)),tag[u]^=1;}
il void pushdown(int u){
	if(!tag[u])	return;
	pushtag(ls(u)),pushtag(rs(u)),tag[u]=0; 
}
il void rotate(int &u,int f){
	int v=ch[u][f];
	pushdown(v),ch[u][f]=ch[v][!f];
	ch[v][!f]=u,pushup(u),pushup(v),u=v;
}
il void maintain(int &u){
	pushdown(u);
	int chk=BF(u);
	if(chk>1){
		pushdown(ls(u));
		if(BF(ls(u))<=0)	rotate(ls(u),1);
		rotate(u,0);
	}
	else if(chk<-1){
		pushdown(rs(u));
		if(BF(rs(u))>=0)	rotate(rs(u),0);
		rotate(u,1);
	}
	else if(u) pushup(u);
}
il int merge(int u,int v){
	if(!u||!v)	return u+v;
	if(h[u]>h[v]){
		pushdown(u),rs(u)=merge(rs(u),v);
		return maintain(u),u;
	}
	else{
		pushdown(v),ls(v)=merge(u,ls(v));
		return maintain(v),v;
	}
}
il void join(int x,int &u,int d){
	if(h[u]-h[ch[x][d]]<2){
		ch[x][!d]=u;
		pushup(u=x);
	}
	else{
		pushdown(u);
		join(x,ch[u][d],d);
		maintain(u);
	}
}
il void merge(int &u){
	int x=u,d=BF(u)>0;
	join(x,u=ch[u][!d],d);
}
il void split(int u,int &x,int &y,int k){
	if(!u)	return x=0,y=0,void();
	pushdown(u);
	if(siz[ls(u)]+1<k){
		x=u;
		split(rs(u),rs(x),y,k-siz[ls(u)]-1);
		merge(x);
	}
	else{
		y=u;
		split(ls(u),x,ls(y),k);
		merge(y);
	}
}
il void build(int &u,int l,int r){
	if(l>r)	return u=0,void();
	u=(l+r)>>1,val[u]=u;
	build(ls(u),l,u-1),build(rs(u),u+1,r);
	pushup(u);
}
il void print(int u){
	if(!u)	return;
	pushdown(u),print(ls(u)),cout<<val[u]<<' ',print(rs(u));
}
signed main(){
	ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
	cin>>n>>m,build(rt,1,n);
	while(m--){
		int l,r;
		cin>>l>>r;
		int u,v,w;
		split(rt,v,w,r+1),split(v,u,v,l),pushtag(v);
		rt=merge(merge(u,v),w);
	}
	print(rt);
	return 0;
}
```

::::

# 可持久化

每次遇到改变一个节点信息的时候，就复制一份，可以参考代码理解，以下代码均为第二种分裂实现方法：

::::info[可持久化普通平衡树]

```cpp
#include<bits/stdc++.h>
#define N 500005
using namespace std;
int q,rt[N];
struct operate{
	#define BF(u) (h[ch[u][0]]-h[ch[u][1]])
	#define ls(u) (ch[u][0])
	#define rs(u) (ch[u][1])
	int tot,ch[N<<5][2],siz[N<<5],val[N<<5],h[N<<5];
	int newnode(int x){
		int u=++tot;
		val[u]=x,siz[u]=h[u]=1,ls(u)=rs(u)=0;
		return u;
	}
	int copy(int x){
		int u=++tot;
		val[u]=val[x],siz[u]=h[u]=siz[x],ls(u)=ls(x),rs(u)=rs(x);
		return u;
	}
	void pushup(int u){
		siz[u]=siz[ls(u)]+siz[rs(u)]+1;
		h[u]=max(h[ls(u)],h[rs(u)])+1;
	}
	void rotate(int &u,bool f){
		int v=copy(ch[u][f]);
		ch[u][f]=ch[v][!f];
		ch[v][!f]=u;
		pushup(u),pushup(v),u=v;
	}
	void maintain(int &u){
		int chk=BF(u);
		if(chk>1){
			if(BF(ls(u))<=0)	rotate(ls(u),1);
			rotate(u,0);
		}
		else if(chk<-1){
			if(BF(rs(u))>=0)	rotate(rs(u),0);
			rotate(u,1);
		}
		else if(u)	pushup(u);
	}
	void insert(int &u,int w){
		if(!u)	return void(u=newnode(w));
		else	u=copy(u);
		if(val[u]<w)	insert(rs(u),w);
		else	insert(ls(u),w);
		maintain(u);
	}
	void del(int &u,int w){
		if(!u)	return;
		u=copy(u);
		if(val[u]==w){
			int v=u;
			if(ls(u)&&(v=rs(u))){
				while(ls(v))	v=ls(v);
				val[u]=val[v],del(rs(u),val[v]);
			}
			else	u=ls(u)?ls(u):rs(u);
		}
		else if(val[u]<w)	del(rs(u),w);
		else	del(ls(u),w);
		maintain(u);
	}
	int kth(int u,int x){
		int tmp=0;
		while(u){
			if((tmp=siz[ls(u)]+1)==x)	return val[u];
			else	u=((tmp>x)?ls(u):(x-=tmp,rs(u)));
		}
		return -1;
	}
	int qrk(int u,int x){
		int ans=1;
		while(u){
			if(val[u]<x)	ans+=siz[ls(u)]+1,u=rs(u);
			else	u=ls(u);
		}
		return ans;
	}
	int pre(int u,int x){
		int ans=1-(1<<31);
		while(u){
			if(val[u]>=x)	u=ls(u);
			else	ans=val[u],u=rs(u);
		}
		return ans;
	}
	int nxt(int u,int x){
		int ans=(1<<31)-1;
		while(u){
			if(val[u]<=x)	u=rs(u);
			else	ans=val[u],u=ls(u);
		}
		return ans;
	}
}T;
int main(){
	ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
	cin>>q;
	for(int i=1;i<=q;i++){
		int op,x,id;
		cin>>id>>op>>x,rt[i]=rt[id];
		switch(op){
			case 1:T.insert(rt[i],x);break;
			case 2:T.del(rt[i],x);break;
			case 3:cout<<T.qrk(rt[i],x)<<'\n';break;
			case 4:cout<<T.kth(rt[i],x)<<'\n';break;
			case 5:cout<<T.pre(rt[i],x)<<'\n';break;
			case 6:cout<<T.nxt(rt[i],x)<<'\n';break;
		}
	}
	return 0;
}
```

::::

::::info[可持久化文艺平衡树]

```cpp
#include<bits/stdc++.h>
#define N 10000005
#define ll long long
#define il inline
#define ls(u) (ch[u][0])
#define rs(u) (ch[u][1])
#define BF(u) (h[ch[u][0]]-h[ch[u][1]])
using namespace std;
int q,rt[N],ch[N][2],siz[N],h[N],val[N],tag[N],cnt,now;
ll sum[N],ans;
il int newnode(int x){++cnt,h[cnt]=siz[cnt]=1,val[cnt]=sum[cnt]=x;return cnt;}
il int copy(int u){if(u>now)return u;++cnt,ls(cnt)=ls(u),rs(cnt)=rs(u),siz[cnt]=siz[u],h[cnt]=h[u],val[cnt]=val[u],tag[cnt]=tag[u],sum[cnt]=sum[u];return cnt;}
il void pushup(int u){
	h[u]=max(h[ls(u)],h[rs(u)])+1;
	siz[u]=siz[ls(u)]+siz[rs(u)]+1;
	sum[u]=sum[ls(u)]+sum[rs(u)]+val[u];
}
il void pushtag(int u){if(!u)return;swap(ls(u),rs(u)),tag[u]^=1;}
il void pushdown(int u){
	if(!tag[u])	return;
	if(ls(u))	ls(u)=copy(ls(u));
	if(rs(u))	rs(u)=copy(rs(u));
	pushtag(ls(u)),pushtag(rs(u)),tag[u]=0; 
}
il void rotate(int &u,int f){
	int v=ch[u][f];
	pushdown(v),ch[u][f]=ch[v][!f];
	ch[v][!f]=u,pushup(u),pushup(v),u=v;
}
il void maintain(int &u){
	pushdown(u);
	int chk=BF(u);
	if(chk>1){
		pushdown(ls(u)=copy(ls(u)));
		if(BF(ls(u))<=0)	rs(ls(u))=copy(rs(ls(u))),rotate(ls(u),1);
		rotate(u,0);
	}
	else if(chk<-1){
		pushdown(rs(u)=copy(rs(u)));
		if(BF(rs(u))>=0)	ls(rs(u))=copy(ls(rs(u))),rotate(rs(u),0);
		rotate(u,1);
	}
	else if(u) pushup(u);
}
il int merge(int u,int v){
	if(!u||!v)	return u+v;
	if(h[u]>h[v]){
		u=copy(u),pushdown(u),rs(u)=merge(rs(u),v);
		return maintain(u),u;
	}
	else{
		v=copy(v),pushdown(v),ls(v)=merge(u,ls(v));
		return maintain(v),v;
	}
}
il void join(int x,int &u,int d){
	if(h[u]-h[ch[x][d]]<2){
		x=copy(x),ch[x][!d]=u;
		pushup(u=x);
	}
	else{
		u=copy(u),pushdown(u);
		join(x,ch[u][d],d);
		maintain(u);
	}
}
il void merge(int &u){
	int x=u,d=BF(u)>0;
	join(x,u=ch[u][!d],d);
}
il void split(int u,int &x,int &y,int k){
	if(!u)	return x=0,y=0,void();
	u=copy(u),pushdown(u);
	if(siz[ls(u)]+1<k){
		x=u;
		split(rs(u),rs(x),y,k-siz[ls(u)]-1);
		merge(x);
	}
	else{
		y=u;
		split(ls(u),x,ls(y),k);
		merge(y);
	}
}
il ll query(int u,int k){
	ll res=0;
	while(u){
		u=copy(u),pushdown(u);
		if(siz[ls(u)]<k)	res+=sum[ls(u)]+val[u],k-=siz[ls(u)]+1,u=rs(u);
		else	u=ls(u);
	}
	return res;
}
signed main(){
	ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
	cin>>q;
	for(int i=1;i<=q;i++){
		int op,id;
		ll p,x,l,r;
		cin>>id>>op,rt[i]=rt[id],now=cnt;
		if(op==1){
			cin>>p>>x,p^=ans,x^=ans;
			int u,v=newnode(x),w;
			split(rt[i],u,w,p+1);
			rt[i]=merge(merge(u,v),w);
		}
		if(op==2){
			cin>>p,p^=ans;
			int u,v,w;
			split(rt[i],v,w,p+1);
			split(v,u,v,p),rt[i]=merge(u,w);
		}
		if(op==3){
			cin>>l>>r,l^=ans,r^=ans;
			int u,v,w;
			split(rt[i],v,w,r+1),split(v,u,v,l),pushtag(v);
			rt[i]=merge(merge(u,v),w);
		}
		if(op==4){
			cin>>l>>r,l^=ans,r^=ans;
			int u,v,w;
			ans=query(rt[i],r)-query(rt[i],l-1),cout<<ans<<'\n';
		}
	}
	return 0;
}
```

::::