## 引言

Ctrl+Alt+Del（全称 Control-Alt-Delete）是 Windows 系统中一个具有特殊地位的组合键。按下这三个键会调出一个独立的安全界面，用户可以在这里锁定电脑、切换用户、注销、更改密码或打开任务管理器。如今当电脑卡死、程序无响应、甚至遭遇病毒攻击时，大家第一反应往往就是按下 Ctrl+Alt+Del，通过任务管理器强制结束问题进程。可以说，这个组合键是微软留给用户的"最后一道防线"。

然而我们的极域电子教室系统有一个令人"印象深刻"的功能——它可以直接屏蔽 Ctrl+Alt+Del 组合键。实际体验更接近于震惊和细思极恐：当老师开启了屏幕广播后，我们会惊愕地发现，那个曾经百试百灵的"救命稻草"彻底失效了。无论你如何用力按下这三个键，甚至怀疑是不是键盘坏了而反复尝试，系统都毫无反应，仿佛这个组合键从未存在过。

那么极域是如何做到这一点的?

答案藏在一个名为 `TDKeybd.sys` 的底层驱动中。[^1] 这个驱动工作在系统的最底层——内核模式，比普通应用程序拥有更高的权限。

:::info[人话]{open}
**不讲武德**，在系统检测到“按下 Ctrl+Alt+Del”就把这个信号吞掉了。

换句话说，它在硬件或驱动层面进行了拦截，根本不给 Windows 系统“知道”的机会。正常情况下，按下 Ctrl+Alt+Del 会触发系统弹出安全桌面（这是一个独立的桌面环境，具有最高优先级），但当驱动层就把按键信号吞掉后，系统根本不知道用户按了这个组合键，自然也就不会有任何反应。
:::

然而，我们却无法做到同样地事情。

如果你想尝试用同样的方法编写一个在驱动层面拦截按键的程序，很快就会碰壁——因为驱动想要加载到 Windows 内核中，**必须经过数字签名**。而获得这个签名,就必须向微软申请代码签名证书并支付**不菲的费用**。这是微软为了系统安全而设置的门槛：防止恶意驱动程序随意进入系统核心。

换言之，极域作为一款**商业软件**，具备合法的用途（帮助教学）和合法的驱动签名，因此能够“光明正大”地在系统底层动手脚。而普通用户，既没有经济实力获取签名，也不具备绕过签名验证的技术手段(后者还可能违反相关法律)，只能眼睁睁看着电脑控制权被剥夺。

从技术角度看，这确实是一个“高明”的设计。但从用户体验角度，这无疑是一种近乎霸道的控制方式——它将 Windows 系统留给用户的最后一道“紧急出口”也牢牢封死了。

本文将讨论作为一名普通用户如何在普通用户态（Ring 3）淦掉 Ctrl+Alt+Del。

:::info[分级保护域]{open}

简单来说，这是 x86 CPU 的分级保护域（Privilege Rings），用来决定软件能干多少事：

- **Ring 0（内核态）**： 权限最高。操作系统内核（Kernel）、内核模式驱动程序（如 `TDKeybd`）跑在这里，可以直接操控硬件（CPU、内存、磁盘）。

- **Ring 1 & 2**：设计用于运行驱动程序。

- **Ring 3 用户态**： 权限最低。你平时运行的普通软件（浏览器、游戏、微信）都跑在这里。它们想访问硬件必须通过“系统调用”请求 Ring 0 帮忙。

总的来说就是 R0 有权限最高，R3 权限最低。

核心目的： 隔离和保护。防止普通程序因为一个 Bug 或恶意代码直接搞挂整个系统。

但现在的 OS，包括 Windows 和 Linux 都没有采用四层权限，而只是使用两层——R0 层和 R3 层，分别来存放操作系统数据和应用程序数据。驱动程序一旦加载了，就运行在R0层。

:::

我们可以查到 Ctrl+Alt+Del 的处理流程。

