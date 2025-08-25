### mac安装破解软件报错解决方法

---

##### 打开应用提示「“xxx”已损坏」和「无法打开“xxx”」的解决方法

> #### 两种弹窗的解决方法

### 弹窗一

无法打开”xxx”，因为无法确认开发者的身份。
无法打开”xxx”，因为 Apple 无法检查其是否包含恶意软件。
无法打开”xxx”，因为它不是从App Store下载。
…

[![img](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202307082321565.png)](https://img.seemac.cn/wp-content/uploads/2021/04/1a76bd00fe00cd4c4242882e5a98fcd9.png)

解决方法：

在「应用程序」里找到安装好的应用图标，右键再点击打开，弹窗会改变（如截图），再点击打开即可

[![img](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202307082321634.png)](https://img.seemac.cn/wp-content/uploads/2021/04/516b2a7b9b9447a1b9f4ea1e30ad333a.png)

### 弹窗二

“xxx”已损坏，无法打开。您应该将它移到废纸篓。

[![img](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202307082321685.png)](https://img.seemac.cn/wp-content/uploads/2021/04/f6da960e292b38708be9af786a03e2b7.png)

解决方法（两步）：

**1. 先启用「任何来源」选项**

打开 系统偏好设置 -> 安全性与隐私，如图选择「任何来源」，如果你没有这个选项

[![img](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202307082321879.png)](https://img.seemac.cn/wp-content/uploads/2021/04/4ac80ef980217f3aaf131deca930ae0c.png)

 

打开系统应用「终端」，复制以下命令粘贴到终端，回车：

```
sudo spctl --master-disable
```

此时输入你的开机密码（密码不会显示），正常输入完再回车即可开启

[![img](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202307082321984.png)](https://img.seemac.cn/wp-content/uploads/2021/04/6a5dd85a3477094a1d481808d906253a.png)

**2. 绕过公证Gatekeeper（绕过应用签名认证）**

打开系统应用「终端」，先复制以下命令粘贴到终端，再加一个空格！然后在「应用程序」里找到“已损坏”的应用，拖放到终端窗口里（如截图），回车：

```
sudo xattr -rd com.apple.quarantine 
```

此时输入你的开机密码（密码不会显示），正常输入完再回车即可

[![img](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202307082321148.png)](https://img.seemac.cn/wp-content/uploads/2021/04/6ede628162648d5e12074b372028825f.png)

### 附：弹窗出现的原因

苹果默认只允许安装自家「App Store」里的应用，如果你想安装第三方的，那么需要在系统偏好设置 -> 安全性与隐私 中启用「App Store 和被认可的开发者」选项，而被认可的开发者是需要购买苹果的企业证书对应用进行签名，然后再提交给苹果审核才可以的，这一点对破解应用来说很不现实，因为破解应用必定会修改应用里的文件，从而导致签名失效。





##### macOS 运行应用出现「意外退出」及「崩溃闪退」问题修复方法



> #### 最近有部分网友反映更新系统后有很多软件打不开，或者出现闪退的情况，其实是因为Apple苹果公司在新系统中删除了TNT的证书。

### 应用签名

先安装Command Line Tools 工具，打开终端，复制以下命令粘贴到终端：

```
xcode-select --install
```

弹出安装窗口后选择安装，安装过程需要几分钟，请耐心等待。

[![img](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202307082322574.png)](https://img.seemac.cn/wp-content/uploads/2020/06/1612279435-833c35f77180f5c.png) [![img](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202307082322128.png)](https://img.seemac.cn/wp-content/uploads/2020/06/1612279437-2f35ab8b4b82db2.png) [![img](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202307082322381.png)](https://img.seemac.cn/wp-content/uploads/2020/06/1612279438-4b613c3d7a625bd.png)[![img](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202307082322616.png)](https://img.seemac.cn/wp-content/uploads/2020/06/1612279591-e329f24c629dc64.png)

安装完成后对应用进行签名，复制以下命令粘贴到终端：

```
sudo codesign --force --deep --sign -
```

注意最后的–后面加一个空格！然后打开Finder（访达），点击左侧的 应用程序，找到相关应用拖进终端，然后按下回车键，输入电脑的开机密码（输入过程中密码是看不到的）输入完成后再按下回车键即可。

正常情况下只有一行提示，即成功：

```
/文件位置 : replacing existing signature
```

如遇如下错误：

```
/文件位置 : replacing existing signature 
/文件位置 : resource fork,Finder information,or similar detritus not allowed
```

先在终端执行：

```
xattr -cr 应用路径（直接将应用拖进去）
```

然后再次执行如下指令即可：

```
codesign --force --deep --sign - 应用路径（直接将应用拖进去）
```

到这儿，百分之九十五的应用都可以正常运行了。如果还不行，那要关闭SIP了。



### 关闭SIP

以上操作如果还不能解决，那就需要关闭SIP系统完整性保护才可以了！[点我查看](https://www.seemac.cn/276.html)



##### SIP系统完整性保护关闭方法



> #### 许多Mac用户反应，装了部分软件后打不开，那可能是sip系统完整性没有关闭。下面我们就来看一下如何关闭sip系统完整性。

以下文章转载自Pertim

### SIP是什么？

系统完整性保护是 macOS 所采用的一项安全技术，能够帮助防止潜在恶意软件修改 Mac 上受保护的文件和文件夹。系统完整性保护可以限制 root 用户帐户，以及 root 用户能够在 Mac 操作系统的受保护部分完成的操作。

这可能对一些新人来说不太好理解，我们换一个说法，SIP 类似 Windows 的防火墙，安卓手机 Root，这应该可以解决很多的人困惑。

要更好的理解 SIP 可以做什么，我们需要先来了解一个概念：沙盒。

### 沙盒

macOS 自从 10.6 系统开始引入了沙盒机制，规定发布到 Mac App Store 中的应用必须使用并遵守沙盒约定。沙盒对应用访问的系统文件、硬件信息、网络等等都做了严格的限制，这样可以防止恶意的 App 通过系统漏洞来攻击系统并获取控制权限，也可以避免应用越权执行不安全的操作导致系统出现故障，从而保障 macOS 系统的安全。

沙盒相当于给每个 App 一个独立的空间，你只能在自己的小天地里面玩耍，要获取自己空间之外的资源必须获得授权（这个也有限制，只能获取有限的资源）。

现在你就大致了解沙盒是什么了。上面说了，因为 Mac App Store 中的应用必须要遵循沙盒约定，所以苹果应用商店的软件都是用沙盒运行的，无法访问修改系统底层文件，所以大部分软件如果想做的功能强一些又想上架 AppStore 那只能发阉割版的了。很多优秀软件没有在 Apple Store 上架就是因为需要一些沙盒外的资源权限，于是一些软件采取双版本，分为官方版全功能版和 App Store 精简版，就是因为这个原因，比如腾讯柠檬之类的系统应用。

你应该能也看出来，苹果为了大家的系统安全可真是煞费苦心。

### macOS 应用的安全划分

综上所述，我们可以把 macOS 应用按安全来划分为这三类：

- 沙盒运行：严格遵守苹果的沙盒机制，只能访问限定的目录及执行有限的操作
- 未关闭 SIP 且不使用沙盒运行：除系统底层受保护的文件外，通过用户授权后可以访问及修改任意文件
- 关闭 SIP 且不使用沙盒运行：几乎可以访问及修改所有系统文件。

### 关闭 SIP 的影响

关闭 SIP 后运行应用将不会再提示：

- 常见报错一：无法打开xxx，因为 Apple 无法检查其是否包含恶意软件/因为它来自身份不明的开发者/因为无法验证开发者
- 常见报错二：xxx已损坏，无法打开，您应该将它移到废纸篓

也就是说，只要应用本身可以运行，那不管应用是否签名/公证，不管应用是不是恶意应用，你打开后它都会直接运行在你的系统中，此时你的电脑如同透明，毫无安全可言，它可以操作你系统的所有文件，如果这个应用是恶意应用，如果你不小心对它授权过，那它后面可以不经你允许在你系统上进行任何操作。

所以如非必要，不建议大家关闭 SIP！如果一定要使用某个需要关闭 SIP 的应用，那一定要自行判断一下应用的来源是否安全。

### 检查SIP状态

在sip系统完整性关闭前，我们先检查是否启用了SIP系统完整性保护。

在终端（command+空格 聚焦搜索：终端）上输入以下命令然后回车：

```
csrutil status
```

你会看到以下信息中的一个，指示SIP状态

未关闭 enabled
System Integrity Protection status: enabled

已关闭 disabled
System Integrity Protection status: disabled

如果是未关闭状态就需要关闭SIP了!

### 如何关闭SIP

1. 关机，然后重新启动你的Mac电脑，在开机时一直按住 Command+R 迸入Recovery模式（m1改为长按电源键，点击选项，选择一个用户进去）
2. 进入Recovery模式后在顶部菜单栏点击 实用工具 -> 终端
3. 在终端上输入以下命令然后回车：

```
csrutil disable
```

4. 点击左上角苹果图标，点击重新启动


重新签名微信
sudo codesign --force --deep --sign - /Applications/WeChat.app/Contents/MacOS/WeChat
