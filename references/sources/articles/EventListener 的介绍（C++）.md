## 声明

本文所使用的语言环境是 C++17，包含了一些 C++17 的特性如 `[[nodiscard]]` 等等。使用到的头文件有：
```cpp
#include <iostream>
#include <map>
#include <string>
#include <typeindex>
#include <vector>
#include <utility>
```
并没有，并且拒绝使用 `using namespace std;`。

本文针对的是项目的开发，对算法竞赛并没有任何帮助，请注意受众和需求。

本文使用四个空格的缩进，变量命名规范并不明确，C++ 的代码写得有点像 Java。希望可以提供一个可供参考的方案，我会进行修改。

以及，如果可能有潜在的 bug，可以提出；如果有我没写明白的地方，欢迎探讨。

## 什么是监听者模式

我的理解简单来说，是当一个「事件」触发的时候让所有的「监听者」知道有这么回事，接下来所有的「监听者」可以通过自己的「方法」来分别处理这个「事件」。具体的，按顺序来讲就是：
1. 发生了「事件」（非处理过程）；
2. 将「事件」传递到「事件管理器」；
3. 「事件管理器」发送该「事件」到所有订阅该事件的「监听者」；
4. 「监听者」进行具体的处理。

### 注意事项

在下文和代码中：
- Event 代表「事件」
- Listener 代表「监听器」
- Impl 代表「方法实现」（implementation）
- EventManager 代表「事件管理器」

## 实现方法

### 文件结构

- Event.hpp
- EventListener.hpp
- EventManager.hpp
- （类的具体实现）.hpp
- （一个「模块」，包含类的更具体实现）.hpp
- main.cpp

下面的六个三级标题对应的就是不同的文件中的内容。

### 基类：Event

Event 是这样的，EventManager 只需要全身心添加监听器、删除监听器和广播事件就可以了，可是 Event 要考虑的事情就很多了。作为一个基类，他需要满足所有 Event 共有的特性：
- 如何创建这个 Event？需要一个构造函数；
- 如何触发这个 Event？需要一个触发器；
- 这个 Event 对应了什么监听器？需要一个 `getListenerType` 函数。

那么这个类的实现就很简单了：

```cpp
template<typename L /*extends EventListener*/>
class Event {
public:
    using listenerType = L;
    virtual ~Event() = default;
    virtual void fireEvent(std::vector<L *> listeners) = 0;
    std::type_index getListenerType() {
        return std::type_index(typeid(L));
    }
};
```

在实际使用的时候，此类只会被继承，方法是 `...... : public Event<CustomListener>`。后面「派生类」一节中会详细讲到。

`typename L` 是「事件」绑定的对应的「监听器」的类型名称。  
构造函数和触发事件的函数全部都设置为虚函数，方便我自定义的事件进行继承。  
获取监听器类型的函数直接返回 `L` 的 `type_index` 即可；这个 `type_index` 是我挖的坑，后面在「事件管理器」一节中会讲到。

### 基类：EventListener

EventListener 其实也是这样的。作为一个基类，他需要满足所有 EventListener 共有的特性：
- 这个监听器到底监听的是什么事件？定义一个变量来表明绑定的类型。
- 这个监听器是怎么声明的？构造函数。
- 监听到事件后要做什么？我不管这事，交给派生类处理吧。

```cpp
class EventListener {
    std::type_index eventType;

protected:
    explicit EventListener(const std::type_index type) : eventType(type) {}

public:
    virtual ~EventListener() = default;
    [[nodiscard]] std::type_index getEventType() const { return eventType; }
};
```

相比于 Event，EventListener 的使用简单了不少，直接继承即可。  
而这个 eventType 挖了个大坑。后面「派生类」一节中会详细讲到。

### 事件管理器：EventManager

具体的，EventManager 只需要全身心添加监听器、删除监听器和广播事件就可以了……吗？

首先，一个合格的 EventManager 需要一个列表来存储都有哪些「监听器」订阅了哪些「事件」。所以先写个开头：