![](https://cdn.luogu.com.cn/upload/image_hosting/yraf8oiy.png)

如图，可以注意到 Winlogon.exe 在执行流中是一个很特殊的节点。因为它负责最后安全桌面（即你看到的那个界面）的弹出。同时它也是 Windows 中少有可以直接获取句柄的进程。

所以考虑对 Winlogon.exe 进行逆向工程，对其中的安全桌面弹出过程进行拦截。

## 逆向 Winlogon.exe

### 函数 `WlSDSimulateSAS()` 分析

![](https://cdn.luogu.com.cn/upload/image_hosting/wgfkyewa.png)

首先通过关键词搜索可以注意到函数 `WlSecureDesktoprSimulateSAS()`。

```cpp
__int64 __fastcall WlSecureDesktoprSimulateSAS(RPC_BINDING_HANDLE BindingHandle)
{
  ULONG v2; // ebx
  NTSTATUS v3; // eax
  __int64 v4; // r8
  int v6; // [rsp+58h] [rbp+10h] BYREF
  NTSTATUS Status; // [rsp+60h] [rbp+18h] BYREF
  DWORD v8; // [rsp+68h] [rbp+20h] BYREF

  v6 = 0;
  v8 = 4;
  RegGetValueA(
    HKEY_LOCAL_MACHINE,
    "Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\System",
    "SoftwareSASGeneration",
    0x10u,
    0i64,
    &v6,
    &v8);
  if ( (v6 & 2) != 0 )
  {
    v2 = RpcImpersonateClient(BindingHandle);
    if ( !v2 )
    {
      v3 = CompareClientSidToLoggedOnUserSid(1i64);
      Status = v3;
      if ( v3 >= 0 )
      {
        RpcRevertToSelf();
        WmsgSendMessage(NtCurrentPeb()->SessionId, 520i64, v4, &Status);
        if ( Status < 0 )
          return RtlNtStatusToDosError(Status);
      }
      else
      {
        v2 = RtlNtStatusToDosError(v3);
        RpcRevertToSelf();
      }
    }
  }
  else
  {
    return 5;
  }
  return v2;
}
```

#### 函数签名

```c
__int64 __fastcall WlSecureDesktoprSimulateSAS(RPC_BINDING_HANDLE BindingHandle)
```

- **函数名**：`WlSecureDesktoprSimulateSAS` 
  
  - `WlSecureDesktop` 前缀表明这是 Winlogon 安全桌面相关函数。
  - **`SimulateSAS`** = Simulate Secure Attention Sequence（模拟安全注意序列）。
  - 它可以**软件模拟** Ctrl+Alt+Del 的效果。

- **参数**：`RPC_BINDING_HANDLE` 表明这是一个 RPC（远程过程调用）接口，可以被其他进程调用。

#### 逐步分析

##### 1. 注册表设置

```c
RegGetValueA(
    HKEY_LOCAL_MACHINE,
    "Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\System",
    "SoftwareSASGeneration",
    0x10u,
    0i64,
    &v6,
    &v8);
```

读取注册表项 `SoftwareSASGeneration`，这个值控制是否允许软件模拟 SAS。

##### 2. 权限位检查

```c
if ( (v6 & 2) != 0 )
```

检查 `SoftwareSASGeneration` 的第 2 位是否设置。如果**没有设置这个位**，函数直接返回错误代码 5（`ERROR_ACCESS_DENIED`，访问被拒绝）。

:::info[SoftwareSASGeneration 的值]{open}
这个注册表值的含义：

- **0**：禁用（默认，安全）
- **1**：仅服务可以模拟
- **2**：所有应用可以模拟（不安全）
- **3**：服务和应用都可以模拟
  :::

##### 3. RPC 身份模拟

```c
v2 = RpcImpersonateClient(BindingHandle);
```

模拟调用客户端的身份，以便进行权限检查。这是 RPC 安全机制的一部分。

##### 4. 验证调用者身份

```c
v3 = CompareClientSidToLoggedOnUserSid(1i64);
Status = v3;
```

**关键检查**：比较调用者的 SID（安全标识符）与当前登录用户的 SID 是否一致。

这个检查确保：

- 只有**当前会话的用户**才能触发自己的 SAS。
- 防止其他用户或进程恶意触发你的安全桌面。

##### 5. 发送消息触发 SAS

```cpp
if ( v3 >= 0 )
{
    RpcRevertToSelf();
    WmsgSendMessage(NtCurrentPeb()->SessionId, 520i64, v4, &Status);
    if ( Status < 0 )
        return RtlNtStatusToDosError(Status);
}
```

如果权限验证通过：

- 恢复 Winlogon.exe 原有身份（`RpcRevertToSelf`）。
- 调用 `WmsgSendMessage` 发送消息码 **520**（0x208）到当前会话。
- 这个消息会触发 Winlogon 显示安全桌面。

#### 核心发现

##### 这个函数做了什么？

它提供了一个**软件接口**来模拟 Ctrl+Alt+Del 的效果，绕过物理按键！内部通过 `WmsgSendMessage()` 触发安全桌面。

所以我们继续向下分析 `WmsgSendMessage()`。

### 函数 `WmsgSendMessage()` 分析

```cpp
__int64 __fastcall WmsgSendMessage(__int64 a1, unsigned int a2, __int64 a3, __int64 a4)
{
  unsigned int v6; // ebx
  __int64 v7; // r8
  RPC_BINDING_HANDLE Binding[3]; // [rsp+20h] [rbp-18h] BYREF

  Binding[0] = 0i64;
  v6 = WmsgpConnect(a1, Binding);
  if ( !v6 )
  {
    v6 = WmsgpSendMessage(Binding[0], a2, v7, a4);
    WmsgpDisconnect(Binding);
  }
  return v6;
}

__int64 __fastcall WmsgpConnect(unsigned int a1, RPC_BINDING_HANDLE *a2)
{
  __int64 v3; // r9
  unsigned int v4; // ebx
  RPC_BINDING_HANDLE_SECURITY_V1_W Security; // [rsp+30h] [rbp-118h] BYREF
  __int128 v7; // [rsp+58h] [rbp-F0h] BYREF
  __int128 v8; // [rsp+68h] [rbp-E0h]
  __int128 v9; // [rsp+78h] [rbp-D0h]
  UUID Uuid; // [rsp+88h] [rbp-C0h] BYREF
  RPC_BINDING_HANDLE_OPTIONS_V1 Options; // [rsp+98h] [rbp-B0h] BYREF
  RPC_BINDING_HANDLE_TEMPLATE_V1_W Template; // [rsp+A8h] [rbp-A0h] BYREF
  unsigned __int16 StringUuid[40]; // [rsp+E0h] [rbp-68h] BYREF

  Uuid = 0i64;
  *(_OWORD *)&Template.Version = xmmword_1400C3AB0;
  memset(&Template.NetworkAddress, 0, 40);
  Options = (RPC_BINDING_HANDLE_OPTIONS_V1)xmmword_1400C3A90;
  v7 = xmmword_1400B0940;
  v8 = xmmword_1400B0950;
  v9 = *(_OWORD *)&off_1400B0960;
  *(_QWORD *)&Security.Version = 1i64;
  Security.ServerPrincName = 0i64;
  Security.AuthnLevel = 6;
  Security.AuthnSvc = 10;
  Security.AuthIdentity = 0i64;
  Security.SecurityQos = (RPC_SECURITY_QOS *)&v7;
  v3 = 0i64;
  if ( a1 != -1 )
    v3 = a1;
  StringCchPrintfW(
    StringUuid,
    0x25ui64,
    L"b08669ee-8cb5-43a5-a017-84fe%08X",
    v3,
    0,
    a2,
    *(_QWORD *)&Security.Version,
    Security.ServerPrincName,
    *(_QWORD *)&Security.AuthnLevel,
    Security.AuthIdentity,
    Security.SecurityQos,
    v7,
    v8,
    v9);
  v4 = UuidFromStringW(StringUuid, &Uuid);
  if ( !v4 )
  {
    Template.Flags = 1;
    Template.ObjectUuid = Uuid;
    v4 = RpcBindingCreateW(&Template, &Security, &Options, a2);
    if ( !v4 )
      v4 = RpcBindingBind(0i64, *a2, &unk_1400B0BA0);
  }
  if ( v4 && *a2 )
  {
    RpcBindingFree(a2);
    *a2 = 0i64;
  }
  return v4;
}

RPC_STATUS __fastcall WmsgpSendMessage(__int64 a1, int a2, __int64 a3, RPC_STATUS *a4)
{
  RPC_STATUS result; // eax
  RPCNOTIFICATION_ROUTINE *EventW; // rax
  RPCNOTIFICATION_ROUTINE *v9; // rdi
  RPC_STATUS v10; // ebx
  DWORD v11; // esi
  struct _RPC_ASYNC_STATE pAsync; // [rsp+50h] [rbp-A8h] BYREF

  memset_0(&pAsync, 0, sizeof(pAsync));
  result = RpcAsyncInitializeHandle(&pAsync, 0x70u);
  if ( !result )
  {
    EventW = (RPCNOTIFICATION_ROUTINE *)CreateEventW(0i64, 0, 0, 0i64);
    v9 = EventW;
    if ( EventW )
    {
      pAsync.UserInfo = 0i64;
      pAsync.NotificationType = RpcNotificationTypeEvent;
      pAsync.u.APC.NotificationRoutine = EventW;
      v10 = 0;
      Ndr64AsyncClientCall((MIDL_STUBLESS_PROXY_INFO *)&stru_1400B0A80, 0, 0i64, &pAsync, a1, a2, 0, 0, EventW, a4);
      v11 = WaitForSingleObject(v9, 0xFFFFFFFF);
      if ( v11 == 258 )
      {
        v10 = RpcAsyncCancelCall(&pAsync, 1);
        if ( !v10 )
          v11 = WaitForSingleObject(v9, 0xFFFFFFFF);
      }
      if ( !v11 )
      {
        v10 = RpcAsyncCompleteCall(&pAsync, 0i64);
        CloseHandle(v9);
      }
      *a4 = v10;
      return v10;
    }
    else
    {
      return 14;
    }
  }
  return result;
}

RPC_STATUS __fastcall WmsgpDisconnect(RPC_BINDING_HANDLE *Binding)
{
  RPC_STATUS result; // eax

  result = RpcBindingUnbind(*Binding);
  if ( !result )
  {
    result = RpcBindingFree(Binding);
    if ( !result )
      *Binding = 0i64;
  }
  return result;
}
```

这是完整的消息发送链路！让我们详细分析这几个函数的协作关系：

#### 整体调用链

- **WmsgpConnect**：建立 RPC 连接
- **WmsgpSendMessage**：发送消息
- **WmsgpDisconnect**：断开连接

##### WmsgSendMessage 入口函数

```c
__int64 __fastcall WmsgSendMessage(__int64 a1, unsigned int a2, __int64 a3, __int64 a4)
```

**参数分析**：

- `a1`：Session ID（会话 ID）
- `a2`：消息码，前面看到的是 **520 (0x208)**
- `a3`、`a4`：消息参数

**工作流程**：

```c
Binding[0] = 0i64;
v6 = WmsgpConnect(a1, Binding);        // 1. 建立连接
if ( !v6 )
{
    v6 = WmsgpSendMessage(Binding[0], a2, v7, a4);  // 2. 发送消息
    WmsgpDisconnect(Binding);          // 3. 断开连接
}
```

标准的"连接、发送、断开"模式。

---

##### 2. WmsgpConnect 建立 RPC 连接

这是最关键的函数，揭示了连接的目标！

###### UUID 生成

```c
StringCchPrintfW(
    StringUuid,
    0x25ui64,
    L"b08669ee-8cb5-43a5-a017-84fe%08X",
    v3,  // Session ID
    ...
);
```

**关键发现**：

- 基础 UUID：`b08669ee-8cb5-43a5-a017-84fe`
- 后面拼接 **Session ID**（8 位十六进制）
- 例如 Session 0：`b08669ee-8cb5-43a5-a017-84fe00000000`
- 例如 Session 1：`b08669ee-8cb5-43a5-a017-84fe00000001`

###### RPC 绑定配置

```c
Security.AuthnLevel = 6;     // RPC_C_AUTHN_LEVEL_PKT_PRIVACY (最高加密级别)
Security.AuthnSvc = 10;      // RPC_C_AUTHN_WINNT (Windows NT 认证)
```

**安全设置**：

- 使用 **Windows NT 身份验证**。
- 认证级别为 **PKT_PRIVACY**（数据包加密），最高安全级别。
- 这就是为什么需要验证调用者身份（前面的 `CompareClientSidToLoggedOnUserSid`）。

###### 绑定创建

```c
Template.Flags = 1;
Template.ObjectUuid = Uuid;
v4 = RpcBindingCreateW(&Template, &Security, &Options, a2);
if ( !v4 )
    v4 = RpcBindingBind(0i64, *a2, &unk_1400B0BA0);
```

创建到 Winlogon 内部服务的 RPC 绑定句柄。

---

##### WmsgpSendMessage 异步发送消息

```c
RPC_STATUS __fastcall WmsgpSendMessage(__int64 a1, int a2, __int64 a3, RPC_STATUS *a4)
```

###### 异步 RPC 调用

```c
memset_0(&pAsync, 0, sizeof(pAsync));
result = RpcAsyncInitializeHandle(&pAsync, 0x70u);
```

初始化异步 RPC 句柄。

###### 创建事件对象

```c
EventW = (RPCNOTIFICATION_ROUTINE *)CreateEventW(0i64, 0, 0, 0i64);
pAsync.NotificationType = RpcNotificationTypeEvent;
pAsync.u.APC.NotificationRoutine = EventW;
```

使用 Windows 事件对象来等待异步调用完成。

###### 实际发送

```c
Ndr64AsyncClientCall(
    (MIDL_STUBLESS_PROXY_INFO *)&stru_1400B0A80, 
    0, 
    0i64, 
    &pAsync, 
    a1,     // RPC Binding Handle
    a2,     // 消息码 520
    0, 
    0, 
    EventW, 
    a4
);
```

**关键点**：

- 使用 `Ndr64AsyncClientCall` 进行异步 RPC 调用。
- `&stru_1400B0A80` 是 RPC 接口的元数据结构。
- 消息码 `a2 = 520` 被传递过去。

###### 等待完成

```c
v11 = WaitForSingleObject(v9, 0xFFFFFFFF);  // 无限等待
if ( v11 == 258 )  // WAIT_TIMEOUT
{
    v10 = RpcAsyncCancelCall(&pAsync, 1);   // 超时则取消
}
if ( !v11 )
{
    v10 = RpcAsyncCompleteCall(&pAsync, 0i64);  // 完成调用
}
```

标准的异步 RPC 等待模式。

---

##### 4. WmsgpDisconnect 清理连接

```c
result = RpcBindingUnbind(*Binding);
if ( !result )
{
    result = RpcBindingFree(Binding);
    if ( !result )
        *Binding = 0i64;
}
```

标准的 RPC 资源清理：解绑 → 释放 → 置空。

---

#### 核心发现总结

##### RPC 服务标识

**UUID 模式**：`b08669ee-8cb5-43a5-a017-84fe{SessionID}`

这是 Winlogon 内部的一个 **本地 RPC 服务端点**，每个用户会话都有自己的端点。

##### 消息码 520 的含义

```
520 (0x208) = SAS 模拟消息
```

当这个消息通过 RPC 发送到 Winlogon 服务时，会触发安全桌面的显示。

考虑通过唯一标识符 `b08669ee-8cb5-43a5-a017-84fe{SessionID}`，找到是谁接收了这一消息。

![](https://cdn.luogu.com.cn/upload/image_hosting/ywts7uwn.png)

可以看到字符串 `b08669ee-8cb5-43a5-a017-84fe{SessionID}` 在多次在 `StartWMsgServer()` 中被使用。

### 函数 `StartWMsgServer` 分析

```cpp
__int64 StartWMsgServer()
{
  unsigned int v0; // ebx
  CUser *v1; // rcx
  __int64 v2; // rdx
  wchar_t pszDest[40]; // [rsp+40h] [rbp-68h] BYREF

  v0 = RpcServerRegisterIfEx(&unk_1400AD4D0, 0i64, 0i64, 0x28u, 0x4D2u, (RPC_IF_CALLBACK_FN *)WmsgRpcSecurityCallback);
  if ( v0 )
  {
    v1 = WPP_GLOBAL_Control;
    if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
      && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 1) != 0
      && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 2u )
    {
      v2 = 21i64;
LABEL_33:
      WPP_SF_d(*((_QWORD *)v1 + 2), v2, &WPP_809920f00406303c4ef054ac00db20a5_Traceguids, v0);
    }
  }
  else
  {
    dword_1400DD894 = 1;
    v0 = RpcServerInqBindings(&BindingVector);
    if ( v0 )
    {
      v1 = WPP_GLOBAL_Control;
      if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
        && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 1) != 0
        && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 2u )
      {
        v2 = 22i64;
        goto LABEL_33;
      }
    }
    else
    {
      StringCchPrintfW(pszDest, 0x25ui64, L"b08669ee-8cb5-43a5-a017-84fe%08X", NtCurrentPeb()->SessionId);
      v0 = UuidFromStringW(pszDest, &Uuid);
      if ( v0 )
      {
        v1 = WPP_GLOBAL_Control;
        if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
          && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 1) != 0
          && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 2u )
        {
          v2 = 23i64;
          goto LABEL_33;
        }
      }
      else
      {
        UuidVector.Count = 1;
        UuidVector.Uuid[0] = &Uuid;
        v0 = WaitForDesiredService(L"RPCSS");
        if ( v0 )
        {
          v1 = WPP_GLOBAL_Control;
          if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
            && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 1) != 0
            && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 2u )
          {
            v2 = 24i64;
            goto LABEL_33;
          }
        }
        else
        {
          v0 = RpcEpRegisterW(&unk_1400AD4D0, BindingVector, &UuidVector, 0i64);
          if ( v0 )
          {
            v1 = WPP_GLOBAL_Control;
            if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
              && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 1) != 0
              && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 2u )
            {
              v2 = 25i64;
              goto LABEL_33;
            }
          }
          else
          {
            dword_1400DD8F0 = 1;
            v0 = RpcServerListen(1u, 0x4D2u, 1u);
            if ( v0 == 1713 )
              v0 = 0;
            if ( v0 )
            {
              v1 = WPP_GLOBAL_Control;
              if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
                && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 1) != 0
                && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 2u )
              {
                v2 = 26i64;
                goto LABEL_33;
              }
            }
          }
        }
      }
    }
  }
  if ( v0 )
    StopWMsgServer();
  return v0;
}
```

可以很容易看出来就是**启动** RPC 服务器。继续顺着调用流往上，可以查到 `StartWMsgServer()` 仅在 `WMsgClntInitialize()` 中被调用。

```cpp
__int64 __fastcall WMsgClntInitialize(struct _WLSM_GLOBAL_CONTEXT *a1, int a2)
{
  __int64 v5[9]; // [rsp+30h] [rbp-48h] BYREF

  memset_0(v5, 0, 0x40ui64);
  if ( !a2 )
    return StartWMsgServer();
  v5[0] = (__int64)WMsgMessageHandler;
  v5[1] = (__int64)WMsgKMessageHandler;
  v5[5] = (__int64)WMsgNotifyHandler;
  v5[2] = (__int64)WMsgPSPHandler;
  v5[3] = (__int64)WMsgReconnectionUpdateHandler;
  v5[4] = (__int64)WMsgGetSwitchUserLogonInfoHandler;
  RegisterWMsgServer(v5);
  return StartWMsgKServer(*(_QWORD *)a1 + 204i64);
}
```

这里为各个事件处理器注册为函数指针。

我们逐一分析这些函数（略），最终在 `WMsgMessageHandler` 发现了对于信号 `520i64` 的处理。

### 函数 WMsgMessageHandler 分析

```cpp
__int64 __fastcall WMsgMessageHandler(unsigned int a1, unsigned int a2, struct _RPC_ASYNC_STATE *a3, int *a4)
{
  unsigned __int8 v7; // dl
  CSession *v8; // rcx
  DWORD v9; // eax
  bool v10; // cl
  CUser *v11; // rcx
  __int64 v12; // rdx
  CSession *v13; // rcx
  DWORD LastError; // eax
  CSession *v15; // rcx
  DWORD v16; // eax

  if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
    && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
    && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 5u )
  {
    WPP_SF_llq(*((_QWORD *)WPP_GLOBAL_Control + 2), 27i64, a3, a1, a2, a3);
  }
  *a4 = 0;
  if ( a1 > 0x200 )
  {
    if ( a1 <= 0x403 )
    {
      if ( a1 != 1027 )
      {
        switch ( a1 )
        {
          case 0x201u:
            v11 = WPP_GLOBAL_Control;
            if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
              || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
              || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
            {
              return 1i64;
            }
            v12 = 33i64;
            goto LABEL_175;
          case 0x202u:
            WlStateMachineSetSignal(0x13u, 0i64);
            v11 = WPP_GLOBAL_Control;
            if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
              || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
              || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
            {
              return 1i64;
            }
            v12 = 34i64;
            goto LABEL_175;
          case 0x203u:
            v11 = WPP_GLOBAL_Control;
            if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
              || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
              || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
            {
              return 1i64;
            }
            v12 = 40i64;
            goto LABEL_175;
          case 0x204u:
            v11 = WPP_GLOBAL_Control;
            if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
              || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
              || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
            {
              return 1i64;
            }
            v12 = 44i64;
            goto LABEL_175;
          case 0x205u:
            if ( (unsigned __int8)wil::details::FeatureImpl<__WilFeatureTraits_Feature_WinlogonTSActivityId2>::__private_IsEnabled(&`wil::Feature<__WilFeatureTraits_Feature_WinlogonTSActivityId2>::GetImpl'::`2'::impl) )
              UpdateTSActivityId();
            WlStateMachineSetSignal(0x14u, 0i64);
            v11 = WPP_GLOBAL_Control;
            if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
              || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
              || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
            {
              return 1i64;
            }
            v12 = 45i64;
            goto LABEL_175;
          case 0x206u:
            v11 = WPP_GLOBAL_Control;
            if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
              || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
              || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
            {
              return 1i64;
            }
            v12 = 46i64;
            goto LABEL_175;
          case 0x208u:
            if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
              && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
              && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 5u )
            {
              WPP_SF_(*((_QWORD *)WPP_GLOBAL_Control + 2), 29i64, &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids);
            }
            if ( (unsigned int)(*((_DWORD *)qword_1400DCB20 + 37) - 1) > 2 && (unsigned int)AllowSAS() )
              WlStateMachineSetSignal(3u, 0i64);
            return 1i64;
          case 0x209u:
            WlStateMachineSetSignal(0x15u, 0i64);
            v11 = WPP_GLOBAL_Control;
            if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
              || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
              || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
            {
              return 1i64;
            }
            v12 = 43i64;
            goto LABEL_175;
          case 0x20Au:
            if ( (unsigned __int8)wil::details::FeatureImpl<__WilFeatureTraits_Feature_WinlogonTSActivityId2>::__private_IsEnabled(&`wil::Feature<__WilFeatureTraits_Feature_WinlogonTSActivityId2>::GetImpl'::`2'::impl) )
              UpdateTSActivityId();
            WlStateMachineSetSignal(0x16u, 0i64);
            v11 = WPP_GLOBAL_Control;
            if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
              || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
              || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
            {
              return 1i64;
            }
            v12 = 41i64;
            goto LABEL_175;
          case 0x20Bu:
            if ( (unsigned __int8)wil::details::FeatureImpl<__WilFeatureTraits_Feature_WinlogonTSActivityId2>::__private_IsEnabled(&`wil::Feature<__WilFeatureTraits_Feature_WinlogonTSActivityId2>::GetImpl'::`2'::impl) )
            {
              RtlAcquireResourceExclusive(&g_TSActivityIdLock, 1u);
              g_TSActivityId = prevTSActivityId;
              prevTSActivityId = xmmword_1400BEA30;
              RtlReleaseResource(&g_TSActivityIdLock);
            }
            if ( lpMem )
              SignalManagerSetSignal(*(PRTL_CRITICAL_SECTION *)lpMem);
            v11 = WPP_GLOBAL_Control;
            if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
              || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
              || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
            {
              return 1i64;
            }
            v12 = 42i64;
            goto LABEL_175;
          case 0x20Cu:
            if ( !*((_DWORD *)qword_1400DCB20 + 60)
              && !*((_DWORD *)qword_1400DCB20 + 62)
              && !*((_DWORD *)qword_1400DCB20 + 61) )
            {
              MicrosoftTelemetryAssertTriggeredArgs("lsm.dll", 0i64, 0i64);
            }
            v11 = WPP_GLOBAL_Control;
            if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
              || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
              || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
            {
              return 1i64;
            }
            v12 = 35i64;
            goto LABEL_175;
          case 0x20Du:
            if ( (unsigned __int8)wil::details::FeatureImpl<__WilFeatureTraits_Feature_WinlogonTSActivityId2>::__private_IsEnabled(&`wil::Feature<__WilFeatureTraits_Feature_WinlogonTSActivityId2>::GetImpl'::`2'::impl) )
              UpdateTSActivityId();
            v13 = qword_1400DCB20;
            if ( !*((_DWORD *)qword_1400DCB20 + 60)
              && !*((_DWORD *)qword_1400DCB20 + 62)
              && !*((_DWORD *)qword_1400DCB20 + 61) )
            {
              MicrosoftTelemetryAssertTriggeredArgs("lsm.dll", 0i64, 0i64);
              v13 = qword_1400DCB20;
            }
            if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
              && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
              && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 5u )
            {
              WPP_SF_(*((_QWORD *)WPP_GLOBAL_Control + 2), 36i64, &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids);
              v13 = qword_1400DCB20;
            }
            if ( !(unsigned int)CSession::SetLsmSyncEvent(v13)
              && WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
              && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
              && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 2u )
            {
              LastError = GetLastError();
              WPP_SF_d(
                *((_QWORD *)WPP_GLOBAL_Control + 2),
                37i64,
                &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids,
                LastError);
            }
            return 1i64;
          case 0x20Eu:
            v15 = qword_1400DCB20;
            if ( !*((_DWORD *)qword_1400DCB20 + 60)
              && !*((_DWORD *)qword_1400DCB20 + 62)
              && !*((_DWORD *)qword_1400DCB20 + 61) )
            {
              MicrosoftTelemetryAssertTriggeredArgs("lsm.dll", 0i64, 0i64);
              v15 = qword_1400DCB20;
            }
            if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
              && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
              && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 5u )
            {
              WPP_SF_(*((_QWORD *)WPP_GLOBAL_Control + 2), 38i64, &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids);
              v15 = qword_1400DCB20;
            }
            if ( *((_DWORD *)v15 + 60) )
            {
              *((_DWORD *)v15 + 60) = 0;
            }
            else if ( *((_DWORD *)v15 + 62) )
            {
              *((_DWORD *)v15 + 62) = 0;
            }
            else
            {
              if ( !*((_DWORD *)v15 + 61) )
                goto LABEL_89;
              *((_DWORD *)v15 + 61) = 0;
            }
            v15 = qword_1400DCB20;
LABEL_89:
            if ( !(unsigned int)CSession::SetLsmSyncEvent(v15)
              && WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
              && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
              && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 2u )
            {
              v16 = GetLastError();
              WPP_SF_d(
                *((_QWORD *)WPP_GLOBAL_Control + 2),
                39i64,
                &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids,
                v16);
            }
            CSession::SetUserSwitchLogonInfoCollectedSyncEvent(qword_1400DCB20);
            WlStateMachineSetSignal(2u, 0i64);
            break;
          default:
            goto LABEL_142;
        }
        return 1i64;
      }
      if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
        && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
        && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 5u )
      {
        WPP_SF_(*((_QWORD *)WPP_GLOBAL_Control + 2), 47i64, &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids);
      }
      if ( !lpMem )
        return 1i64;
      goto LABEL_136;
    }
    if ( a1 != 1029 )
    {
      switch ( a1 )
      {
        case 0x500u:
          if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
            && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
            && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 5u )
          {
            WPP_SF_(*((_QWORD *)WPP_GLOBAL_Control + 2), 48i64, &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids);
          }
          if ( lpMem )
            SignalManagerSetSignal(*(PRTL_CRITICAL_SECTION *)lpMem);
          break;
        case 0x501u:
          if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
            && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
            && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 5u )
          {
            WPP_SF_(*((_QWORD *)WPP_GLOBAL_Control + 2), 50i64, &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids);
          }
          if ( !lpMem )
            return 1i64;
