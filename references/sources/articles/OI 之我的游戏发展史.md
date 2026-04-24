我一直对编游戏这件事很感兴趣。

## 壹：Python 打字方式改进

在一年半之前，我学的是 Python。

当时觉得很有意思，没有什么压力感。

我记得第一课的时候我就自己打了个文字小游戏，然后觉得自己无敌了。。。

不过一开始弱弱的我对在屏幕上输出文字的方式很烦恼，直接输出不仅不好看，还要停几秒作为缓冲时间，如果看的快会觉得慢，看的慢又会觉得来不及。

怎么办呢？其实很简单，可以像 AI 那样一个字一个字打，写个函数控制好间隔的时间，用的时候调用函数，参数是要输出的字符串。

下面有一个示例，可以实现这样的效果，用的是最简单的循环 + sleep 函数。

```python
import time

def Print(s,huanhang = 1):
    for i in s:
        print(i,end = '')
        time.sleep(0.02)
    if huanhang == 1:
        print()
```

但如果想要暂停呢？这时就可以用 keyboard 库监听全局按键，需要稍稍复杂一点。如果有这个需求，那么可以使用。

下面是一个按空格暂停的示例。

```cpp
import time
import keyboard
import threading

is_paused = False
pause_lock = threading.Lock()

def monitor_space():
    global is_paused
    while True:
        keyboard.wait('space')  
        with pause_lock:
            is_paused = not is_paused 
        if is_paused:
            print("\n[输出已暂停，按空格键继续]", end='', flush=True)
        else:
            print("\r                                   ", end='', flush=True)
            print("\r", end='', flush=True)  

def Print(s, huanhang=1):
    global is_paused

    thread = threading.Thread(target=monitor_space, daemon=True)
    thread.start()

    for i in s:
        while is_paused:
            time.sleep(0.1)  

        print(i, end='', flush=True)
        time.sleep(0.02)  

    if huanhang == 1:
        print()
```

当然还有一些更厉害的，可以实现倒退功能。但是比较少见。可以用多线程并发控制实现。

下面是按 p 回退的实现。

```cpp
import time
import keyboard
import threading

current_pos = 0     
is_paused = False
running = True         
lock = threading.Lock()

def monitor_keys():
    global is_paused, current_pos, running
    while running:
        event = keyboard.read_event(suppress=False)  
        if event.event_type == keyboard.KEY_DOWN:
            with lock:
                if event.name == 'space':
                    is_paused = not is_paused
                    if is_paused:
                        print("\n[输出暂停，按空格继续]", end='', flush=True)
                    else:
                        print("\r", end='', flush=True) 
                elif event.name == 'p':
                    if current_pos > 0:
                        current_pos -= 1
                        print('\b \b', end='', flush=True)

def Print(s, huanhang=1):
    global current_pos, running, is_paused

    thread = threading.Thread(target=monitor_keys, daemon=True)
    thread.start()

    try:
        for i, char in enumerate(s):
            while is_paused:
                time.sleep(0.1)

            print(char, end='', flush=True)
            current_pos += 1
            time.sleep(0.05)  

        if huanhang == 1:
            print()
    except KeyboardInterrupt:
        pass
    finally:
        running = False  
        print("\n[输出结束]")
```

经过这样的实现，感觉看起来会比原来好很多。

## 贰：Python 模拟加载

既然是做游戏，肯定就要有真实感。

重要的，加载系统肯定少不了。毕竟大部分游戏都需要加载。我们通过 Python 可以手写一个简易的加载系统。

当然，我们就用恒定的数字来表示进度。比如用 3 的倍数。

接着我们可以认识一下 print('\x1bc')，它的作用是清空屏幕。

搭配起来，我们就得到了极其简单的代码：

```python
Print('开机中······')
jindu = [0,3,5,8,11,15,16,18,21,24,28,33,35,38,41,44,48,51,54,56,59,61,63,66,68,72,74,77,79,83,86,88,92,94,97,99,100]
for j in jindu:
    print('\x1bc')
    if j < 10:
        print('引擎中······')
    elif j < 30:
        print('网络加载中······')                       
    elif j < 45:
        print('资源加载中······')
    elif j < 75:
        print('程序加载中······')
    elif j < 90:
        print('选项加载中······')
    else:
        print('用户加载中······')
    print(f'[{j}%~~~~~]')
    time.sleep(random.uniform(0.2,0.3))
print('\x1bc')
Print('网络连接')
time.sleep(1)
print('\x1bc')
Print("开机成功！")
print('\x1bc')
```

## 叁：Python 内部游戏实现

这个就比较广泛了。大部分简单的游戏是问答体系。

但是如果自己输结果就太广泛了。所以可以用选择题的形式来进行问答。

你可以通过 time 库和 random 库来写很多东西，像计算题，概率抽奖之类的。