```cpp
class EventManager {
    std::map<std::type_index, std::vector<EventListener *>> listenerMap;

public:
    ......
} eventManager;
```

别忘了创建一个 eventManager 对象，因为所有的操作都严重依赖于它。如果这个可以是个单例那么就更好了，但是我不会写。

#### 添加事件

现在考虑添加一个事件监听器，传入的参数只有监听器自己。

因为我们有 `listenerMap`，所以无脑传入进去就可以了。在键值对中，「键」是类型的 `type_index`，而值是对应的动态数组（`vector`），表示订阅了事件对应了 `type_index` 的所有监听器。

```cpp
    template<typename L /*extends EventListener*/>
    void addEventListener(L *eventListener) {
        try {
            auto &listeners = listenerMap[eventListener->getEventType()];
            listeners.push_back(eventListener);
            std::cout << "Adding Event Listener: eventType = " << eventListener->getEventType().name() << std::endl;
        } catch (const std::exception &e) {
            std::cerr << "Failed in adding: " << e.what() << std::endl;
            std::cerr << "eventType = " << eventListener->getEventType().name() << std::endl;
            throw;
        }
    }
```

这个 `try ... catch` 是直接借鉴参考了 Java 的错误处理，我也不知道具体会不会出错（我本人没用出问题过），也许吧。

#### 删除事件

删除事件和添加事件极为类似。把 `push_back` 改成 `erase` 并稍微调整参数即可；并且还要更改一下输出的日志。

```cpp
    template<typename L /*extends EventListener*/>
    void removeEventListener(L *eventListener) {
        try {
            auto &listeners = listenerMap[eventListener->getEventType()];
            listeners.erase(std::remove(listeners.begin(), listeners.end(), eventListener), listeners.end());
            std::cout << "Removed Event Listener: eventType = " << eventListener->getEventType().name() << std::endl;
        } catch (const std::exception &e) {
            std::cerr << "Failed in removing: " << e.what() << std::endl;
            std::cerr << "eventType = " << eventListener->getEventType().name() << std::endl;
            throw;
        }
    }
```

#### Fire Event

现在我考虑我有一个事件，我要用 `eventManager.fireEvent` 来通知所有的监听器对这个事件进行处理。

首先考虑传入的参数：它会使用到很多，例如说事件类型、事件具体信息和事件监听器的类型；  
但是，第一个可以自动推导，第三个已经在 `Event` 类的声明中解决了（因为我不会在只传入一个参数的情况下自动推导下面代码中注释掉的 `typename L`，我可能必须要写尖括号，这样太麻烦、啰嗦）。  
所以，参数中仅包含第二个内容即可。

把事件发给所有监听器的过程包含如下步骤：
- 获取所有「监听者列表」中监听对应事件的「监听者」；
- 分别给所有「监听者」发送事件。

前者使用 `listenersMap.find()` 即可，如果包含，则后者……

哦，后者没有直接发送这些事件的能力。每一个不同的事件包含的成员变量都是不一样的，仅仅这一个函数压根没办法处理所有的东西。直接把这个活推给对应的 Event 派生类即可。

```cpp
    template<typename E /*extends Event, L extends EventListener*/>
    void fireEvent(E *event) {
        try {
            using L = typename E::listenerType;
            auto eventType = std::type_index(typeid(E));
            auto listenerType = event->getListenerType();
            std::cout << "Firing event:\n- eventType = " << eventType.name();
            std::cout << ";\n- listenerType = " << listenerType.name() << std::endl;
            const auto it = listenerMap.find(eventType);
            if (it == listenerMap.end() || it->second.empty())
                return;

            std::vector<L *> listenersCopy;
            for (auto listener: it->second) {
                if (auto castedListener = dynamic_cast<L *>(listener)) {
                    listenersCopy.push_back(castedListener);
                }
            }
            std::cout << listenersCopy.size() << " listeners affected." << std::endl;

            event->fireEvent(listenersCopy);
            std::cout << "Fired event: event.class = " << typeid(E).name() << std::endl;
        } catch (const std::exception &e) {
            std::cerr << "Failed in firing: " << e.what() << std::endl;
            std::cerr << "event.class = " << typeid(E).name() << std::endl;
            throw;
        }
    }
```