LABEL_136:
          SignalManagerSetSignal(*(PRTL_CRITICAL_SECTION *)lpMem);
          return 1i64;
        case 0x502u:
          if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
            && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
            && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 5u )
          {
            WPP_SF_(*((_QWORD *)WPP_GLOBAL_Control + 2), 49i64, &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids);
          }
          if ( lpMem )
          {
            SignalManagerSetSignal(*(PRTL_CRITICAL_SECTION *)lpMem);
            return 0i64;
          }
          break;
        case 0x550u:
          WppStart(0, 4u);
          v11 = WPP_GLOBAL_Control;
          if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
            || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
            || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
          {
            return 1i64;
          }
          v12 = 51i64;
LABEL_175:
          WPP_SF_(*((_QWORD *)v11 + 2), v12, &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids);
          return 1i64;
        default:
          goto LABEL_142;
      }
      return 0i64;
    }
    if ( lpMem )
      SignalManagerSetSignal(*(PRTL_CRITICAL_SECTION *)lpMem);
    v11 = WPP_GLOBAL_Control;
    if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
      || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
      || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
    {
      return 1i64;
    }
    v12 = 28i64;
    goto LABEL_175;
  }
  if ( a1 == 512 )
  {
    v11 = WPP_GLOBAL_Control;
    if ( WPP_GLOBAL_Control == (CUser *)&WPP_GLOBAL_Control
      || (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) == 0
      || *((_BYTE *)WPP_GLOBAL_Control + 25) < 5u )
    {
      return 1i64;
    }
    v12 = 32i64;
    goto LABEL_175;
  }
  if ( a1 != 1 )
  {
LABEL_142:
    if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
      && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
      && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 3u )
    {
      WPP_SF_d(*((_QWORD *)WPP_GLOBAL_Control + 2), 52i64, &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids, a1);
    }
    return 1i64;
  }
  if ( WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
    && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
    && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 5u )
  {
    WPP_SF_d(*((_QWORD *)WPP_GLOBAL_Control + 2), 30i64, &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids, a2);
  }
  LogLocalRpcCaller();
  v8 = qword_1400DCB20;
  if ( !*((_DWORD *)qword_1400DCB20 + 60) )
  {
    if ( *((_DWORD *)qword_1400DCB20 + 62) )
    {
      *((_DWORD *)qword_1400DCB20 + 62) = 0;
    }
    else
    {
      if ( !*((_DWORD *)qword_1400DCB20 + 61) )
      {
LABEL_25:
        CSession::SetUserSwitchLogonInfoCollectedSyncEvent(v8);
        WLEventWrite(&WLEvt_ReceivedLogoffRequest_Info, a2);
        if ( (a2 & 0x22000) == 0 && (a2 & 0xB) != 0 )
          WLEventWriteStartStopScenario(v10, &WLDiagEvt_ShutdownDiagnostics_Start, &stru_1400BEA70, a2);
        *a4 = AsyncLogoff((PVOID)a2);
        return 1i64;
      }
      *((_DWORD *)qword_1400DCB20 + 61) = 0;
    }
LABEL_19:
    if ( !(unsigned int)CSession::SetLsmSyncEvent(qword_1400DCB20)
      && WPP_GLOBAL_Control != (CUser *)&WPP_GLOBAL_Control
      && (*((_BYTE *)WPP_GLOBAL_Control + 28) & 0x40) != 0
      && *((_BYTE *)WPP_GLOBAL_Control + 25) >= 2u )
    {
      v9 = GetLastError();
      WPP_SF_d(*((_QWORD *)WPP_GLOBAL_Control + 2), 31i64, &WPP_e0c92ba51fd43896baa8c54dc4fddbcc_Traceguids, v9);
    }
    v8 = qword_1400DCB20;
    goto LABEL_25;
  }
  if ( !(unsigned int)CallCheckForHiberbootRpc(0, v7) )
  {
    *((_DWORD *)qword_1400DCB20 + 60) = 0;
    goto LABEL_19;
  }
  return 1i64;
}
```

![](https://cdn.luogu.com.cn/upload/image_hosting/qgabi458.png)

不难看出这里通过 `WlStateMachineSetSignal(3u, 0i64);` 触发了安全桌面。其中 `3u` 是触发安全桌面的信号。

但是这里仍然是处理软件模拟请求。

![](https://cdn.luogu.com.cn/upload/image_hosting/2wnqfv4v.png)

通过查询 `WlStateMachineSetSignal(3u, 0i64);` 交叉引用，它也在 `WMsgKMessageHandler` 中被调用。**推测**此处为处理来自硬件的中断。不论请求来自哪里，它们都具有共同的出口 `WlStateMachineSetSignal(3u, 0i64);`。故只需要 Hook 住它。安全桌面就无法弹出。

## 如何 Hook

接下来只需要提取特征码并进行 Hook。

```cpp
/*
 * WlStateMachineSetSignal 函数特征码 - 多个Windows版本
 * 
 * Win11版本 (直接开始，使用jmp):
 * 4C 8B CA             mov r9, rdx
 * 8B D1                mov edx, ecx
 * 48 8B 0D XX XX XX XX mov rcx, cs:lpMem
 * 48 85 C9             test rcx, rcx
 * 74 12                jz short ...
 * 4C 8B 41 20          mov r8, [rcx+20h]
 * 48 8B 09             mov rcx, [rcx]
 * 4D 8B 04 D0          mov r8, [r8+rdx*8]
 * E9 XX XX XX XX       jmp SignalManagerSetSignal
 * 
 * Win10版本 (有栈帧，使用call):
 * 48 83 EC 28          sub rsp, 28h
 * 4C 8B CA             mov r9, rdx
 * B8 0D 00 00 00       mov eax, 0Dh
 * 8B D1                mov edx, ecx
 * 48 8B 0D XX XX XX XX mov rcx, cs:qword_XXX
 * 48 85 C9             test rcx, rcx
 * 74 10                jz short ...
 * 4C 8B 41 20          mov r8, [rcx+20h]
 * 48 8B 09             mov rcx, [rcx]
 * 4D 8B 04 D0          mov r8, [r8+rdx*8]
 * E8 XX XX XX XX       call SignalManagerSetSignal
 * 
 * 共同特征码 (在函数中间，所有版本都有):
 * 4C 8B 41 20          mov r8, [rcx+20h]
 * 48 8B 09             mov rcx, [rcx]
 * 4D 8B 04 D0          mov r8, [r8+rdx*8]
 */
