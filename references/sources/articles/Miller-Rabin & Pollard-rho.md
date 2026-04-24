# Miller–Rabin & Pollard-rho

## Miller–Rabin

质数判断。

考虑费马小定理：$a^p\equiv a\pmod p$。也就是说，当 $1\le a<p$ 时，$a^{p-1}\equiv 1\pmod p$。

其逆定理几乎也成立。那么随机选择一些底数 $a$，求 $a^{p-1}\bmod p$ 是否为 $1$。如果不是，那么 $p$ 必然不是质数。否则，$p$ 一定是质数。

这是错的……正确率暂且不谈。有一些合数 $n$，对于**任意的** $1\le a<n$ 且 $a\perp n$，都有 $a^{n-1}\equiv 1\pmod n$。这种数字叫做 [Carmichael 数](https://oeis.org/A002997)。这种数字有无穷多个。

看来不太行。考虑一个东西叫做二次探测定理。

对于质数 $p$，那么满足 $x^2\equiv 1\pmod p$ 的 $x$ 只能有 $\pm 1$（$1$ 或 $p-1$）。

怎么证？首先移项并因式分解为 $p\mid (x-1)(x+1)$。此时 $x-1$ 或 $x+1$ 为 $p$ 的倍数。显然，它们不能同时为 $p$ 的倍数。那么，显然 $x=\pm 1$ 为唯一可能解。

对于合数就不一定了。比如，$3^2\equiv 1\pmod 8$。

那么，每次随机一个数字 $a$（$1<a<p-1$），判断是否有 $a^2\equiv 1\pmod n$……？

诶，你会发现，这个还是错完了。这种数字的反例比 Carmichael 数还要密集得多。光是 $[1,10^5]$ 以内就有 $5296$ 个，占比高达 $0.05296\%$！我们要的是接近 $0\%$ 的占比。

但是你会发现，这种数字和 Carmichael 数……好像没有交集？

还真是。我们先写出判断证据（即能够证明 $n$ 不是质数的数字）的伪代码。

$$\begin{array}{l}\operatorname{Witness}(n,x)\\\begin{array}{ll}& \textbf{Note}.\space\text{“witness”意为“证人”。同时需要满足}\space x\in [1,n-1]\text{。}\\1&\textbf{if}\space x^{n-1}\bmod n\neq 1\space\textbf{or}\space (x\neq 1\space\mathbf{and}\space x\neq n-1\space\textbf{and}\space x^2\bmod n=1)\\2&\qquad\textbf{return}\space \textbf{True}\\3&\textbf{else}\\4&\qquad\textbf{return}\space\textbf{False} \end{array}\end{array}$$

等一下，我们还没有榨干 $x$ 的用途。注意到快速幂计算 $x^{n-1}$ 可以变成先找到最大的 $b$ 使得 $2^b\mid n-1$，然后设 $n=m2^b$，那么先求 $x^m$，然后反复平方。如果发现平方之前不是 $\pm 1$ 但是平方之后得到了 $1$，就可以直接返回 $\textbf{True}$（找到了一个非平凡平方根，这里不一定是 $x$）。

然后想一想怎么写更简单。我们可以对 $g=x^m$ 进行反复平方，每次循环开始如果 $g\equiv-1$ 就返回 $\textbf{False}$ 并退出（可以预测到接下来平方一次之后 $g$ 会变成 $1$ 然后就一直是 $1$ 了）。否则 $g\gets g^2\bmod n$。这个过程总共持续 $b$ 次。如果 $b$ 次之内没有退出，则返回 $\textbf{True}$。

$$\begin{array}{l}\operatorname{Witness}(n,x)\\\begin{array}{ll}1&\text{calculate the maximum }b\text{ that }2^b\mid n-1\text{ and }m=\dfrac{n-1}{2^b}\text{.}\\2&g\gets x^m\bmod n\\3&\textbf{if}\space g=1\\4&\qquad\textbf{return}\space\textbf{False}\\5&\textbf{repeat}\space b\space\textbf{times}\\6&\qquad\textbf{if}\space g=n-1\\7&\qquad\qquad\textbf{return}\space\textbf{False}\\8&\qquad g\gets g^2\bmod n\\9&\textbf{return}\space\textbf{True}\end{array}\end{array}$$


Miller-Rabin 就可以这么写。

$$\begin{array}{l}\operatorname{Miller-Rabin}(n)\\\begin{array}{ll}1&\textbf{const}\space k=20\\&\textbf{Note}.\space k\space \text{is the  number of tests.}\\2&\textbf{repeat}\space k\space\textbf{times}\\3&\qquad p=\operatorname{Random}(1,n-1)\\&\qquad\textbf{Note}.\space\operatorname{Random}(l,r)\space\text{产生一个}\space[l,r]\space\text{中的随机数}\\4&\qquad\textbf{if}\space\operatorname{Witness}(n,p)\\5&\qquad\qquad\textbf{return}\space\textbf{False}\\6&\textbf{return}\space\textbf{False}\end{array}\end{array}$$

然后我们再证明它的正确性。

定理：如果 $n$ 是一个奇合数，那么 $n$ 至少有 $\dfrac{n-1}{2}$ 个证据。众所周知，算法导论上花费了超过一页进行证明的定理都不是好证明的东西（并查集的时间复杂度都只用了四页）。所以，这里不进行证明。

而显然如果 $n$ 是偶数那么非证据一定要是奇数。

所以，只要 $n$ 是合数，那么证据的个数一定 $\ge \dfrac{n}{2}$。也就是说，随机选择一个数字，有 $\dfrac{1}{2}$ 的概率随出证据。这就证明了，Miller-Rabin 的错误率是 $2^{-k}$，其中 $k$ 是试验次数。比如，$k=20$ 时出错率约为 $9.53\times 10^{-7}$（也就是 $0.0000953\%$）。

### 理性一点？

如果我们要变成确定性算法，那自然是控制随机数的选择了。实际上，对于 $[1,2^{32})$ 范围内的数字，选择 $\{2,7,61\}$ 就可以确定所有数字了。对于 $[1,2^{64})$，选择 $\{2,325,9375,28178,450775,9780504,1795265022\}$ 即可。

什么，你记不住 $\{2,325,9375,28178,450775,9780504,1795265022\}$？那也可以退而求其次，选择前 $12$ 个质数 $\{2,3,5,7,11,13,17,19,23,29,31,37\}$ 也可以（也能确定 $[l,2^{64})$ 所有数字的素性）。

要注意的一点是，在 $\operatorname{Witness}$ 函数中应当判定是否有 $n=x$。如果是，那么显然不仅不是证据，还满足 $n$ 是质数。需要在 $\operatorname{Witness}$ 中特判。

### 代码

注意一个事，如果 $n$ 是 C++ 中 `long long` 范围的那么如果乘法不使用龟速乘那么会爆炸。所以可以使用 `__int128`（当然用龟速乘也可以）。

:::info[Miller-Rabin]

```cpp
__int128 qpow(__int128 x, __int128 y, __int128 p); // 快速幂怎么实现不用我说了吧

constexpr unsigned basis[15] = {0, 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37};

bool is_probab_prime(long long n)
{
    unsigned k = 0, m = n - 1;
    while(m % 2 == 0)
    {
        k++;
        m /= 2;
    }
    auto chkone = [&](long long x)
    {
        if(x==n) return true;
        __int128 g = qpow(x, m);
        if(g == 1) return true;
        for(int i=1;i<=k;i++)
        {
            if(g == n - 1) return true;
            g = g * g % n;
        }
        return false;
    };
    for(int i=1;i<=12;i++)
    {
        if(!chkone(basis[i])) return false;
    }
    return true;
}
```

:::

代码是十分简洁的。

### 时间复杂度

显然是 $O(k\log n)$，其中 $k$ 是试验次数。

## Pollard-rho

Pollard-rho 和 Miller–Rabin 一样，充满人类智慧。

Pollard-rho 用于求出一个合数 $n$ 的某个非平凡因子。不一定是质因子，所以如果要进行质因子分解还需要继续递归下去。

Pollard-rho 的期望时间复杂度为 $O(\sqrt p)$，其中 $p$ 是 $n$ 的最小质因子。很奇怪吧，Pollard-rho 不求最小质因子，但是期望时间复杂度和最小质因子有关。所以需要注意的是，如果 $n$ 是质数，那么 $p=n$，复杂度就是 $O(\sqrt n)$，和朴素算法无异。所以需要先用 Miller–Rabin 判断一遍是否为质数。如果不是质数，那么显然其最小质因子是 $\le \sqrt{n}$ 的，所以期望时间复杂度为 $O(\sqrt[4]{n})$。

Pollard-rho 非常非常玄学。它既不能保证分解成功，也不能保证运行时间。尽管你可以多次尝试分解，但是一次分解失败时间复杂度就是 $O(\sqrt n)$ 的，直接就炸了。并且，它为了速度进行了非常多的优化，甚至牺牲了一定的正确性（这样的结果是，可能复杂度都会退化）。但是，它在实际中运行的效果非常好。

首先，我们有一种玄学的求解方式。多次随机一个数字 $x\in [2,n)$，求 $\gcd(x,n)$。如果得到大于 $1$ 的数字，那么我们就得到了一个因子！

这个东西假完了，设其最小质因子为 $p$，那么随机出 $p$ 的倍数就可以了。所以期望时间复杂度是……$O(p)$。好吧，其实就是 $O(\sqrt n)$，和 $O(\sqrt p)$ 还差得远呢。顺便说一句，这种复杂度有“最小质因子”的一般都可以设 $n$ 是某个质数的平方来卡满。

什么情况下会引入根号？好吧，其实挺多的。但是 Pollard-rho 的核心原理是生日悖论。

生日悖论：不停随机 $[1,n]$ 的整数，第一次随机出两个相同的（比如分别随机出 $1,4,5,3,4$ 就是在第 $5$ 次出现相同）数字的期望随机次数为 $O(\sqrt n)$。更精确地，是 $\sqrt{\dfrac{\pi n}{2}}-\dfrac{1}{3}+o(1)$。

如果我们可以构建一个模 $p$（为 $n$ 的最小质因子）的随机数列，那么我们就可以通过找到两个相同的元素 $x_i\equiv x_j\pmod p$ 来找出 $n$ 的一个因子（$\gcd(x_i-x_j,n)$）了！并且，期望出现重复的数字个数是 $O(\sqrt p)$ 的！

额，其实我们不知道 $p$。我们最多只能构建一个模 $n$ 的随机数列。

如果两两作差求和 $n$ 的 $\gcd$，那时间复杂度就爆了。如果随机选取一些求 $\gcd$，那正确性就假完了。甚至你无法遍历整个环，因为那样复杂度是 $O(\sqrt n)$ 的。我们考虑从随机数列上做文章。

通常采用的迭代方式是 $x_{i+1}=(x_i^2+c)\bmod n$，其中 $c$ 是某个常数。这样有一个好处就是如果 $p\mid (x_i-x_j)$，那么 $x_{i+1}-x_{j+1}=x_i^2-x_j^2=(x_i-x_j)(x_i+x_j)$，所以 $p\mid (x_{i+1}-x_{j+1})$！

也就是说，相同距离的数对，只需要检查一个！

考虑 Floyd 判环。两个人分别从起点开始，一个人一次移动 $1$ 位，另一个移动 $2$ 位，那么显然每次它们的差距会增加 $1$。但是这样会在环长的时间复杂度内进入循环，也就是两个人的位置重新变得相同。此时分解失败。如果两个人某个时刻的查值和 $n$ 的 $\gcd$ 超过 $1$，那么就找到了一个非平凡因子。

那么考虑最小因子 $p$。期望下 $O(\sqrt p)$ 步之内就可以得到一个是 $p$ 的倍数的因子，所以期望时间复杂度最多是 $O(\sqrt p)$ 的！实际上如果 $n=p^2$ 那确实就是 $O(\sqrt p)$。

解决了！

等一下如果分解失败那不就是 $O(\sqrt n)$ 了吗？但是实际上分解失败的概率非常小，可以忽略不计（和 $O(\sqrt n)$ 可以抵消）。

### 优化

两个优化。

Brent 判环：在第 $k$ 轮（从 $0$ 开始），首先 $j$ 迭代 $2^k$ 步，每一步检查 $i$ 和 $j$。如果一只都不满足，那么赋值 $i\gets j$，然后开始下一轮。

看起来假完了，但实际上是对的。这样的移动次数甚至永远不大于 Floyd 判环。平均情况下能减少 $24\%$ 的检查次数。

“分块”$\gcd$（或称倍增优化）：按照上面的方法，有一个 $\gcd$，复杂度多一个 $\log$。那么我们不是每一步就判断一次，而是分成若干块，每个块长度为 $O(\log)$。然后对于每个块，求块中所有数字（$x_i-x_j$）的乘积（对 $n$ 取模），然后和 $n$ 求 $\gcd$。实际上这个 $O(\log)$ 通常设为一个常数。

### 代码

使用 Brent 判环和倍增优化。

:::info[代码]
```cpp
#include <cstdio>
#ifndef ONLINE_JUDGE // 这份代码中所有位于 #ifndef ONLINE_JUDGE 和 #endif 之间的部分都可以不管，是用于本地调试的。
#include <__msvc_int128.hpp> // 本地不支持 __int128，刚好 MSVC 有 __msvc_int128.hpp，定义了 _Signed128，可以使用。
#define __int128 _Signed128
#endif

using namespace std;

__int128 qpow(__int128 x, __int128 y, __int128 mod)
{
    __int128 p = x % mod, ans = 1;
    do
    {
        if (y & 1) ans = ans * p % mod;
        p = p * p % mod;
    } while (y >>= 1);
    return ans;
}

unsigned basis[] = { 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37 };

bool is_probab_prime(__int128 n)
{
    long long k = 0;
    __int128 m = n - 1;
    while (1 ^ m & 1) // 什么花式写法，但是其实是我不知道 ^ 和 & 的优先级哪个高，但是你发现不管哪个更高结果都是对的。
    {
        k++;
        m >>= 1;
    }
    auto chkone = [&](__int128 p)
        {
            if (n == p) return true;
            p = qpow(p, m, n);
            if (p == 1) return true;
            for (int i = 1; i <= k; i++)
            {
                if (p == n - 1) return true;
                p = p * p % n;
            }
            return false;
        };
    for (unsigned x : basis)
    {
        if (!chkone(x)) return false;
    }
    return true;
}

__int128 gcd(__int128 x, __int128 y) { return y == 0 ? x : gcd(y, x % y); }
unsigned pool[] = { 956008337,1070185313,1097705111,1083875081,930518497,948253391,1023815263,905918851,969311227,1074002201,934286407,1009093507,981354329,1040723269,1076316677,1090025219 }; // 使用 Python 生成随机质数.jpg
unsigned primes[] = { 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37 };

long long pollard_rho_inner(long long n, __int128 c)
{
    for (int x : primes)
    {
        if (n % x == 0) return x;
    }
    __int128 alpha = 1, beta = 1;
    int ctt = 0;
    constexpr unsigned big_banana = 31; // 神金 B.B.
    for (int i = 1;; i <<= 1)
    {
        __int128 mul = 1;
        for (int j = 1; j <= i; j++)
        {
            beta = (beta * beta % n + c) % n;
            auto dd = beta > alpha ? beta - alpha : alpha - beta;
            mul = mul * dd % n;
            if (alpha == beta) return n;
            if (++ctt == big_banana)
            {
                ctt = 0;
                auto res = gcd(mul, n);
                if (res > 1) return (long long)res;
                mul = 1;
            }
        }
        auto res = gcd(mul, n);
        if (res > 1) return (long long)res;
        alpha = beta;
    }
}
long long pollard_rho(long long n)
{
    for (auto x : pool)
    {
        auto res = pollard_rho_inner(n, x);
        if (res != n) return res;
    }
    return -1;
}

int main()
{
#ifndef ONLINE_JUDGE // 测试是否都能成功分解。
    for (int i = 2; i <= 10000000; i++)
    {
        if (is_probab_prime(i)) continue;
        long long res = pollard_rho(i);
        if (res == i || res == -1)
        {
            printf("%d\n", i);
        }
    }
    return 0;
#endif
    int t;
    scanf("%d", &t);
    while (t--)
    {
        long long n;
        scanf("%lld", &n);
        if (is_probab_prime(n)) printf("Prime\n");
        else
        {
            auto calc = [&](auto self, long long n) -> long long
            {
                if (n <= 1 || is_probab_prime(n)) return n;
                long long g = pollard_rho(n);
                while (n % g == 0) n /= g;
                long long r = self(self, g);
                while (n % r == 0) n /= r;
                long long p = self(self, n);
                return r > p ? r : p;
            };
             printf("%lld\n", calc(calc, n));
        }
    }
    return 0;
}
```
:::

:::info[彩蛋]

```cpp
#include<bits/stdc++.h>

using namespace std;

char p[1024];

int main()
{
    int t;
    cin >> t;
    stringstream w;
    for (int i = 1; i <= t; i++)
    {
        long long n;
        cin >> n;
        w << "factor " << n << " ; ";
    }
    FILE* fp = popen(w.str().c_str(), "r");
    for (int i = 1; i <= t; i++)
    {
        fgets(p, 1024, fp);
        int le = strlen(p);
        p[le++] = ' ';
        vector<string> c;
        string buc;
        for (int i = 0; i < le; i++)
        {
            if (p[i] == ' ')
            {
                c.push_back(buc);
                buc = "";
            }
            else buc += p[i];
        }
        if (c.size() == 2) cout << "Prime\n";
        else
        {
            c.back().pop_back();
            cout << c.back() << '\n';
        }
    }
    return 0;
}
```

:::