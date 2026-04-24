## 前言

突然发现并查集题单里面出现了 [P11593 [NordicOI 2024] Thin Ice](https://www.luogu.com.cn/problem/P11593) ，但我不会 Kruskal 重构树（至少当时不会），于是学习了一下，并写了这篇学习笔记。

## Kruskal 重构树

下文只讨论最小生成树，最大生成树同理。

Kruskal 求最小生成树都会吧， Kruskal 重构树就是在每次合并节点时新建出一个父亲节点来构造出一个二叉树。

具体操作过程如下：

1. 找到一条当前权值最小的边。

2. 找到这条边两个端点当前所在树的根节点。

3. 新建一个节点作为新的根节点，把找到的两个根节点作为它的左右儿子。

4. 把新建点的权值设为选中边的权值。

5. 删去这条边。

6. 循环至第一步。

举个例子（来自 OIwiki ）：

![mst5](https://cdn.luogu.com.cn/upload/image_hosting/vtcaunia.png?x-oss-process=image/resize,m_lfit,h_170,w_225)

这张图的 Kruskal 重构树如下：

![mst6](https://cdn.luogu.com.cn/upload/image_hosting/50up4a3o.png?x-oss-process=image/resize,m_lfit,h_170,w_225)

Kruskal 重构树的性质：

1. 原图上的点一定是 Kruskal 重构树的叶子节点。

2. Kruskal 重构树的叶子节点一定是原图上的点。

3. 除了叶子节点， Kruskal 重构树上的所有节点的父亲的权值一定大于当前节点的权值。

4. 两个点的 LCA 的权值就是原图上两个点之间的所有路径上最大值的最小值。

证明：

1. 注意到过程中能拥有儿子的一定是新建出来的节点，那么原图上的点就没有儿子，所以只能是叶子节点。

2. 同理，一个节点被新建出来是就一定会有儿子，没有儿子的叶子节点就一定是原图上的点。

3. 每个点在新建时的权值都是当前图上最小的权值，比它大的权值都还没有新建点，所以它的儿子的权值只能比它小。

4. 由于我们是按照边权值由小到大来建立 Kruskal 重构树的，所以树上两个点的路径上点的最大值一定是最小的，又由于性质 3 ，两个点的 LCA 是这两个点的路径上的最大值，所以它是原图上两个点路径上的最大值的最小值。

## 例题： [P1967 [NOIP 2013 提高组] 货车运输](https://www.luogu.com.cn/problem/P1967)

这是一道模板题。

不难发现题目就是要我们求图上两个点间所有路径上边的最小值的最大值，我们直接建出 Kruskal 重构树并求两点的 LCA 即可。

:::info[code]

```
#include <bits/stdc++.h>
using namespace std;

const int M = 5e4;
const int N = 1e4;
const int LogN = 15;

struct Edge {
	int u, v, w;
} edge[M + 5];
int n, m, edge_cnt, fa[2 * N + 5], head_cnt, val[2 * N + 5], le[2 * N + 5], ri[2 * N + 5];
int a[2 * N + 5], a_cnt, tt[2 * N + 5], dfn[2 * N + 5], dfn_cnt, st[2 * N + 5][LogN + 5];
int pw[LogN + 5], lg[2 * N + 5], q, f[2 * N + 5];
bool vis[2 * N + 5];

bool cmp(Edge x, Edge y) {
	return x.w > y.w;
}
int Find(int x) {
	if (fa[x] == x) return x;
	return fa[x] = Find(fa[x]);
}
void Union(int x, int y, int w) {
	x = Find(x);
	y = Find(y);
	if (x == y) return;
	++head_cnt;
	fa[x] = head_cnt;
	fa[y] = head_cnt;
	val[head_cnt] = w;
	le[head_cnt] = x;
	ri[head_cnt] = y;
	f[x] = f[y] = head_cnt;
	return;
}
void dfs(int u) {
	vis[u] = true;
	dfn[u] = ++dfn_cnt;
	++a_cnt;
	a[a_cnt] = f[u];
	tt[u] = a_cnt;
	if (le[u] == 0 && ri[u] == 0) return;
	dfs(le[u]);
	dfs(ri[u]);
	return;
}
int Min(int x, int y) {
	if (dfn[x] < dfn[y]) return x;
	return y;
}
int LCA(int x, int y) {
	if (Find(x) != Find(y)) return -1;
	if (x == y) return x;
	if (tt[x] > tt[y]) swap(x, y);
	int tmp = lg[tt[y] - tt[x]];
	return Min(st[tt[x] + 1][tmp], st[tt[y] - pw[tmp] + 1][tmp]);
}
int main() {
	pw[0] = 1;
	for (int i = 1; i <= LogN; i++) pw[i] = pw[i - 1] << 1;
	lg[1] = 0;
	for (int i = 2; i <= 2 * N; i++) lg[i] = lg[i / 2] + 1;
	scanf("%d%d", &n, &m);
	head_cnt = n;
	for (int i = 1; i <= 2 * n; i++) fa[i] = i;
	int u, v, w;
	for (int i = 1; i <= m; i++) {
		scanf("%d%d%d", &u, &v, &w);
		edge[++edge_cnt] = (Edge) {u, v, w};
	}
	sort(edge + 1, edge + edge_cnt + 1, cmp);
	for (int i = 1; i <= m; i++) {
		Union(edge[i].u, edge[i].v, edge[i].w);
	}
	for (int i = 1; i <= head_cnt; i++) if (! vis[i]) dfs(Find(i));
	for (int i = 1; i <= head_cnt; i++) st[i][0] = a[i];
	for (int j = 1; j <= LogN; j++) {
		for (int i = 1; i <= head_cnt; i++) {
			if (i + pw[j] - 1 > head_cnt) continue;
			st[i][j] = Min(st[i][j - 1], st[i + pw[j - 1]][j - 1]);
		}
	}
	scanf("%d", &q);
	while (q--) {
		scanf("%d%d", &u, &v);
		int tmp = LCA(u, v);
		if (tmp == -1) printf("-1\n");
		else printf("%d\n", val[tmp]);
	}
	return 0;
}
```
:::

## 例题： [P4768 [NOI2018] 归程](https://www.luogu.com.cn/problem/P4768)

由于海拔 $> p$ 的道路我们可以随意乱走，在这些道路可以走到的点中找到一个点到 1 的距离最小，然后在这个点下车走到 1 号点就是最优的。换而言之，在这些点中找到距离 1 最小的点，这个距离就是答案。

根据性质 3 ，我们可以找到一个包含 v 的较大子树使得这个子树内所有点的路径都 $> p$ ，即可以随意乱走，那我们可以做一个树上 dp ，令 $dp_i$ 表示以 $i$ 为根的子树中到 1 号点距离的最大值，转移也是好写的：

$$dp_i \gets \begin{cases} \min (dp_{l_i}, dp_{r_i}) & \text{i 不为叶子节点} \\ dis_i & \text{i 为叶子节点} \end{cases}$$

那么这题思路就清晰了：我们先用 Dijkstra 计算出每个点到 1 的距离，然后在 Kruskal 重构树上从 v 开始不断向上跳直到找到一个 $dp_i \le p$ 的祖先，最后输出最后一个合法子树的 dp 值，这个找祖先的过程可以用倍增优化。

:::info[code]

```
#include <bits/stdc++.h>
using namespace std;

const int M = 4e5;
const int N = 2e5;
const int INF = 0x7fffffff;
const int Log2N = 20;

struct Kru {
	int u, v, a;
} kru[M + 5];
struct Edge {
	int l, v, nxt;
} edge[2 * M + 5];
struct Dij {
	int val, pos;
	bool operator < (const Dij &flc) const {
		return val > flc.val;
	}
};
int n, m, kru_cnt, fa[2 * N + 5], le[2 * N + 5], ri[2 * N + 5], val[2 * N + 5];
int head_cnt, head[N + 5], edge_cnt, dis[N + 5], dp[2 * N + 5], f[2 * N + 5][Log2N + 5];
int q, k, s, lastans;
bool vis[N + 5];
priority_queue <Dij> que;

bool cmp(Kru x, Kru y) {
	return x.a > y.a;
}
int Find(int x) {
	if (fa[x] == x) return x;
	return fa[x] = Find(fa[x]);
}
void Union(int x, int y, int a) {
	x = Find(x);
	y = Find(y);
	if (x == y) return;
	++head_cnt;
	f[x][0] = f[y][0] = fa[x] = fa[y] = head_cnt;
	val[head_cnt] = a;
	le[head_cnt] = x;
	ri[head_cnt] = y;
	return;
}
void add(int u, int v, int l) {
	++edge_cnt;
	edge[edge_cnt].v = v;
	edge[edge_cnt].l = l;
	edge[edge_cnt].nxt = head[u];
	head[u] = edge_cnt;
	return;
}
void dij() {
	while (! que.empty()) que.pop();
	for (int i = 1; i <= n; i++) dis[i] = INF;
	dis[1] = 0;
	que.push((Dij) {0, 1});
	while (! que.empty()) {
		int u = que.top().pos;
		que.pop();
		vis[u] = true;
		int v, l;
		for (int i = head[u]; ~ i; i = edge[i].nxt) {
			v = edge[i].v;
			l = edge[i].l;
			if (vis[v]) continue;
			if (dis[u] + l < dis[v]) {
				dis[v] = dis[u] + l;
				que.push((Dij) {dis[v], v});
			}
		}
	}
	return;
}
void dfs(int u) {
	if (le[u] == 0 && ri[u] == 0) {
		dp[u] = dis[u];
		return;
	}
	dfs(le[u]);
	dfs(ri[u]);
	dp[u] = min(dp[le[u]], dp[ri[u]]);
	return;
}
void Solve() {
	memset(f, 0, sizeof(f));
	memset(le, 0, sizeof(le));
	memset(ri, 0, sizeof(ri));
	memset(val, 0, sizeof(val));
	memset(vis, false, sizeof(vis));
	kru_cnt = 0;
	edge_cnt = 0;
	memset(head, -1, sizeof(head));
	scanf("%d%d", &n, &m);
	head_cnt = n;
	for (int i = 1; i <= 2 * n; i++) fa[i] = i;
	int u, v, l, a;
	for (int i = 1; i <= m; i++) {
		scanf("%d%d%d%d", &u, &v, &l, &a);
		kru[++kru_cnt] = (Kru) {u, v, a};
		add(u, v, l);
		add(v, u, l);
	}
	sort(kru + 1, kru + m + 1, cmp);
	for (int i = 1; i <= m; i++) Union(kru[i].u, kru[i].v, kru[i].a);
	dij();
	dfs(head_cnt);
	for (int j = 1; j <= Log2N; j++) 
		for (int i = 1; i <= head_cnt; i++) 
			f[i][j] = f[f[i][j - 1]][j - 1];
	scanf("%d%d%d", &q, &k, &s);
	lastans = 0;
	int p;
	while (q--) {
		scanf("%d%d", &v, &p);
		v = (v + k * lastans - 1) % n + 1;
		p = (p + k * lastans) % (s + 1);
		for (int i = Log2N; i >= 0; i--) {
			if (f[v][i] == 0 || val[f[v][i]] <= p) continue;
			v = f[v][i];
		}
		printf("%d\n", dp[v]);
		lastans = dp[v];
	}
	return;
}
int main() {
	int T;
	scanf("%d", &T);
	while (T--) Solve();
	return 0;
}
```

:::

## 例题： [P4197 [ONTAK2010] Peaks](https://www.luogu.com.cn/problem/P4197)

题目要求只能走权值 $\le x$ 的路径，与上一题类似，我们可以用倍增法找到 Kruskal 重构树中包含 v 的较大子树，在这个子树内的山峰都可以走到，如果我们按 dfs 序给这些山峰标号，答案就是求一个区间的第 k 大值，这个区间是静态的，用主席树维护即可。

:::info[code]

```
#include <bits/stdc++.h>
using namespace std;

const int N = 1e5;
const int M = 5e5;
const int Log2N = 20;
const int LogC = 35;
const int C = 1e9;

struct Kru {
	int u, v, w;
} kru[M + 5];
struct Tree {
	int l, r, val;
} tree[N * LogC + 5];
int n, m, q, c[N + 5], fa[2 * N + 5], le[2 * N + 5], ri[2 * N + 5], co[2 * N + 5], head_cnt;
int f[2 * N + 5][Log2N + 5], ll[2 * N + 5], rr[2 * N + 5], in_tree_cnt, tree_cnt, rt[N + 5];
bool vis[2 * N + 5];

bool cmp(Kru x, Kru y) {
	return x.w < y.w;
}
int Find(int x) {
	if (fa[x] == x) return x;
	return fa[x] = Find(fa[x]);
}
void Union(int x, int y, int w) {
	x = Find(x);
	y = Find(y);
	if (x == y) return;
	++head_cnt;
	fa[x] = fa[y] = head_cnt;
	le[head_cnt] = x;
	ri[head_cnt] = y;
	co[head_cnt] = w;
	f[x][0] = f[y][0] = head_cnt;
	return;
}
void push_up(int k) {
	tree[k].val = tree[tree[k].l].val + tree[tree[k].r].val;
	return;
}
void change(int k, int l, int r, int mb) {
	if (l == r) {
		tree[k].val++;
		return;
	}
	int mid = (l + r) >> 1;
	if (mb <= mid) {
		++tree_cnt;
		tree[tree_cnt] = tree[tree[k].l];
		tree[k].l = tree_cnt;
		change(tree[k].l, l, mid, mb);
	} else {
		++tree_cnt;
		tree[tree_cnt] = tree[tree[k].r];
		tree[k].r = tree_cnt;
		change(tree[k].r, mid + 1, r, mb);
	}
	push_up(k);
	return;
}
int query(int k1, int k2, int l, int r, int mb) {
	if (l == r) return l;
	int mid = (l + r) >> 1, tmp = tree[tree[k2].r].val - tree[tree[k1].r].val;
	if (tmp < mb) return query(tree[k1].l, tree[k2].l, l, mid, mb - tmp);
	else return query(tree[k1].r, tree[k2].r, mid + 1, r, mb);
}
void dfs(int u) {
	vis[u] = true;
	if (le[u] == 0 && ri[u] == 0) {
		++in_tree_cnt;
		ll[u] = in_tree_cnt;
		rr[u] = in_tree_cnt;
		rt[in_tree_cnt] = ++tree_cnt;
		tree[rt[in_tree_cnt]] = tree[rt[in_tree_cnt - 1]];
		change(rt[in_tree_cnt], 1, C, c[u]);
		return;
	}
	dfs(le[u]);
	dfs(ri[u]);
	ll[u] = ll[le[u]];
	rr[u] = rr[ri[u]];
	return;
}
int main() {
	scanf("%d%d%d", &n, &m, &q);
	head_cnt = n;
	for (int i = 1; i <= n; i++) scanf("%d", &c[i]);
	for (int i = 1; i <= 2 * n; i++) fa[i] = i;
	int u, v, w;
	for (int i = 1; i <= m; i++) {
		scanf("%d%d%d", &u, &v, &w);
		kru[i] = (Kru) {u, v, w};
	}
	sort(kru + 1, kru + m + 1, cmp);
	for (int i = 1; i <= m; i++) Union(kru[i].u, kru[i].v, kru[i].w);
	for (int i = 1; i <= head_cnt; i++) if (! vis[i]) dfs(Find(i));
	for (int j = 1; j <= Log2N; j++) 
		for (int i = 1; i <= head_cnt; i++) 
			f[i][j] = f[f[i][j - 1]][j - 1];
	int x, k;
	while (q--) {
		scanf("%d%d%d", &v, &x, &k);
		for (int i = Log2N; i >= 0; i--) {
			if (f[v][i] == 0 || co[f[v][i]] > x) continue;
			v = f[v][i];
		}
		if (tree[rt[rr[v]]].val - tree[rt[ll[v] - 1]].val < k) {
			printf("-1\n");
			continue;
		}
		printf("%d\n", query(rt[ll[v] - 1], rt[rr[v]], 1, C, k));
	}
	return 0;
}
```
:::

## 例题： [P11593 [NordicOI 2024] Thin Ice](https://www.luogu.com.cn/problem/P11593)

这题正着拿硬币后效性超级大，我们考虑反着放硬币。

注意到我们如果想要从一个点走到另外一个点身上的硬币数应当 $\le \min d$ 那我们就可以在这两个点中建边，边权为这两个点的 $d$ 的最小值，对着这个东西跑最大生成树，题目就变成了从一个边缘节点在最大生成树上不断放硬币最后看能不能放完。

由于答案具有单调性，可以用二分答案解决这个问题。

如果我们建立 Kruskal 重构树，就可以在上面跑树形 dp ，用 $dp_i$ 表示以 i 为根的子树下随意放硬币能剩下的最大硬币，我们可以写出转移：

$$dp_i \gets \begin{cases} k - size_i & \max (dp_{l_i}, dp_{r_i}) \le val_i \\ \inf & \text{else} \end{cases}$$

然后记录能否从一个边缘节点转移到 $i$ ，判断如果能从边缘节点转移到 $i$ 且 $dp_i \le 0$ 则说明答案为 $k$ 时是成立的，做完了。

:::info[code]

```
#include <bits/stdc++.h>
using namespace std;

const int N = 2e5;
const int INF = 0x7fffffff;

struct Edge {
	int u, v, w;
} edge[4 * N + 5];
int n, m, a[N + 5], dx[4] = {0, 0, 1, -1}, dy[4] = {-1, 1, 0, 0}, edge_cnt, fa[2 * N + 5], head_cnt;
int le[2 * N + 5], ri[2 * N + 5], val[2 * N + 5], hy[2 * N + 5], dp[2 * N + 5];
bool is_b[2 * N + 5], flag;

int ZH(int x, int y) {
	return (x - 1) * m + y;
}
bool cmp(Edge x, Edge y) {
	return x.w > y.w;
}
int Find(int x) {
	if (fa[x] == x) return x;
	return fa[x] = Find(fa[x]);
}
void Union(int x, int y, int w) {
	x = Find(x);
	y = Find(y);
	if (x == y) return;
	++head_cnt;
	fa[x] = head_cnt;
	fa[y] = head_cnt;
	le[head_cnt] = x;
	ri[head_cnt] = y;
	val[head_cnt] = w;
	hy[head_cnt] = hy[le[head_cnt]] + hy[ri[head_cnt]];
	return;
}
void dfs(int u, int x) {
	if (le[u] == 0 && ri[u] == 0) {
		dp[u] = x > val[u] ? INF : x - 1;
		return;
	}
	dfs(le[u], x);
	dfs(ri[u], x);
	if (min(dp[le[u]], dp[ri[u]]) > val[u]) {
		dp[u] = INF;
		is_b[u] = false;
	} else {
		dp[u] = x - hy[u];
		is_b[u] = false;
		is_b[u] = is_b[u] || (dp[le[u]] <= val[u] && is_b[le[u]]);
		is_b[u] = is_b[u] || (dp[ri[u]] <= val[u] && is_b[ri[u]]);
	}
	if (dp[u] <= 0 && is_b[u]) flag = true;
	return;
}
bool check(int x) {
	flag = false;
	dfs(head_cnt, x);
	return flag;
}
int binary_query() {
	int l = 0, r = n * m;
	while (l < r) {
		int mid = (l + r + 1) >> 1;
		if (check(mid)) l = mid;
		else r = mid - 1;
	}
	return l;
}
int main() {
	scanf("%d%d", &n, &m);
	for (int i = 1; i <= 2 * N; i++) {
		fa[i] = i;
		hy[i] = 1;
	}
	head_cnt = n * m;
	for (int i = 1; i <= n; i++) 
		for (int j = 1; j <= m; j++) {
			scanf("%d", &a[ZH(i, j)]);
			val[ZH(i, j)] = a[ZH(i, j)];
			if (i == 1 || i == n || j == 1 || j == m) is_b[ZH(i, j)] = true;
		}
	for (int i = 1; i <= n; i++) 
		for (int j = 1; j <= m; j++) 
			for (int k = 0; k < 4; k++) {
				int xx = i + dx[k], yy = j + dy[k];
				if (xx < 1 || xx > n || yy < 1 || yy > m) continue;
				edge[++edge_cnt] = (Edge) {ZH(i, j), ZH(xx, yy), min(a[ZH(i, j)], a[ZH(xx, yy)])};
			}
	sort(edge + 1, edge + edge_cnt + 1, cmp);
	for (int i = 1; i <= edge_cnt; i++) Union(edge[i].u, edge[i].v, edge[i].w);
	printf("%d", binary_query());
	return 0;
}
```
:::

## 小结

对于一个某个值 $<$ 或 $>$ 另一个值就可以随意移动的图上问题，我们可以建立出 Kruskal 重构树解决问题。