还有，如果要调用网址，可以用 webbrowser 来实现。

例如：

```python
webbrowser.open('https://cybermap.kaspersky.com/')
```

这里实现一个猜数小游戏：

```cpp
import random

def guess_number_game():
    number = random.randint(1, 100)
    guess_count = 0 
    print("欢迎来到猜数字小游戏！")
    print("我已经想好了一个 1 到 100 之间的整数，你来猜猜看吧！")

    while True:
        try:
            guess = int(input("请输入你的猜测："))
            guess_count += 1

            if guess < 1 or guess > 100:
                print("请输入 1~100 之间的整数！")
            elif guess < number:
                print("太小了！再大一点！")
            elif guess > number:
                print("太大了！再小一点！")
            else:
                print(f"恭喜你！猜对了！答案就是 {number}！")
                print(f"你一共猜了 {guess_count} 次。")
                break
        except ValueError:
            print("请输入一个有效的整数！")

    while True:
        play_again = input("还想再玩一次吗？(yes/no): ").strip().lower()
        if play_again == 'yes' or play_again == 'y':
            guess_number_game()
            break
        elif play_again == 'no' or play_again == 'n':
            print("谢谢游戏，再见！")
            break
        else:
            print("请输入 yes 或 no。")

if __name__ == "__main__":
    guess_number_game()
```

## 肆：Python 游戏结尾

这部分比较简单，或许你可以用 886 这种低级结尾，或者是关于游戏时长及成绩的。

这是其中一种比较规范的：

```python
time_end = time.time() #结束计时
time_c = time_end - time_start #运行所花时间
time_c = int(time_c)
time_w = time_c / 60;
Print("本次运行你用了" + str(time_w) + "分")
if time_w > 60:
    Print("你的运行时间过久,为保护青少年视力,运行一次时间不得超过60分钟！")
elif time_w > 50:
    Print("你的运行时间有一点久,以后注意！")
elif time_w > 40:
    Print("你的运行时间还不错,以后争取做得更好！")
else:
    Print("你的运行时间刚刚好,以后保持！")
```

## 伍：C++ 游戏中获取打出的字符

后来我学了 C++。在这个时候，我先制作了一个用了计时的小游戏，需要小猴编程里的库，具体请看代码：

```cpp
#include <ctime>
#include <iostream>
#include <xiaohoucode.h>
using namespace std;

int main()
{
    cout << "欢迎来到时间机器" << endl;
    sleep(1);
    cout << "你要设置什么？1倒计时 2闹钟：" << endl;
    int qe;
    cin >> qe;
    if(qe == 1)
    {
        cout << "请问你要定一个几分的倒计时？" << endl;
        int n;
        cin >> n;
        cout << "要设置秒吗？1要 2不要：" << endl;
        int m;
        cin >> m;
        if(m == 1)
        {
            cout << "要设置多少秒？" << endl;
            int y;
            cin >> y;
            if(y < 10)
            {
                cout << n << ":" << "0" << y << endl;
            }
            else
            {
                cout << n << ":" << y << endl;
            }
            sleep(1);
            clear();
            for(int i = y-1; i >= 0; i--)
            {
                if(i < 10)
                {
                    cout << n << ":" << "0" << i << endl;
                }
                else
                {
                    cout << n << ":" << i << endl;
                }
                sleep(1);
                clear();
            }
            for(int i = n-1; i >= 0; i--)
            {
                for(int j = 59; j >= 0; j--)
                {
                    if(j < 10)
                    {
                        cout << i << ":" << "0" << j << endl;
                    }
                    else
                    {
                        cout << i << ":" << j << endl;
                    }
                    sleep(1);
                    clear();
                }
            }
            cout << endl;
            cout << "倒计时结束！^_^" << endl;
        }
        else if(m == 2)
        {
            cout << n << ":" << "00" << endl;
            sleep(1);
            clear();
            for(int i = n-1; i >= 0; i--)
            {
                for(int j = 59; j >= 0; j--)
                {
                    if(j < 10)
                    {
                        cout << i << ":" << "0" << j << endl;
                    }
                    else
                    {
                        cout << i << ":" << j << endl;
                    }
                    sleep(1);
                    clear();
                }
            }
            cout << endl;
            cout << "倒计时结束！^_^" << endl;
        }
        else
        {
            cout << endl;
        }
    }
    if(qe == 2)
    {
        cout << "请问你要设置一个几分钟后的闹钟？" << endl;
        int x;
        cin >> x;
        cout << "要设置秒吗？1要 2不要：" << endl;
        int s;
        cin >> s;
        if(s == 1)
        {
            cout << "要设置多少秒？" << endl;
            int m;
            cin >> m;
            cout << "闹钟已定" << endl;
            for(int i = 1; i <= m; i++)
            {
                sleep(1);
            }
            for(int i = 1; i <= x; i++)
            {
                for(int i = 1; i <= 60; i++)
                {
                    sleep(1);
                }
            }
            cout << endl;
            cout << "闹钟到了！滴滴滴！" << endl;
        }
        else if(s == 2)
        {
            cout << "闹钟已定" << endl;
            for(int i = 1; i <= x; i++)
            {
                for(int i = 1; i <= 60; i++)
                {
                    sleep(1);
                }
            }
            cout << endl;
            cout << "闹钟到了！滴滴滴！" << endl;
        }
        else
        {
            cout << endl;
        }
    }
    return 0;
}
```

