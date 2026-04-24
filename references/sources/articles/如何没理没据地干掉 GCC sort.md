没理没据是因为在数据量 $5\times 10^{6}$ 左右时才有明显效果．

---

大家一般都知道 C++ STL 中的 `sort` 函数是基于快速排序算法的，实际上，这个说法不太准确．

更准确的一些解析可以看这篇文章 [浅析 std::sort、std::stable_sort 和 std::partial_sort 的实现细节](https://www.luogu.com.cn/article/ut9ko5n2)．

下面我重复一些要用到的知识．

---

首先，与 C 中的 `qsort` 不同，C++ 明确规定 `sort` 的复杂度必须为 $\mathcal O\left(N\log N\right)$；更准确地说，令 $N$ 为 `last - first`，则最多进行 $\mathcal O\left(N\log N\right)$ 次比较．

因此，想要卡掉 `sort`，必须从常数入手．

---

其次，`sort` 要求行为是确定性的，这意味着它不会调用随机函数．

我们熟知快速排序算法平均情况下的复杂度 $\mathcal O(N\log N)$ 依赖于 `pivot` 可以近似地将序列平均划分，而确定性地选择 `pivot` 使得一定可以构造出序列，让划分达到最劣．

为了保证时间复杂度，GCC 的实现采用了一种叫做内省排序的方法，这个方法规定快速排序的递归最多进行 $\mathcal O\left(\log N\right)$ 层，之后采用堆排序．另外，这个值的推荐取值为 $2\times \left\lfloor\log_2 N\right\rfloor$．

此外，GCC 的实现中，使用选择排序进行最终的整理，不过这个细节不太重要．

---

等等！为什么大家都认为快速排序的常数远小于堆排序，使得 GCC 会优先进行快速排序呢？

:::info[解释]
学过朴素矩阵乘法的小朋友可能记得，仅仅交换循环变量的枚举顺序，在答案保持正确的前提下，运行时间可能存在 $2$ 倍左右的差距．

这涉及到硬件层面的一些知识，简单来说，CPU 计算速度是非常快的，远快于内存的读写速度；因此，为了使得数据读写速度与 CPU 计算速度相匹配，人们设计了各级缓存，缓存离 CPU 越近，读写速度越快，单位容量也就越贵，因此容量也越小．

计算机操作内存时，如果这部分数据不在缓存中，会先把内存中的数据复制到各级缓存中，尽可能加快读写速度．因此，操作内存地址相近的数据，有利于计算机更少地将内存复制到缓存，从而加快程序的运行速度．

熟知快速排序算法中常常对临近的数据进行比较（尤其在递归树靠近叶子的部分）；而堆排序操作堆的时候数据通常距离很远，因此快速排序的平均表现远强于堆排序．
:::

---

那么，堆排序在什么情况下运行时间最慢呢？

通过简单的测试可以发现，对于升序、降序和乱序数列，堆排序在乱序上表现最差，我猜测这可能与分支预测相关，这里不展开了．

---

因此，我们的思路非常明确，即通过构造数列使得快速排序部分选择较差的 `pivot`，从而使得序列主要由堆排序算法进行排序．

这里不得不提及 GCC 选择 `pivot` 的具体方式了，简单来说，GCC 从 `first + 1`、`mid`、`last - 1` 三个位置中选择中位数，交换到 `first` 位置，作为 `pivot`．这里数列的范围是 `[first, last)`，`mid = first + (last - first) / 2`．

这个方法完全可以看作特判掉了序列基本有序的情形．

为了简化讨论，我们只需要使 `first + 1` 位置上的数最小，`mid` 位置上的数次小，`last - 1` 位置上的数最大即可．

剩余未确定的数随机排列即可．

:::success[参考代码]

```cpp
#include <algorithm>
#include <chrono>
#include <iostream>
#include <limits>
#include <random>
#include <set>
#include <vector>
using namespace std;

struct Item {
  static constexpr size_t null = numeric_limits<size_t>::max();

  size_t index, value;

  bool operator<(const Item &other) const {
    if (value == null) return true;
    if (other.value == null) return false;
    return value < other.value;
  }
};

vector<size_t> anti_sort(size_t len) {
  static constexpr int _S_threshold = 16;

  vector<size_t> result(len);
  if (len == 0) return result;

  vector<Item> items(len);
  for (size_t i = 0; i < len; ++i)
    items[i].index = i, items[i].value = Item::null;

  size_t lvalue = 0;
  size_t depth_limit = __lg(items.size()) * 2;
  auto first = items.begin(), last = items.end();
  for (; last - first > _S_threshold; first += 2, lvalue += 2) {
    if (depth_limit-- == 0) break;
    auto mid = first + (last - first) / 2;
    next(first)->value = lvalue;
    mid->value = lvalue + 1;
    prev(last)->value = lvalue + (last - first - 1);
    swap(*first, *mid);
  }

  set<size_t> unused_set;
  for (size_t i = 0; i < len; ++i)
    unused_set.emplace_hint(unused_set.end(), i);
  for (const auto &item : items) unused_set.erase(item.value);

  static mt19937 rng(
      chrono::steady_clock::now().time_since_epoch().count());
  vector<size_t> unused(unused_set.begin(), unused_set.end());
  shuffle(unused.begin(), unused.end(), rng);
  for (size_t i = 0; i < len; ++i)
    if (items[i].value == Item::null)
      items[i].value = unused.back(), unused.pop_back();

  for (size_t i = 0; i < len; ++i)
    result[items[i].index] = items[i].value;
  return result;
}

int main() {
  size_t len;
  cin >> len;
  auto result = anti_sort(len);
  cout << len << "\n";
  for (size_t i = 0; i < len; ++i)
    cout << result[i] << " \n"[i + 1 == len];
}
```

:::

在 $1\times 10^{7}$ 的量级下大约可以拉开 $3.6$ 倍的差距．

在本机[^1]上：

| 数据规模             | 构造数据 ($\mathrm{ms}$) | 有序数据 ($\mathrm{ms}$) | 乱序数据 ($\mathrm{ms}$) |
|:----------------:|:--------------------:|:--------------------:|:--------------------:|
| $1\times 10^{6}$ | $173$                | $10$                 | $64$                 |
| $3\times 10^{6}$ | $560$                | $32$                 | $206$                |
| $5\times 10^{6}$ | $1100$               | $56$                 | $345$                |
| $8\times 10^{6}$ | $2016$               | $89$                 | $570$                |
| $1\times 10^{7}$ | $2651$               | $122$                | $725$                |

~~因此建议是在调用 sort 前保证序列有序！~~

~~如果没法保证有序可以 shuffle 一遍．~~

[^1]: Windows 11，GCC，O2 优化，处理器 AMD Ryzen 7 8745HS w/ Radeon 780M Graphics (3.80 GHz)，内存 16.0 GB，测量 $10$ 次取平均值，未测量读入等开销