```

我这里写了 Hook 住该函数的 dll。

```cpp
/*
 * WlStateMachineHook.dll
 * 
 * 这个DLL使用MinHook库来Hook WlStateMachineSetSignal函数
 * 通过特征码搜索定位目标函数，拦截 WlStateMachineSetSignal(3, 0) 调用
 * 
 * 编译命令 (x64):
 * cl /LD /EHsc WlStateMachineHook.cpp /link lib\libMinHook.x64.lib /OUT:WlStateMachineHook.dll
 */

#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "MinHook.h"

#pragma comment(lib, "lib\\libMinHook.x64.lib")

// ============================================================================
// 类型定义
// ============================================================================

// StateMachineSignalData 结构体（根据原型推断）
struct _StateMachineSignalData;

// 原始函数类型定义
typedef __int64 (__fastcall *WlStateMachineSetSignal_t)(__int64 a1, struct _StateMachineSignalData* a2);

// ============================================================================
// 全局变量
// ============================================================================

// 原始函数指针（用于调用原始函数）
static WlStateMachineSetSignal_t fpOriginalWlStateMachineSetSignal = NULL;

// 目标函数地址
static LPVOID g_TargetFunctionAddress = NULL;

// 日志文件句柄
static HANDLE g_hLogFile = INVALID_HANDLE_VALUE;