但后来决定制作关于上下左右跑酷的游戏。如何获取打出的字符呢？

这里就需要用到 _kbhit 和 getch() 了。_kbhit 用于检查当前是否有键盘输入，getch() 用来读入字符。两者配合 switch 语句即可实现内容。

下面是一部分：

```cpp
int getk;
while(1){
    if(_kbhit){
        getk=getch();
        break;
    }
}
switch(getk){
    case 'H':
        getk='w';
        break;
    case 'K':
        getk='a';
        break;
    case 'P':
        getk='s';
        break;
    case 'M':
        getk='d';
        break;
}
return getk;
```

这样就可以实现很多操作，比如暂停游戏，上下左右移动什么的。

## 陆：C++ windows.h库

在 windows.h 库，有几个好用的东西：

1. SetConsoleTextAttribute() 函数：
   
   Windows 提供了一个未公开在标准 windows.h 中的扩展函数：SetCurrentConsoleFontEx，可以用来改变控制台字体。但这需要：
   
   - 包含 <windows.h>
   - 链接 kernel32.lib
   - 使用 CONSOLE_FONT_INFOEX 结构
   
   例如：
   
   ```cpp
   #include <iostream>
   #include <windows.h>
   using namespace std;
   int main() {
       HANDLE hConsole = GetStdHandle(STD_OUTPUT_HANDLE);
       SetConsoleTextAttribute(hConsole, 12); 
       cout << "这是红色文字" << endl;
       SetConsoleTextAttribute(hConsole, 10); 
       cout << "这是绿色文字" << endl;
       SetConsoleTextAttribute(hConsole, 14);  
       cout << "这是黄色文字" << endl;
       SetConsoleTextAttribute(hConsole, 15); 
       cout << "这是白色文字（默认）" << endl;
       return 0;
   }
   ```

2. Sleep() 函数：
   
   可以停住一段时间。Sleep(1000) 可以暂停一秒。 在程序中很好用。

3. CreateWindowEx{} 函数：
   
   可以创建一个窗口，也很好用。
   
   例如：
   
   ```cpp
   HWND hwnd = CreateWindowEx(
      0,                  
      "MyGameClass",      
      "我的小游戏",          
      WS_OVERLAPPEDWINDOW,     
      CW_USEDEFAULT, CW_USEDEFAULT,
      800, 600,
      NULL, NULL, hInstance, NULL
   );
   ```

4. GetDC() 和 ReleaseDC() 函数：
   
   GetDC() 用于获取指定窗口的 DC，ReleaseDC() 用于释放 DC，两者配合 GDI 函数（如 Rectangle,Ellipse,TextOut）使用。
   
   例如：
   
   ```cpp
   HDC hdc = GetDC(hwnd);
   Ellipse(hdc, 100, 100, 200, 200);// 画一个椭圆（这里是圆）
   ReleaseDC(hwnd, hdc);    
   ```

另外还有一些画图形的函数，这里不再解释。

这些函数可以进行基础的绘画了，可以制作一些 2D 小游戏，例如坦克大战。只是感觉似乎能画的内容不多。

比如下面这个画房子的代码：