在这个过程中，`addEventListener` 和 `removeEventListener` 把不同的各式各样的监听器都转换成了基类，而在 `fireEvent` 的时候其实把不同的监听器全都用基类 `EventListener` 来存储不会导致多出来的函数丢失的问题。

C++ 的多态特性保证了指向派生类的基类指针在正确转换后仍能访问派生类的方法，在 `dynamic_cast` 的时候不会返回一个 `nullptr`。谢天谢地，没事。

### 派生类 I：具体的事件和监听器类

现在不难发现我到了这些东西之后什么事都干不了。因为这个可以类比是一个「接口」，没有相对具体的应用是压根没有用的。

现在我设置一个具体的情景：
> 有一个人要给我发消息，这个消息包含两段或三段内容：
> - `int uid;`，这个人的名字
> - `string msg_type`，这个人消息的类型（欢迎消息 / 自动回复 / 聊天信息）
> - `string msg_content`，若类型是聊天信息，则存在这个人消息的内容
> 
> 在消息被广播的时候，我会接收到一个类型为 `MsgClass` 的 `msg`，包含上面提及的成员变量，且已经填好。  
> 被广播的时候，触发了 `eventManager.fireEvent(new BroadcastEvent(msg));`

那么我需要实现的是具体的 `BroadcastEvent` 和 `BroadcastListener` 类。感谢于面向对象编程的继承特性。

```cpp
class BroadcastListener : public EventListener {
public:
    BroadcastListener();
    virtual void onBroadcast(const nlohmann::json &rawMessage)  {
        std::cerr << "Default implementation of onBroadcast. Check if overrides successfully."
                  << std::endl;
    }
};
```

在上面代码中，监听器的方法直接进行了继承，而多出来的一个方法是具体的实现（交给派生类来处理吧），之所以这个是个虚函数是因为不同的具体的监听器需要进行不同的操作。

注意事项：请注意这个 `override`。但凡一个参数是不一样的，就不会成功覆写；并且，C++ 不会像 Java 一样提示你必须实现哪些方法。在具体实现的时候，若出错就会直接使用了基类的，导致并没有按照预期运行。

注意到了 `BroadcastListener();` 里面什么都没写。这个是故意的。

接下来是具体的 Event，写完了就不用改了：

```cpp
class BroadcastEvent final : public Event<BroadcastListener> {
public:
    MsgClass rawMessage;
    explicit BroadcastEvent(const MsgClass &rawMsg) : rawMessage(std::move(rawMsg)) {}

    void fireEvent(std::vector<BroadcastListener *> listeners) override {
        for (const auto listener: listeners) {
            listener->onBroadcast(rawMessage);
        }
    }
};
```

这里面包含一个构造函数，这个派生类自己的杂七杂八的东西，和 EventManager 没法直接处理的 fireEvent 操作。

在最后面需要声明一下 Listener 的构造函数：

```cpp
inline BroadcastListener::BroadcastListener() : EventListener(typeid(BroadcastEvent)) {}
```

声明到了最后面是因为，`typeid()` 里面没法写一个前置声明的类名，因为这个前置被使用时尚未完全定义。只能把构造函数放到后面了。

### 派生类 II：触发事件时的动作

然而我有了 BroadcastListener 之后还是没有用，因为具体实现还是没写。如果具体实现用函数和 if 等等来堆砌起来，和不使用 EventListener 一模一样。但是这个派生类写起来相对简单。以及，可能需要写不少个派生类，满足了「一个类只做一件事」的原则。