// ============================================================================
// 日志功能
// ============================================================================

void WriteLog(const char* format, ...) {
    if (g_hLogFile == INVALID_HANDLE_VALUE) {
        return;
    }

    char buffer[1024];
    va_list args;
    va_start(args, format);
    int len = vsnprintf(buffer, sizeof(buffer) - 2, format, args);
    va_end(args);

    if (len > 0) {
        buffer[len] = '\r';
        buffer[len + 1] = '\n';
        buffer[len + 2] = '\0';

        DWORD bytesWritten;
        WriteFile(g_hLogFile, buffer, len + 2, &bytesWritten, NULL);
        FlushFileBuffers(g_hLogFile);
    }
}

void InitLog() {
    // 在DLL所在目录创建日志文件
    char logPath[MAX_PATH];
    GetModuleFileNameA(NULL, logPath, MAX_PATH);

    // 找到最后一个反斜杠，替换为日志文件名
    char* lastSlash = strrchr(logPath, '\\');
    if (lastSlash) {
        strcpy(lastSlash + 1, "WlStateMachineHook.log");
    } else {
        strcpy(logPath, "WlStateMachineHook.log");
    }

    g_hLogFile = CreateFileA(logPath, GENERIC_WRITE, FILE_SHARE_READ, NULL,
                             CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);

    if (g_hLogFile != INVALID_HANDLE_VALUE) {
        WriteLog("========================================");
        WriteLog("WlStateMachineHook DLL Loaded");
        WriteLog("========================================");
    }
}