```cpp
#include <windows.h>

LRESULT CALLBACK WndProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam) {
    HDC hdc;
    PAINTSTRUCT ps;
    RECT rect;

    switch (msg) {
        case WM_PAINT:
            {
            hdc = BeginPaint(hwnd, &ps);

            GetClientRect(hwnd, &rect);

            FillRect(hdc, &rect, (HBRUSH) GetStockObject(WHITE_BRUSH));

            int x = (rect.right - 200) / 2; 
            int y = rect.bottom - 150;   
            HBRUSH hHouseBrush = CreateSolidBrush(RGB(220, 80, 80)); 
            SelectObject(hdc, hHouseBrush);
            Rectangle(hdc, x, y - 100, x + 200, y); 

            HPEN hRoofPen = CreatePen(PS_SOLID, 2, RGB(100, 30, 30));
            HBRUSH hRoofBrush = CreateSolidBrush(RGB(150, 40, 40));   
            SelectObject(hdc, hRoofPen);
            SelectObject(hdc, hRoofBrush);

            POINT roof[3] = {
                {x, y - 100},      
                {x + 100, y - 150}, 
                {x + 200, y - 100} 
            };
            Polygon(hdc, roof, 3);

            HBRUSH hDoorBrush = CreateSolidBrush(RGB(100, 60, 30)); 
            SelectObject(hdc, hDoorBrush);
            Rectangle(hdc, x + 80, y - 60, x + 120, y); 

            HBRUSH hWindowBrush = CreateSolidBrush(RGB(255, 255, 150)); 
            SelectObject(hdc, hWindowBrush);
            Rectangle(hdc, x + 30, y - 80, x + 70, y - 40); 
            Rectangle(hdc, x + 130, y - 80, x + 170, y - 40); 

            DeleteObject(hHouseBrush);
            DeleteObject(hRoofPen);
            DeleteObject(hRoofBrush);
            DeleteObject(hDoorBrush);
            DeleteObject(hWindowBrush);

            EndPaint(hwnd, &ps);
            break;
        }
        case WM_DESTROY:
            PostQuitMessage(0);
            break;

        default:
            return DefWindowProc(hwnd, msg, wParam, lParam);
    }
    return 0;
}

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
    const char CLASS_NAME[] = "SimpleHouseClass";

    WNDCLASS wc = {0};
    wc.lpfnWndProc = WndProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;
    wc.hCursor = LoadCursor(NULL, IDC_ARROW);
    wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1);

    if (!RegisterClass(&wc)) {
        MessageBox(NULL, "窗口类注册失败！", "错误", MB_ICONERROR);
        return 0;
    }

    HWND hwnd = CreateWindowEx(
        0,
        CLASS_NAME,
        "画一个简单的房子",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, 500, 400,
        NULL, NULL, hInstance, NULL
    );

    if (!hwnd) {
        MessageBox(NULL, "创建窗口失败！", "错误", MB_ICONERROR);
        return 0;
    }

    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);

    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return (int)msg.wParam;
}
```

## 柒：C++ 简单 windows.h 小游戏

我们可以尝试做一个坦克按空格发射炮弹的小游戏。

1. 创建窗口
   
   这部分可以保持不变，主要是设置窗口的基本属性，如标题、大小等。

2. 处理键盘消息（空格键）
   
   这部分也基本相同，主要是在检测到空格键按下时，创建新的炮弹对象。

3. 使用定时器实现动画
   
   通过 SetTimer 设置定时器，定期更新炮弹的位置并重绘画面。

4. GDI 绘图与颜色
   
   示例：
   
   背景色：使用浅蓝色代表天空，RGB 值为 (135, 206, 235)。
   
   坦克车身：灰色矩形，RGB 值为 (100, 100, 100)。
   
   坦克炮管：黑色矩形，RGB 值为 (0, 0, 0)。
   
   坦克履带：深灰色矩形，RGB 值为 (60, 60, 60)。
   
   炮弹：红色圆形，RGB 值为 (255, 0, 0)。
   
   我们可以通过 CreateSolidBrush 函数来创建这些颜色的画刷，并通过 SelectObject 将它们选入设备上下文（HDC）中，以便后续绘图函数使用。

5. 处理越界
   
   最后就是确保每次移动后检查炮弹是否超出屏幕顶部，以移除它们。

下面是代码：