其一，处理聊天信息
```cpp
class messageListener : public BroadcastListener {
public:
    void onBroadcast(const MsgClass &msg) override {
        if (msg.msg_type != "chat_message")
            return;
        std::cout << "Received Message: " << msg.msg_content << std::endl;
        // 此处包含把聊天消息显示到另一个地方的代码，涉及到了太多内容故省略了
    }
} messageHdl;
```
其二，处理自动回复
```cpp
class autoreplyListener : public BroadcastListener {
public:
    void onBroadcast(const MsgClass &msg) override {
        if (msg.msg_type != "auto_reply_message")
            return;
        std::cout << "Ignored an auto reply" << std::endl;
        // 此处忽略（不显示）自动回复的具体内容，但是留档
    }
} autoreplyHdl;
```
其三，处理欢迎消息
```cpp
class welcomeListener : public BroadcastListener {
public:
    void onBroadcast(const MsgClass &msg) override {
        if (msg.msg_type != "welcome_message")
            return;
        std::cout << "Welcome user which uid is " << uid << std::endl;
        // 此处包含系统通知提示欢迎的代码，涉及到了太多内容故省略了
    }
} welcomeHdl;
```

### 使用以上所写的东西

在实际使用的时候，需要注册这些监听器，然后按需求把它们纳入到监听列表里。

```cpp
void registerEvents() {
    eventManager.addEventListener(&messageHdl);
    eventManager.addEventListener(&autoreplyHdl);
    eventManager.addEventListener(&welcomeHdl);
    std::cout << "Events registered!" << std::endl;
}
```

而在一些合适的时候，也需要适当的 `removeEventListener` 来防止这个监听器继续工作。

情境中提到的「被广播的时候，触发了 `eventManager.fireEvent(new BroadcastEvent(msg));`」在实际上是不存在的，你往往需要在合适的代码中自己写进去这一句。

## 具体应用

本文章并非面向 OI 所写，所以信息学竞赛选手看这个对竞赛是毫无帮助的。

在处理一个项目的时候，有的时候各种的处理逻辑很复杂和难受，触发与响应逻辑高度耦合。并且我并不需要知道事件的触发方。

在面对这种情况的时候就可以选择使用这个监听者模式（或者说观察者模式）来对项目进行重构，防止了硬编码多个响应逻辑。此外，因为避免了修改整个核心代码，此举也可以增加维护性和扩展性——只需要多注册一个监听器就可以使用新的逻辑了。

还有一些其他的好处，我不太确定。

## 其他玩法

一个类是可以多重继承的。

例如说，我可以使用：

```cpp
class welcomeListener : public BroadcastListener, public NewFriendListener {
public:
    void onBroadcast(const MsgClass &msg) override {
        ......
    }
    void onFriendRequest(const UserClass &usr) override {
        ......
    }
} welcomeHdl;
```

上述代码的具体实现已经省略，具体是让我在接受到群聊新群成员通知和接受到好友申请时进行迎新。因为都属于迎新模块的内容，写到一个头文件里我认为是没问题的。

## 参考文献

- [观察者设计模式](https://refactoringguru.cn/design-patterns/observer)
- [C++ 观察者模式讲解和代码示例](https://refactoringguru.cn/design-patterns/observer/cpp/example)
- [MrPython 的指导](https://www.luogu.com.cn/discuss/961871)
- [VS Code API](https://code.visualstudio.com/api/references/vscode-api#Event)
- [监听者模式（listener）(c++实现)](https://blog.csdn.net/liukang325/article/details/44461449)
- [用C++实现事件监听器](https://www.wds.ink/2023/03/16/%E7%94%A8C-%E5%AE%9E%E7%8E%B0%E4%BA%8B%E4%BB%B6%E7%9B%91%E5%90%AC%E5%99%A8/)
- **[Wurst7/src/main/java/net/wurstclient/event/](https://github.com/Wurst-Imperium/Wurst7/tree/master/src/main/java/net/wurstclient/event)**

但是可能有些压根没参考，只是看了看。

[Wurst7/src/main/java/net/wurstclient/event/](https://github.com/Wurst-Imperium/Wurst7/tree/master/src/main/java/net/wurstclient/event) 里面的代码非常直观的展示了是怎么处理的：在 Mixin 中触发事件（fireEvent），事件被不同的模块使用，不同模块进行不同操作。