void CloseLog() {
    if (g_hLogFile != INVALID_HANDLE_VALUE) {
        WriteLog("========================================");
        WriteLog("WlStateMachineHook DLL Unloaded");
        WriteLog("========================================");
        CloseHandle(g_hLogFile);
        g_hLogFile = INVALID_HANDLE_VALUE;
    }
}

// ============================================================================
// 特征码搜索
// ============================================================================

/*
 * WlStateMachineSetSignal 函数特征码 - 支持多个Windows版本
 * 
 * Win11版本 (直接开始，使用jmp):
 * 4C 8B CA             mov r9, rdx
 * 8B D1                mov edx, ecx
 * 48 8B 0D XX XX XX XX mov rcx, cs:lpMem
 * 48 85 C9             test rcx, rcx
 * 74 12                jz short ...
 * 4C 8B 41 20          mov r8, [rcx+20h]
 * 48 8B 09             mov rcx, [rcx]
 * 4D 8B 04 D0          mov r8, [r8+rdx*8]
 * E9 XX XX XX XX       jmp SignalManagerSetSignal
 * 
 * Win10版本 (有栈帧，使用call):
 * 48 83 EC 28          sub rsp, 28h
 * 4C 8B CA             mov r9, rdx
 * B8 0D 00 00 00       mov eax, 0Dh
 * 8B D1                mov edx, ecx
 * 48 8B 0D XX XX XX XX mov rcx, cs:qword_XXX
 * 48 85 C9             test rcx, rcx
 * 74 10                jz short ...
 * 4C 8B 41 20          mov r8, [rcx+20h]
 * 48 8B 09             mov rcx, [rcx]
 * 4D 8B 04 D0          mov r8, [r8+rdx*8]
 * E8 XX XX XX XX       call SignalManagerSetSignal
 * 
 * 共同特征码 (在函数中间，所有版本都有):
 * 4C 8B 41 20          mov r8, [rcx+20h]
 * 48 8B 09             mov rcx, [rcx]
 * 4D 8B 04 D0          mov r8, [r8+rdx*8]
 */