```cpp
#include <windows.h>
#include <vector>

struct Shell {
    int x, y;
    Shell(int x, int y) : x(x), y(y) {}
};

const char CLASS_NAME[] = "TankGame";
HWND hwnd;
std::vector<Shell> shells;
int tankX = 0, tankY = 0;

LRESULT CALLBACK WndProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam) {
    HDC hdc;
    PAINTSTRUCT ps;
    RECT rect;

    switch (msg) {
        case WM_CREATE:
            SetTimer(hwnd, 1, 16, NULL);
            break;

        case WM_KEYDOWN:
            if (wParam == VK_SPACE) {
                shells.push_back(Shell(tankX + 50, tankY - 10));
            }
            break;

        case WM_TIMER:
            if (wParam == 1) {
                for (auto it = shells.begin(); it != shells.end(); ) {
                    it->y -= 5;
                    if (it->y < 0) {
                        it = shells.erase(it);
                    } else {
                        ++it;
                    }
                }
                InvalidateRect(hwnd, NULL, FALSE);
            }
            break;

        case WM_PAINT:
        {
            hdc = BeginPaint(hwnd, &ps);
            GetClientRect(hwnd, &rect);

            FillRect(hdc, &rect, CreateSolidBrush(RGB(135, 206, 235)));

            tankX = (rect.right - 100) / 2;
            tankY = rect.bottom - 30;

            HBRUSH hBodyBrush = CreateSolidBrush(RGB(100, 100, 100));
            SelectObject(hdc, hBodyBrush);
            Rectangle(hdc, tankX, tankY, tankX + 100, tankY + 30);

            HBRUSH hGunBrush = CreateSolidBrush(RGB(0, 0, 0));
            SelectObject(hdc, hGunBrush);
            Rectangle(hdc, tankX + 45, tankY - 20, tankX + 55, tankY);

            HBRUSH hTrackBrush = CreateSolidBrush(RGB(60, 60, 60));
            SelectObject(hdc, hTrackBrush);
            Rectangle(hdc, tankX - 5, tankY + 25, tankX + 105, tankY + 30);
            Rectangle(hdc, tankX - 5, tankY + 30, tankX + 105, tankY + 35);

            HBRUSH hShellBrush = CreateSolidBrush(RGB(255, 0, 0));
            SelectObject(hdc, hShellBrush);
            for (const auto& shell : shells) {
                Ellipse(hdc, shell.x - 5, shell.y - 5, shell.x + 5, shell.y + 5);
            }

            DeleteObject(hBodyBrush);
            DeleteObject(hGunBrush);
            DeleteObject(hTrackBrush);
            DeleteObject(hShellBrush);

            DeleteObject(GetStockObject(WHITE_BRUSH));

            EndPaint(hwnd, &ps);
            break;
        }
        case WM_DESTROY:
            KillTimer(hwnd, 1);
            PostQuitMessage(0);
            break;

        default:
            return DefWindowProc(hwnd, msg, wParam, lParam);
    }
    return 0;
}

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
    WNDCLASS wc = {0};
    wc.lpfnWndProc = WndProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;
    wc.hCursor = LoadCursor(NULL, IDC_ARROW);
    wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1);

    if (!RegisterClass(&wc)) {
        MessageBox(NULL, "注册窗口类失败！", "错误", MB_ICONERROR);
        return 0;
    }

    hwnd = CreateWindowEx(
        0,
        CLASS_NAME,
        "坦克发射炮弹游戏",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, 600, 500,
        NULL, NULL, hInstance, NULL
    );

    if (!hwnd) {
        MessageBox(NULL, "创建窗口失败！", "错误", MB_ICONERROR);
        return 0;
    }

    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);

    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return (int)msg.wParam;
}
```

## 捌：C++ EasyX 库

