- 本文已同步到 [CSDN](https://blog.csdn.net/XieYilang/article/details/145502872)

## **chrono** 库简介

C++ 的 `chrono` 库是 C++11 引入的一个用于处理时间和日期的标准库，它提供了一套丰富的工具来测量时间间隔、执行时间点的计算以及处理日期和时间。

`chrono` 库主要有以下几个功能：

- 时间点：表示了一个特定的时间点，就比如当前的时间；

- 时间段：表示一段时间的长度，如一秒，一分钟，一小时，一天等；

- 时钟：是时间点和持续时间的来源，运行库提供了几种不同的时钟，例如系统时钟、高分辨率时钟等。

接下来分别看一下 `chrono` 库中三个不同的时钟：

| 时钟                      | 描述                   |
| ----------------------- | -------------------- |
| `system_clock`          | 表示你的电脑当前的系统时间。       |
| `steady_clock`          | 比较稳定的时钟，不随电脑性能等因素变化。 |
| `high_resolution_clock` | 系统可用的最高精度时钟。         |

下面还有两个 `chrono` 库中常用的类：

| 类名           | 描述            |
| ------------ | ------------- |
| `time_point` | 时间点，表示一个具体时间。 |
| `duration`   | 时间段，表示一段时间。   |

接下来我们来看一下它们具体的用法。

## **chrono** 库函数与用法

### I. chrono 的头文件以及命名空间

首先在使用 `chrono` 库前，我们肯定需要导入 `chrono` 库：

```cpp
#include<chrono>
```

`chrono` 库还是 `std` 命名空间的一个**子命名空间**，所以我们在使用的时候也要在程序前面加上：

```cpp
using namespace std::chrono;
```

接下来，我们就可以愉快的写代码了。

### II. system_clock

`system_clock` 类表示系统时间，也就是说，不管你的系统是什么时间，这个类都会表示出你的系统时间。

#### steady_clock 的类成员

| 类成员             | 描述                                          |
| --------------- | ------------------------------------------- |
| `duration`      | 接下来要介绍到的时间段类，是 `system_clock` 类的子类，表示一个时间段。 |
| `from_time_t()` | 从 `time_t` 类导入时间。                           |
| `now()`         | 用于获取当前系统时间。                                 |
| `period`        | 表示一个时间单位的子类。                                |
| `time_point`    | 表示一个时间点的类，也是 `system_clock` 中的一个子类。         |
| `to_time_t()`   | 将 `time_point` 转化为 `time_t` 类型。             |

#### 获取与输出当前时间

##### 示例代码

```cpp
#include<bits/stdc++.h>
using namespace std;
using namespace std::chrono;
int main(){
    system_clock::time_point now=system_clock::now();
    time_t tp1=system_clock::to_time_t(now);
    //I
    string str_time=ctime(&tp1);
    cout<<str_time;
    //II
    tm* tpm=localtime(&tp1);
    cout<<put_time(tpm,"%c")<<endl;
    //III
    char buf[80];
    strftime(buf,sizeof(buf),"%Y/%m/%d %H:%M:%S",tpm);
    printf("%s",buf);

    return 0;
}
```

运行结果：

```cpp
Fri Feb 07 18:16:55 2025
02/07/25 18:16:55
2025/02/07 18:16:55
--------------------------------
Process exited after 0.0337 seconds with return value 0
请按任意键继续. . .
```

##### 代码解析

上面的代码展示了获取系统时间方法以及三种不同的输出时间的方式，下面给出了每行代码具体的作用与解释：

---

```cpp
system_clock::time_point now=system_clock::now();
```

`time_point` 类不能单独使用，必须作为其他类中的子类使用。`system_clock::now` 函数返回了一个 `system_clock::time_point` 类，表示当前的时间。

---

```cpp
time_t tp1=system_clock::to_time_t(now);
```

`system_clock_to_time_t` 函数用于将一个时间点类转化为 `time.h` 中的 `time_t` 类。

---

```cpp
string str_time=ctime(&tp1);
cout<<str_time;
```

`ctime` 函数将 `time_t` 类型转化为 `string` 并**自带换行**。它的使用方法非常简单，但是**只能以固定格式输出**，比较不灵活。

---

```cpp
tm* tpm=localtime(&tp1);
cout<<put_time(tpm,"%c")<<endl;
```

---

`put_time` 函数输出 `tm` 类型的时间，格式也是固定的。

```cpp
char buf[80];
strftime(buf,sizeof(buf),"%Y/%m/%d %H:%M:%S",tpm);
printf("%s\n",buf);
```

这个函数就比较灵活，可以以任意格式输出时间。

---

### III. steady_clock

`steady_clock` 是一个“稳定的时间”，不会因为电脑算力或者其他意外的因素而改变。应该会有同学用过 `windows.h` 库中的 `Sleep` 函数用于等待一定的时间，你会发现在不同的电脑上 `Sleep` 的等待时间可能会不一样，因为这个函数依赖于系统的速度。

而 `steady_clock` 就很好的解决了这个问题。接下来我们来看一下如何去使用这个类。

#### steady_clock 的类成员

`steady_clock` 类中的类成员大多与 `system_clock` 中的相似。

| 类成员          | 描述         |
| ------------ | ---------- |
| `duration`   | 表示时间段的子类。  |
| `now()`      | 获取当前时间。    |
| `period`     | 表示时间单位的子类。 |
| `time_point` | 表示时间点的子类。  |

#### 计算经过的时间

##### 示例代码

```cpp
#include<bits/stdc++.h>
using namespace std;
using namespace std::chrono;
int main(){
    steady_clock::time_point start=steady_clock::now();
    getchar();
    steady_clock::time_point end=steady_clock::now();
    cout<<(duration_cast<milliseconds>(end-start).count()/1000.0)<<endl;
    return 0;
}
```

我们运行上面的程序，等一会按下任意的一个键，就会输出程序开始到我们按下键盘的秒数。

```cpp
1.979

--------------------------------
Process exited after 2.012 seconds with return value 0
请按任意键继续. . .
```

##### 代码解析

```cpp
steady_clock::time_point start=steady_clock::now();
```

记录开始的时间点，使用 `steady_clock::now()` 函数。

---

```cpp
getchar();
```

获取键盘输入。

---

```cpp
steady_clock::time_point end=steady_clock::now();
```

记录结束时间点。

---

```cpp
cout<<(duration_cast<milliseconds>(end-start).count()/1000.0)<<endl;
```

这一行代码是难点，我们来逐步解析。

首先，`duration_cast` 函数用于转换时间单位，他是一个模版函数，所以使用的语法可能有点奇怪。

两个 `<>` 中间填的是你要转化成的时间单位，`milliseconds` 是库里面已经规定好的一个特定的时间单位，意为毫秒。

下面列举了一些其他的库中已规定的时间单位：

- `hours`：小时；

- `minutes`：分钟；

- `seconds`：秒；

- `milliseconds`：毫秒；

- `nanoseconds`：微妙。

所以上面的 `duration_cast<milliseconds>(end-start).count()` 返回 `end-start` 时间段所代表的毫秒数。

整行代码的意思就是输出 `(end-start)` 经过的秒数。

### IV. high_resolution_clock

`high_resolution_clock` 是一个高精度的时钟，也是**计算机内精度最高**的时钟。其用法与 `steady_clock` 差不多，只是精度有所不同，所以此处也不多赘述。

### V. duration

一个表示时间段长度的类，使你可以自定义时间单位，不再局限于库中自带的 `hours`、`minutes` 时间单位。

#### duration 的类成员

| 类成员      | 描述          |
| -------- | ----------- |
| `max()`  | 获取可能的最大时间段。 |
| `min()`  | 获取可能的最小时间段。 |
| `period` | 时间单位类。      |
| `zero()` | 代表时间段 0。    |

#### 自定义时间单位

如下面的示例，我们可以自由地自定义事件单位：

```cpp
duration< int > _5s(5);
duration< double,ratio<3600> > _6min(0.1);
duration< double,ratio<1,500> > _1ms(0.5);
minutes _2min(2);
milliseconds _1s(1000);
```

看上去很复杂，所以让我来具体讲一下如何正确地使用 `duration`。

---

```cpp
duration<int> _5s(5);
```

首先，`duration` 是一个模版类，上面代码中的 `int` 表示参数类型。

比如，要定义五秒就填 `int`，半秒就填 `double` 或 `float`，以此类推。

上面的 `_5s` 是时间单位的名称，这样我们就可以像上面的 `milliseconds` 一样使用了。

再后面的 `(5)`，代表了 5 个默认单位，也就是五秒。

所以，我们就定义了一个名为 `_5s`，代表五秒的时间单位了。

---

```cpp
duration< double,ratio<3600> > _6min(0.1);
```

这里又多出了一个新的模版参数。`chrono` 中默认的时间单位是秒，所以这里的 `ratio<3600>` 就相当于是一个临时的时间单位，表示 3600 秒，一个小时。

上面的 `_6min(0.1)` 定义了一个时长为 0.1 个 `ratio<3600>`，也就是六分钟的时间单位。

---

```cpp
duration< double,ratio<1,500> > _1ms(0.5);
```

如果在 `ratio` 里填入两个参数则又是不同的意思，上面的 `ratio<1,500>` 代表的是 5001​ 个默认单位，也就是 5001​ 秒，2 毫秒。

所以后面的 `_1ms(0.5)` 定义了 0.5 个 `ratio<1,500>`，为一毫秒。

---

```cpp
minutes _2min(2);
```

这里的 `minutes` 代表着一分钟的默认单位，所以上面代码中的 `_2min` 是一个代表两分钟的时间段。

---

```cpp
milliseconds _1s(1000);
```

同上面一样，这里定义了一个 `_1s` 的时间段对象，而它代表的时间是 1000 个 `milliseconds`，也就是一秒。

#### duration 对象的操作

有时候我们需要将几个时间段相加或进行比较，这时候我们就可以使用 `duration` 对象的操作了。

| 操作       | 代码                            | 描述                                          |
| -------- | ----------------------------- | ------------------------------------------- |
| 相加       | `duration1+duration2`         | 将两个时间段相加，返回一个新的 `duration` 对象。              |
| 相减       | `duration1-duration2`         | 将两个时间段相减，返回一个新的 `duration` 对象。              |
| 相乘       | `duration1*n` 或 `n*duration1` | 将时间段与一个实数相乘，返回一个新的 `duration` 对象。           |
| 相除       | `duration1/n`                 | 将时间段与一个实数相除，返回一个新的 `duration` 对象。           |
| 判断相等     | `duration1==duration2`        | 判断两个 `duration` 对象是否相等。                     |
| 判断不等     | `duration1!=duration2`        | 判断两个 `duration` 对象是否不等。                     |
| 判断是否大于   | `duration1>duration2`         | 判断第一个 `duration` 对象是否大于第二个 `duration` 对象。   |
| 判断是否小于   | `duration1<duration2`         | 判断第一个 `duration` 对象是否小于第二个 `duration` 对象。   |
| 判断是否大于等于 | `duration1>=duration2`        | 判断第一个 `duration` 对象是否大于等于第二个 `duration` 对象。 |
| 判断是否小于等于 | `duration1<=duration2`        | 判断第一个 `duration` 对象是否小于等于第二个 `duration` 对象。 |

#### 时间单位的转换

如果我们现在有一个 `hours` 对象，想要把他转换为 `seconds`，则可以通过下面的代码实现：

```cpp
hours _1h(1);
seconds _1s=duration_cast<seconds>(_1h);
```

这样 `_1s` 就是一个表示一秒的时间段对象了。

### VI. time_point

一个表示**时间点**的类，通常作为子类出现，不能单独使用。

#### time_point 的类成员

| 类成员        | 描述          |
| ---------- | ----------- |
| `duration` | 表示时间段的子类。   |
| `max()`    | 获取可能的最大时间点。 |
| `min()`    | 获取可能的最小时间点。 |
| `period`   | 时间单位类。      |

### VII. chrono 库在实际中的应用

依据上面的所有知识点，我们便可以使用 `chrono` 库写出一个简易的倒计时程序。

```cpp
#include<bits/stdc++.h>
using namespace std;
using namespace std::chrono;
int main(){
    printf("请输入需要等待的秒数：\n");
    double dur;
    scanf("%lf",&dur);
    steady_clock::time_point start=steady_clock::now();
    while((duration_cast<milliseconds>(steady_clock::now()-start).count())/1000.0<=dur);
    printf("时间到！！！\n按任意键退出. . . \n");
    getchar();
    return 0;
}
```

## 总结

至此，我们已经学会了 `chrono` 库的基本操作。我们还需要在实际应用中慢慢提高我们使用 `chrono` 库的能力，这样才能让我们熟练地运用 `chrono` 库和理解它的本质。

## 参考资料

1. [CSDN [C++] C++ std::chrono 库使用](https://blog.csdn.net/FL1768317420/article/details/136346786)作者：[赵闪闪 168.](https://blog.csdn.net/FL1768317420)

2. [CSDN C++11 计时器：chrono 库介绍](https://blog.csdn.net/qq_44918090/article/details/128571499)作者：[森明帮大于黑虎帮](https://blog.csdn.net/qq_44918090)