// 中间特征码 - 所有版本共有的指令序列
static unsigned char g_CommonSignature[] = {
    0x4C, 0x8B, 0x41, 0x20,             // mov r8, [rcx+20h]
    0x48, 0x8B, 0x09,                   // mov rcx, [rcx]
    0x4D, 0x8B, 0x04, 0xD0              // mov r8, [r8+rdx*8]
};

static size_t g_CommonSignatureLen = sizeof(g_CommonSignature);

// Win11函数开头特征 (无栈帧)
static unsigned char g_Win11Start[] = {
    0x4C, 0x8B, 0xCA,                   // mov r9, rdx
    0x8B, 0xD1                          // mov edx, ecx
};

// Win10函数开头特征 (有栈帧)
static unsigned char g_Win10Start[] = {
    0x48, 0x83, 0xEC, 0x28              // sub rsp, 28h
};

// 从共同特征位置向前回溯，找到函数真正的开头
LPVOID FindFunctionStart(unsigned char* commonPatternAddr) {
    // 向前检查，最多回溯64字节
    for (int offset = 0; offset < 64; offset++) {
        unsigned char* checkAddr = commonPatternAddr - offset;

        // 检查Win11版本开头 (4C 8B CA 8B D1)
        if (offset >= 5 && memcmp(checkAddr, g_Win11Start, sizeof(g_Win11Start)) == 0) {
            return (LPVOID)checkAddr;
        }

        // 检查Win10版本开头 (48 83 EC 28)
        if (offset >= 4 && memcmp(checkAddr, g_Win10Start, sizeof(g_Win10Start)) == 0) {
            return (LPVOID)checkAddr;
        }
    }

    return NULL;
}

// 在指定内存区域搜索特征码
LPVOID FindSignatureInRegion(LPVOID baseAddress, SIZE_T regionSize) {
    unsigned char* buffer = (unsigned char*)baseAddress;

    if (regionSize < g_CommonSignatureLen + 64) {  // 需要额外空间用于回溯
        return NULL;
    }

    // 从偏移64开始搜索，确保有足够的回溯空间
    for (SIZE_T i = 64; i <= regionSize - g_CommonSignatureLen; i++) {
        if (memcmp(buffer + i, g_CommonSignature, g_CommonSignatureLen) == 0) {
            // 找到共同特征码，现在向前回溯找函数开头
            LPVOID funcStart = FindFunctionStart(buffer + i);
            if (funcStart != NULL) {
                return funcStart;
            }
        }
    }
    return NULL;
}