后来我学习了 EasyX 库。但这个需要到 [easyx](https://easyx.cn/) 这里下载，在引用时加上头文件 graphics.h 就可以画图形了。

安装的话，登进网址后，点击下载，再按照安装步骤来。

下面是几个常用的函数：

1. initgraph(x,y)
   
   创建一个 x*y 大小的窗口。

2. setbkcolor(RGB(x,y,z))
   
   设置绘图设备的背景颜色。（RGB 控制颜色，0≤x,y,z≤256）

3. cleardevice()
   
   清除屏幕并将当前位置设置为 (0,0)。

4. settextstyle(x, y, _T(z));
   
   设置字体高、宽、字体名称。

5. outtextxy(x,y,_T(z));
   
   在坐标（x,y）的地方输出字符串 z。
   
   假如你想输出变量，其中一种方法是用 swprintf_s：
   
   ```cpp
   wchar_t wScore[105], wLength[105];
   swprintf_s(wScore, L"%d", score);
   swprintf_s(wLength, L"%d", length);
   outtextxy(1, 1, _T("积分: "));
   outtextxy(80, 1, wScore);
   outtextxy(150, 1, _T("长度: "));
   outtextxy(230, 1, wLength);
   ```

6. solidrectangle(x1,y1,x2,y2)
   
   绘制一个从左上角 (x1, y1) 到右下角 (x2, y2) 的实心矩形。

7. setfillcolor(RGB(x,y,z))
   
   用于设置图形的填充颜色。

8. line(x1,y1,x2,y2)
   
   绘制一个从左上角 (x1, y1) 到右下角 (x2, y2) 的直线。

9. circle(x,y,r)
   
   在坐标（x,y）绘制一个半径为 r 的圆。

另外，认识一下 getchar() 函数，它可以读入一个字符，然后配合之前的知识就可以做简单的游戏了。

## 玖：C++ 简单的贪吃蛇小游戏

通过之前学习的内容，我们即可变出简单的贪吃蛇小游戏，上下左右或 wasd 操控，整体像在一个没有打印的网格上移动。

注意：只有下载了 EasyX 库才能运行成功。

```cpp
#include <bits/stdc++.h>
#include <graphics.h>
#include <stdio.h>
#include <conio.h>
#include<string.h>
#define NODE_WIDTH 20
struct node {
    int x, y;
};
int score = 0;
node snakeMove(node* snake, int length, int d) {
    node tail = snake[length - 1];
    for (int i = length - 1; i > 0; i--)
        snake[i] = snake[i - 1];
    node newHead;
    newHead = snake[0];
    if (d == 0)
        newHead.y--;
    else if (d == 1)
        newHead.y++;
    else if (d == 2)
        newHead.x--;
    else
        newHead.x++;
    if (newHead.x < 0 || newHead.x >= 75 || newHead.y < 0 || newHead.y >= 40) {
        settextstyle(200, 0, _T("宋体"));
        outtextxy(500, 100, _T("GAME"));
        Sleep(1000);
        settextstyle(250, 0, _T("宋体"));
        outtextxy(450, 400, _T("OVER"));
        Sleep(1000);
        exit(0);
    }
    snake[0] = newHead;
    return tail;
}
node createFood(node* snake, int length) {
    node food;
    while (1) {
        food.x = rand() % 75;
        food.y = rand() % 40;
        int i;
        for (i = 0; i < length; i++)
            if (snake[i].x == food.x && snake[i].y == food.y)
                break;
        if (i < length)
            continue;
        else
            break;
    }
    return food;
}
int main() {
    srand(time(0));
    initgraph(1500, 800);
    setbkcolor(RGB(rand() % 256 + 1, rand() % 256 + 1, rand() % 256 + 1));
    cleardevice();
    node snake[100] = { {5, 7}, {4, 7}, {3, 7}, {2, 7}, {1, 7} };
    int length = 5;
    int d = 3;
    node food = createFood(snake, length);
    while (1) {
        cleardevice();
        settextstyle(30, 0, _T("宋体"));
        wchar_t wScore[105], wLength[105];
        swprintf_s(wScore, L"%d", score);
        swprintf_s(wLength, L"%d", length);

        outtextxy(1, 1, _T("积分: "));
        outtextxy(80, 1, wScore);

        outtextxy(150, 1, _T("长度: "));
        outtextxy(230, 1, wLength);
        int left, top, right, bottom;
        for (int i = 0; i < length; i++) {
            left = snake[i].x * NODE_WIDTH;
            top = snake[i].y * NODE_WIDTH;
            right = (snake[i].x + 1) * NODE_WIDTH;
            bottom = (snake[i].y + 1) * NODE_WIDTH;
            solidrectangle(left, top, right, bottom);
        }
        left = food.x * NODE_WIDTH;
        top = food.y * NODE_WIDTH;
        right = (food.x + 1) * NODE_WIDTH;
        bottom = (food.y + 1) * NODE_WIDTH;
        setfillcolor(YELLOW);
        solidrectangle(left, top, right, bottom);
        setfillcolor(WHITE);
        Sleep(200);
        if (_kbhit()) {
            int c = _getch();
            switch (c)
            {
            case 'w':
                if (d != 1)
                    d = 0;
                break;
            case 's':
                if (d != 0)
                    d = 1;
                break;
            case 'a':
                if (d != 3)
                    d = 2;
                break;
            case 'd':
                if (d != 2)
                    d = 3;
                break;
            case 72:
                if (d != 1)
                    d = 0;
                break;
            case 80:
                if (d != 0)
                    d = 1;
                break;
            case 75:
                if (d != 3)
                    d = 2;
                break;
            case 77:
                if (d != 2)
                    d = 3;
                break;
            case 32:
                setbkcolor(RGB(rand() % 256 + 1, rand() % 256 + 1, rand() % 256 + 1));
                cleardevice();
            }
        }
        node tail = snakeMove(snake, length, d);
        if (snake[0].x == food.x && snake[0].y == food.y) {
            if (length < 100) {

                snake[length] = tail;
                length++;
                score += 10;
            }
            food = createFood(snake, length);
        }
        bool flag = false;
        for (int i = 1; i < length; i++) {
            if (snake[i].x == snake[0].x && snake[i].y == snake[0].y) {
                flag = true;
                break;
            }
        }
        if (flag) {
            settextstyle(200, 0, _T("宋体"));
            outtextxy(500, 100, _T("GAME"));
            Sleep(1000);
            settextstyle(250, 0, _T("宋体"));
            outtextxy(450, 400, _T("OVER"));
            Sleep(1000);
            exit(0);
        }
    }
    getchar();
    closegraph();
    return 0;
}
```

## 拾：C++ GetAsyncKeyState() 优化

GetAsyncKeyState() 函数用于检查某个键是否被按下，前面的代码中必须要让黑色的窗口出现在屏幕上，十分麻烦，而且反应很慢，于是想到了这个函数。

GetAsyncKeyState() 是否简洁快速，反应很快，用过都会感觉很灵敏。这也是 windows.h 库里的一个函数。

于是我们修改代码得到最终结果：

```cpp
#include <bits/stdc++.h>
#include <graphics.h>
#include <stdio.h>
#include <conio.h>
#include <string.h>
#include <windows.h>
#define NODE_WIDTH 20
struct node {
    int x, y;
};
int score = 0;
node snakeMove(node* snake, int length, int d) {
    node tail = snake[length - 1];
    for (int i = length - 1; i > 0; i--)
        snake[i] = snake[i - 1];
    node newHead;
    newHead = snake[0];
    if (d == 0)
        newHead.y--;
    else if (d == 1)
        newHead.y++;
    else if (d == 2)
        newHead.x--;
    else
        newHead.x++;
    if (newHead.x < 0 || newHead.x >= 75 || newHead.y < 0 || newHead.y >= 40) {
        settextstyle(200, 0, _T("宋体"));
        outtextxy(500, 100, _T("GAME"));
        Sleep(1000);
        settextstyle(250, 0, _T("宋体"));
        outtextxy(450, 400, _T("OVER"));
        Sleep(1000);
        exit(0);
    }
    snake[0] = newHead;
    return tail;
}
node createFood(node* snake, int length) {
    node food;
    while (1) {
        food.x = rand() % 75;
        food.y = rand() % 40;
        int i;
        for (i = 0; i < length; i++)
            if (snake[i].x == food.x && snake[i].y == food.y)
                break;
        if (i < length)
            continue;
        else
            break;
    }
    return food;
}
int main() {
    srand(time(0));
    initgraph(1500, 800);
    setbkcolor(RGB(rand() % 256 + 1, rand() % 256 + 1, rand() % 256 + 1));
    cleardevice();
    node snake[100] = { {5, 7}, {4, 7}, {3, 7}, {2, 7}, {1, 7} };
    int length = 5;
    int d = 3;
    node food = createFood(snake, length);
    while (1) {
        cleardevice();
        settextstyle(30, 0, _T("宋体"));
        wchar_t wScore[105], wLength[105];
        swprintf_s(wScore, L"%d", score);
        swprintf_s(wLength, L"%d", length);

        outtextxy(1, 1, _T("积分: "));
        outtextxy(80, 1, wScore);

        outtextxy(150, 1, _T("长度: "));
        outtextxy(230, 1, wLength);
        int left, top, right, bottom;
        for (int i = 0; i < length; i++) {
            left = snake[i].x * NODE_WIDTH;
            top = snake[i].y * NODE_WIDTH;
            right = (snake[i].x + 1) * NODE_WIDTH;
            bottom = (snake[i].y + 1) * NODE_WIDTH;
            solidrectangle(left, top, right, bottom);
        }
        left = food.x * NODE_WIDTH;
        top = food.y * NODE_WIDTH;
        right = (food.x + 1) * NODE_WIDTH;
        bottom = (food.y + 1) * NODE_WIDTH;
        setfillcolor(YELLOW);
        solidrectangle(left, top, right, bottom);
        setfillcolor(WHITE);
        Sleep(200);
        if (GetAsyncKeyState('W') && d != 1) { 
            d = 0;
        }
        else if (GetAsyncKeyState('S') && d != 0) {
            d = 1;
        }
        else if (GetAsyncKeyState('A') && d != 3) {
            d = 2;
        }
        else if (GetAsyncKeyState('D') && d != 2) { 
            d = 3;
        }
        else if (GetAsyncKeyState(VK_UP) && d != 1) {
            d = 0;
        }
        else if (GetAsyncKeyState(VK_DOWN) && d != 0) {
            d = 1;
        }
        else if (GetAsyncKeyState(VK_LEFT) && d != 3) {
            d = 2;
        }
        else if (GetAsyncKeyState(VK_RIGHT) && d != 2) {
            d = 3;
        }
        else if (GetAsyncKeyState(VK_SPACE)) { 
            setbkcolor(RGB(rand() % 256 + 1, rand() % 256 + 1, rand() % 256 + 1));
            cleardevice();
        }
        node tail = snakeMove(snake, length, d);
        if (snake[0].x == food.x && snake[0].y == food.y) {
            if (length < 100) {

                snake[length] = tail;
                length++;
                score += 10;
            }
            food = createFood(snake, length);
        }
        bool flag = false;
        for (int i = 1; i < length; i++) {
            if (snake[i].x == snake[0].x && snake[i].y == snake[0].y) {
                flag = true;
                break;
            }
        }
        if (flag) {
            settextstyle(200, 0, _T("宋体"));
            outtextxy(500, 100, _T("GAME"));
            Sleep(1000);
            settextstyle(250, 0, _T("宋体"));
            outtextxy(450, 400, _T("OVER"));
            Sleep(1000);
            exit(0);
        }
    }
    getchar();
    closegraph();
    return 0;
}
```

## 拾壹：C++ EasyX 画立体图形

EasyX 本身并没有画立体图形的函数，但我们可以通过二维画出三维的效果。

依靠刚刚认识的 line 等函数，我们可以间接实现画立体图形。

下面是画立方体的代码；

```cpp
#include <graphics.h>
#include <conio.h>

void convert3Dto2D(int x, int y, int z, int& screenX, int& screenY, int viewWidth, int viewHeight, int depth)
{
    screenX = x - y + (viewWidth >> 1);
    screenY = ((x + y) >> 1) - z + depth + (viewHeight >> 1);
}

int main()
{
    initgraph(640, 480); 

    int viewWidth = 640;
    int viewHeight = 480;
    int depth = 200;

    int cube[8][3] = { {50, 50, 50}, {150, 50, 50}, {150, 150, 50}, {50, 150, 50},
                      {50, 50, 150}, {150, 50, 150}, {150, 150, 150}, {50, 150, 150} };

    for (int i = 0; i < 4; i++)
    {
        int x1, y1, x2, y2;
        convert3Dto2D(cube[i][0], cube[i][1], cube[i][2], x1, y1, viewWidth, viewHeight, depth);
        convert3Dto2D(cube[(i + 1) % 4][0], cube[(i + 1) % 4][1], cube[(i + 1) % 4][2], x2, y2, viewWidth, viewHeight, depth);
        line(x1, y1, x2, y2);

        convert3Dto2D(cube[i + 4][0], cube[i + 4][1], cube[i + 4][2], x1, y1, viewWidth, viewHeight, depth);
        convert3Dto2D(cube[((i + 1) % 4) + 4][0], cube[((i + 1) % 4) + 4][1], cube[((i + 1) % 4) + 4][2], x2, y2, viewWidth, viewHeight, depth);
        line(x1, y1, x2, y2);

        convert3Dto2D(cube[i][0], cube[i][1], cube[i][2], x1, y1, viewWidth, viewHeight, depth);
        convert3Dto2D(cube[i + 4][0], cube[i + 4][1], cube[i + 4][2], x2, y2, viewWidth, viewHeight, depth);
        line(x1, y1, x2, y2);
    }

    _getch();
    closegraph();
    return 0;
}
```

## 拾贰：C++ 游戏库配合 ofstream 和 ifstream

有时候，游戏需要存一下账号信息，下次运行还要用的，那么就不能用寻常方法了。

ifstream 可以从文件里读取信息，ofstream 可以往文件里放信息。

要读取信息首先是这两行代码用于找到文件：

```cpp
string filePath = "C:\\example\\data.txt";
ifstream readFileStream(filePath);
```

然后可以用 is_open() 函数判断是否打开，接着用 getline 函数读取信息。

如果要放信息首先也是这两行代码用于找到文件：

```cpp
string filePath = "C:\\example\\data.txt";
ofstream writeFileStream(filePath, ios::app);
```

然后也可以用 is_open() 函数判断是否打开，接着用 writeFileStream << newContent 的方式在文件里加入 newContent 这个字符串。

接下来是一个使用示例：

```cpp
#include <iostream>
#include <fstream>
#include <string>

using namespace std;

int main() {
    string filePath = "C:\\example\\data.txt";

    ifstream readFileStream(filePath);
    if (readFileStream.is_open()) {
        cout << "文件内容如下：" << endl;
        string line;
        while (getline(readFileStream, line)) {
            cout << line << endl;
        }
        readFileStream.close();
    } else {
        cout << "无法打开文件进行读取" << endl;
    }

    ofstream writeFileStream(filePath, ios::app);
    if (writeFileStream.is_open()) {
        string newContent = "\n这是通过程序添加的新行";
        writeFileStream << newContent;
        cout << "已成功向文件添加新内容: " << newContent << endl;
        writeFileStream.close();
    } else {
        cout << "无法打开文件进行写入" << endl;
    }

    return 0;
}
```

## 拾叁：C++ 更高级的库

这里介绍的是较简单的游戏，不支持音效和 3D（音效只能用 windows.h 的 PlaySound 函数等），而且没那么好看，假如你想要制作更高级的游戏，建议用一下几个库：

| 库名        | 优点                     | 适合类型             |
| --------- | ---------------------- | ---------------- |
| Allegro 5 | 简单易学，支持图像、音频、字体、动画     | 2D 游戏（如平台跳跃、RPG） |
| SFML      | C++ 接口优雅，支持高清纹理、音效、网络  | 2D/轻量 3D，跨平台     |
| SDL2      | 高性能，被 Unity、Godot 内部使用 | 2D 游戏、模拟器、引擎开发   |
| Raylib    | 轻量、现代，支持 OpenGL，有中文文档  | 2D/3D 小游戏、原型开发   |

这些库的效果都很好，建议使用。

## 拾肆：结尾

看完这个文章后，希望你也能制作出自己的游戏！

感谢观看！
