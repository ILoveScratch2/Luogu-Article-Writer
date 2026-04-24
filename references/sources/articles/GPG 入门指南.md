GnuPG（GPG）是由 GNU 在 [PGP](https://en.wikipedia.org/wiki/Pretty_Good_Privacy) 基础上开发的加密软件。

通过 GPG，您可以为您的文件进行签名，加密等操作。

## 1 安装 GPG

对于 Linux 用户，可以通过包管理器进行安装。

对于 Windows 用户，可以下载 [gpg4win](http://www.gpg4win.org/)。gpg4win 提供图形化界面，不过笔者还是推荐在命令行下进行操作。

## 2 生成密钥

在终端下输入 `gpg --generate-key` 来生成一对新的密钥。

这时候 gpg 会询问您采用何种加密方式，默认情况下采用 RSA 进行加密和签名。

```cpp
请选择您要使用的密钥类型：
   (1) RSA 和 RSA （默认）
   (2) DSA 和 Elgamal
   (3) DSA（仅用于签名）
   (4) RSA（仅用于签名）
您的选择是？
```

（关于各加密方式的实现和安全性，可以自行查阅资料，这里不再赘述）

接下来 gpg 会询问生成密钥的长度。对于 RSA，默认情况下是 3072 位。为了安全起见，推荐将密钥长度设置为 4096 位。

在完成密钥长度的设置后，gpg 会询问密钥的过期时间。

```cpp
请设定这个密钥的有效期限。
         0 = 密钥永不过期
      <n>  = 密钥在 n 天后过期
      <n>w = 密钥在 n 周后过期
      <n>m = 密钥在 n 月后过期
      <n>y = 密钥在 n 年后过期
密钥的有效期限是？(0)
```

默认情况下密钥永远不会过期，如果你的密钥需要长期使用，且能保证自己能安全保管私钥，推荐设置为永不过期。

在完成上述设置后，gpg 会让你输入自己的姓名，邮箱等信息，这些信息将会用于签名。

最后一步是设置密码来确保私钥的安全。

在生成密钥的时间里，按照 gpg 的要求，随便敲敲你的键盘，让随机数生成器攒够熵吧！

然后你就生成了一对密钥。

**切记：保管好你的私钥！**

**切记：保管好你的私钥！**

**切记：保管好你的私钥！**

## 3 导出密钥

首先使用 `gpg --list-keys` 可以列出系统中已有的密钥。

列表中那一大串 hash 值是密钥的指纹（你也可以通过 `gpg --fingerprint [uid]` 的方式来查看密钥指纹）。

接下来我们将公钥导出，这样别人就可以用公钥加密文件，向我们发送了。

```cpp
gpg --armor --output public-key.txt --export [uid]
```

uid 可以是如下参数（只要能唯一确定密钥就行）：你的姓名，密钥指纹的后若干位。`--armor` 意味着导出 ASCII 文件。

你可以将这个公钥直接贴在网站上，让别人用这个公钥加密，但这样真的安全吗？

假如网站不安全，攻击者可能会把你的公钥换掉，换上攻击者的公钥，这样攻击者就可以拦截别人给你发的文件了。

一种简单的解决方案是把密钥上传到专门存储的服务器。

像这样：

```cpp
gpg --send-keys [uid] --keyserver pool.sks-keyservers.net
```

现在你只需告诉别人你的公钥指纹就行。别人利用这个公钥指纹就可以获取你的公钥，方法下面会讲到。

## 4 导入密钥

在得到公钥文件的前提下，通过 `gpg --import [密钥文件]` 的方式可以进行导入。

你也可以通过密钥服务器导入：

```cpp
gpg --keyserver pool.sks-keyservers.net --search-keys [uid]
```

## 5 文件签名

现在我们准备公开发布一个文件，为了确保这个文件不被篡改，我们需要给这个文件签名，以确保这个文件确实是本人发布的。

```cpp
gpg --armor --detach-sign example.txt
```

这个命令会生成一个 ASCII 签名文件 `example.txt.asc`。

现在我们将这个文件下载了下来，如果校验文件的真实性呢？

```cpp
gpg --verify example.txt.asc example.txt
```

## 6 文件加密

```cpp
gpg --recipient [uid] --output example.en.txt --encrypt example.txt
```

这个命令将通过 uid 的公钥进行加密，加密后的文件为 `example.en.txt`。

对方收到文件后，用私钥进行解密：

```cpp
gpg --decrypt example.en.txt --output example.txt
```

## 7 在 Github 上为你的提交签名

如果你在 Github 网页版上为某个仓库作出了提交，会看见这个提交带有一个“已验证”标识。

![](https://cdn.luogu.com.cn/upload/image_hosting/ub4jh7gf.png)

为提交签名表示提交是由本人作出的，防止其他人假借你的身份对仓库作出更改。

如果更改是在本地进行的，如何进行签名呢？

只需对 Git 进行如下配置：

```cpp
git config --global user.signingkey [uid]
```

然后在 commit 的时候加一个 `-S` 参数，就可以对你的提交签名了（注意 commit 时使用的邮箱应该和密钥的邮箱相同）。

（如果你想要每次提交都默认签名的话，执行 `git config --global commit.gpgsign true` 命令即可）

commit 的时候系统会提醒你输入当初添加私钥时配置的密码，以确认身份。

这样本地的提交就带有签名了。下一步是将本地提交推送到 Github，首先我们需要将密钥添加到 Github 账户。

在设置页面点击 SSH and GPG keys，接下来点击 New GPG key 按钮添加新密钥。你需要提交你的**公钥**。

到这里就差不多了。如果你 commit 时使用的邮箱绑定了 Github 账户，你就可以看见你的本地提交已经被验证。

![](https://cdn.luogu.com.cn/upload/image_hosting/54x74yaa.png)

## 8 补充说明

GPG 还可以为邮件加密，详情可以看邮件客户端的帮助。

在命令行下输入 `gpg --help` 可以获取 GPG 的帮助信息。

## Reference

- [GPG入门教程 - 阮一峰的网络日志](https://ruanyifeng.com/blog/2013/07/gpg.html)
- [管理提交签名验证 - Github 帮助](https://help.github.com/cn/github/authenticating-to-github/managing-commit-signature-verification)