// 在当前进程中搜索特征码
LPVOID FindSignatureInCurrentProcess() {
    MEMORY_BASIC_INFORMATION mbi;
    LPVOID currentAddress = NULL;

    WriteLog("Starting signature scan in current process...");

    while (VirtualQuery(currentAddress, &mbi, sizeof(mbi))) {
        // 检查是否为已提交的可执行内存区域
        if (mbi.State == MEM_COMMIT && 
            (mbi.Protect & (PAGE_EXECUTE | PAGE_EXECUTE_READ | PAGE_EXECUTE_READWRITE | PAGE_EXECUTE_WRITECOPY))) {

            // 跳过保护页
            if (!(mbi.Protect & PAGE_GUARD) && !(mbi.Protect & PAGE_NOACCESS)) {
                WriteLog("Scanning region: Base=0x%p, Size=0x%llX", mbi.BaseAddress, (unsigned long long)mbi.RegionSize);

                LPVOID foundAddr = FindSignatureInRegion(mbi.BaseAddress, mbi.RegionSize);
                if (foundAddr != NULL) {
                    WriteLog(">>> SIGNATURE FOUND at address 0x%p <<<", foundAddr);
                    return foundAddr;
                }
            }
        }

        // 移动到下一个内存区域
        currentAddress = (LPVOID)((DWORD_PTR)mbi.BaseAddress + mbi.RegionSize);
    }

    WriteLog("Signature not found in process memory.");
    return NULL;
}

// ============================================================================
// Hook 函数
// ============================================================================

/*
 * Hooked WlStateMachineSetSignal
 * 
 * 当 a1 == 3 且 a2 == NULL (0) 时拦截调用
 */
__int64 __fastcall HookedWlStateMachineSetSignal(__int64 a1, struct _StateMachineSignalData* a2) {
    WriteLog("WlStateMachineSetSignal called: a1=%lld, a2=0x%p", a1, (void*)a2);

    // 拦截 WlStateMachineSetSignal(3, 0) 调用
    if (a1 == 3 && a2 == NULL) {
        WriteLog(">>> INTERCEPTED: WlStateMachineSetSignal(3, 0) - Blocking this call!");
        // 返回 13 (0x0D)，这是函数在 qword_1400D0E28 为 NULL 时的默认返回值
        return 13;
    }

    // 其他调用正常传递给原始函数
    WriteLog("Passing through to original function...");
    return fpOriginalWlStateMachineSetSignal(a1, a2);
}

// ============================================================================
// 初始化和清理
// ============================================================================

BOOL InitializeHook() {
    MH_STATUS status;

    WriteLog("Initializing MinHook...");

    // 初始化 MinHook
    status = MH_Initialize();
    if (status != MH_OK) {
        WriteLog("MH_Initialize failed: %d", status);
        return FALSE;
    }
    WriteLog("MinHook initialized successfully.");

    // 搜索目标函数
    WriteLog("Searching for WlStateMachineSetSignal...");
    g_TargetFunctionAddress = FindSignatureInCurrentProcess();

    if (g_TargetFunctionAddress == NULL) {
        WriteLog("ERROR: Could not find WlStateMachineSetSignal function!");
        MH_Uninitialize();
        return FALSE;
    }

    WriteLog("Target function found at: 0x%p", g_TargetFunctionAddress);

    // 创建 Hook
    WriteLog("Creating hook...");
    status = MH_CreateHook(
        g_TargetFunctionAddress,
        (LPVOID)HookedWlStateMachineSetSignal,
        (LPVOID*)&fpOriginalWlStateMachineSetSignal
    );

    if (status != MH_OK) {
        WriteLog("MH_CreateHook failed: %d", status);
        MH_Uninitialize();
        return FALSE;
    }
    WriteLog("Hook created successfully.");

    // 启用 Hook
    WriteLog("Enabling hook...");
    status = MH_EnableHook(g_TargetFunctionAddress);
    if (status != MH_OK) {
        WriteLog("MH_EnableHook failed: %d", status);
        MH_Uninitialize();
        return FALSE;
    }

    WriteLog(">>> Hook enabled successfully! WlStateMachineSetSignal is now hooked. <<<");
    return TRUE;
}

void CleanupHook() {
    WriteLog("Cleaning up hooks...");

    if (g_TargetFunctionAddress != NULL) {
        MH_DisableHook(g_TargetFunctionAddress);
    }

    MH_Uninitialize();
    WriteLog("Hooks cleaned up.");
}

// ============================================================================
// DLL 入口点
// ============================================================================

BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {
    switch (ul_reason_for_call) {
        case DLL_PROCESS_ATTACH:
            // 禁用线程通知以提高性能
            DisableThreadLibraryCalls(hModule);

            // 初始化日志
            InitLog();
            WriteLog("DLL_PROCESS_ATTACH: Module handle = 0x%p", hModule);

            // 初始化 Hook
            if (!InitializeHook()) {
                WriteLog("ERROR: Failed to initialize hook!");
                CloseLog();
                return FALSE;  // 如果 hook 失败，可以选择返回 FALSE 阻止加载
            }
            break;

        case DLL_PROCESS_DETACH:
            WriteLog("DLL_PROCESS_DETACH");
            CleanupHook();
            CloseLog();
            break;

        case DLL_THREAD_ATTACH:
        case DLL_THREAD_DETACH:
            break;
    }
    return TRUE;
}

// ============================================================================
// 导出函数（用于手动触发/测试）
// ============================================================================

extern "C" __declspec(dllexport) BOOL ManualInitHook() {
    if (g_TargetFunctionAddress != NULL) {
        WriteLog("Hook already initialized.");
        return TRUE;
    }
    return InitializeHook();
}

extern "C" __declspec(dllexport) void ManualCleanupHook() {
    CleanupHook();
}

extern "C" __declspec(dllexport) LPVOID GetHookedFunctionAddress() {
    return g_TargetFunctionAddress;
}
```

只需要在用户态注入该 dll，即可用户态禁用安全桌面。

放个我编译好的：[hooking.dll](https://wwblw.lanzouv.com/ixp8H3icegmb)，密码 `3uyr`。

[^1]: [对极域64位禁止终止进程、键盘锁定的分析](https://blog.csdn.net/weixin_42112038/article/details/126228989)

*部分内容使用 Claude 润色，部分分析由 AI 辅助完成。我为其准确性负责。*
