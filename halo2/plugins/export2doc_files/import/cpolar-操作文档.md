### 1.入门指南 #

### 1.1.cpolar下载与安装 #

### 1.1.1.Windows #

在[官网下载](https://www.cpolar.com/download)下载适用于Windows平台的zip压缩包，解压后得到cpolar安装包，然后双击安装包一路默认安装即可。

![20221222163240](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717873.png)

![20221222163519](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717390.png)

### 1.1.2.linux #

- 自动安装方式：一键自动安装脚本
- 手动安装方式：在[官网下载](https://www.cpolar.com/download)下载适用于Linux平台的zip压缩包，解压后得到cpolar，然后通过命令行带参数运行即可。

### 一键自动安装脚本

#### 环境需求：

该脚本适用于Ubuntu16.04/18.04/20.04及以后，Centos7/8及以后版本，树莓派最新官方镜像，及支持systemd的新式Linux操作系统，该脚本会自动判断CPU架构（i386/amd64/mips/arm/arm64等等），自动下载对应cpolar客户端，并自动部署安装。

#### 1. cpolar 安装（国内使用）

```shell
curl -L https://www.cpolar.com/static/downloads/install-release-cpolar.sh | sudo bash
```

或 cpolar短链接安装方式：(国外使用）

```shell
curl -sL https://git.io/cpolar | sudo bash
```

#### 2. 查看版本号，有正常显示版本号即为安装成功

```shell
cpolar version
```

#### 3. token认证

登录cpolar官网[后台](https://dashboard.cpolar.com/get-started)，点击左侧的`验证`，查看自己的认证token，之后将token贴在命令行里

```shell
cpolar authtoken xxxxxxx
```

![20230111103532](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717526.png)

#### 4. 简单穿透测试

```shell
cpolar http 8080
```

按ctrl+c退出

#### 5. 向系统添加服务

```shell
sudo systemctl enable cpolar
```

#### 6. 启动cpolar服务

```shell
sudo systemctl start cpolar
```

#### 7. 查看服务状态

```shell
sudo systemctl status cpolar
```

![20221222164000](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717834.png)

#### 8. 登录后台，查看隧道在线状态

https://dashboard.cpolar.com/status

#### 9. 安装完成

可以参考系列文章进一步使用cpolar——[`linux系列教程文章`](https://www.cpolar.com/blog/build-a-website-on-ubuntu-system)

#### 注: cpolar 卸载方法

```shell
curl -L https://www.cpolar.com/static/downloads/install-release-cpolar.sh | sudo bash -s -- --remove
```

### 安装说明：

- cpolar默认安装路径 /usr/local/bin/cpolar,
- 安装脚本会自动配置systemd服务脚本，启动以后，可以开机自启动。
- 如果第一次安装，会默认配置一个简单的样例配置文件，创建了两个样例隧道，一个web，一个ssh
- cpolar配置文件路径: /usr/local/etc/cpolar/cpolar.yml

### 1.1.3.macOS #

在Mac系统上需通过Homebrew包管理器来安装cpolar内网穿透。

#### 1. 安装homebrew

Homebrew是一款Mac OS下的套件管理工具，拥有安装、卸载、更新、查看、搜索等很多实用的功能。

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 2. 安装cpolar

```shell
brew tap probezy/core && brew install cpolar
```

#### 3. token认证

登录cpolar官网，点击左侧的验证，查看自己的认证token，之后将token贴在命令行里

```shell
cpolar authtoken xxxxxxx
```

![202302221701](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717063.png)

#### 4. 安装服务

```shell
sudo cpolar service install
```

#### 5. 启动服务

```shell
sudo cpolar service start
```

#### 6. 安装完成

在浏览器上访问本地9200端口【[127.0.0.1:9200](https://www.cpolar.com/127.0.0.1:9200)】，使用cpolar邮箱账号登录cpolar web UI管理界面，即可开始使用cpolar。

![20230222170301](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717362.png)

#### 7. 系列文章

- [macOS系列教程](https://www.cpolar.com/blog/macos-builds-web-services-and-accesses-them-on-the-public-network)

### 1.1.4.群晖NAS #

> cpolar群晖套件下载地址：https://www.cpolar.com/synology-cpolar-suite

本次以x64 CPU架构的群晖型号举例。

### 1. 下载安装cpolar套件

下载相应版本的套件

![20221222170135](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717750.png)

打开`套件中心`，点击右上角的`手动安装`按钮。

![20221117182200](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717130.png)

选择我们本地下载好的cpolar安装包。

![20221117182209](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717289.png)

点击`下一步`按钮

![20221117182214](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717581.png)

点击`同意`按钮

![20221117182222](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717834.png)

之后，一路点击`下一步`按钮安装完成即可。

![20221117182230](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717929.png)

### 2. 打开cpolar套件

![20221117182235](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717937.png)

选择`已安装`，找到cpolar，点击

![20221117182241](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717205.png)

打开下面的Web管理界面，默认端口为9200

### 3. 登录cpolar Web管理界面

![20221117182246](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717299.png)

输入cpolar邮箱账号与密码进行登录

初次登录后，系统会在后台自动获取Authtoken，并且自动配置到配置文件中。

如果还没有账号，就登录官网注册。

登录成功后，进入主界面

![20221117182251](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717410.png)

主界面目前仅显示登录的管理员。

左侧菜单栏里，可以配置隧道，及查看在线隧道列表（仅本客户端的在线隧道列表）。

### 群晖系列教程

- *[NAS群晖系列教程](https://www.cpolar.com/blog/use-cpolar-to-access-synology-nas-remotely)*

### 1.1.5.威联通NAS #

本次教程将实现在NAS威联通上安装cpolar内网穿透。【注意威联通需为X86架构，支持安装docker镜像】

# 1. NAS威联通安装docker容器

进入App Centrer，下载容器工具`container station`

![20230214155001·](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717225.png)

# 2. 安装cpolar镜像

进入Container Station,然后选择创建,然后搜索`cpolar`,找到cpolar镜像，点击安装

![20230214155002](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717330.png)

安装时配置一下网络,点击打开高级设置

![20230214155003](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717418.png)

网络模式更改为host,然后点击创建

![20230214155004](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717510.png)

创建好后在列表中可以看到cpolar容器

![20230214155005](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717618.png)

打开浏览器访问威联通ip+:9200，就能访问到cpolar web UI管理界面，使用cpolar邮箱账号登录即可。

![20230214155006](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717765.png)

### 1.1.6.openwrt路由器 #

# 1.环境配置

通过ssh连接操作openwrt,下载公钥:

```shell
wget -O cpolar-public.key http://openwrt.cpolar.com/releases/public.key
```

![image-20230426162055194](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717549.png)

下载完成后添加公钥

```shell
opkg-key add cpolar-public.key
```

![image-20230426162154671](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717646.png)

添加cpolar的opkg仓库源

```shell
echo "src/gz cpolar_packages http://openwrt.cpolar.com/releases/packages/(. /etc/openwrt_release ; echoDISTRIB_ARCH)"  >>  /etc/opkg/customfeeds.conf
```

![image-20230426162323936](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717802.png)

更新仓库

```shell
opkg update
```

![image-20230426164614851](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717958.png)

# 2.安装cpolar内网穿透

开始安装cpolar内网穿透

```shell
opkg install cpolar
```

![image-20230426165214487](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717120.png)

```shell
opkg install luci-app-cpolar
opkg install luci-i18n-cpolar-zh-cn
```

![image-20230426165122463](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717301.png)

然后重启一下设备:

```shell
reboot
```

# 3.访问cpolar内网穿透管理界面

打开openwrt Web管理界⾯,我们可以看到有个`service`,下面即可看到cpolar

![image-20230426172304148](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717493.png)

进入后可能会出现以下问题,如果没有出现可以忽略.

![image-20230426173352997](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717675.png)

解决办法：更新仓库

```shell
opkg update
```

然后安装相关基础包

```shell
opkg install luci luci-base luci-compat
```

安装好后刷新几次页面即可看到cpolar界面

![image-20230426173631379](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717908.png)

登录cpolar官网[https://www.cpolar.com](https://www.cpolar.com/) ,如果没有注册账号,可以先注册账号,然后点击左侧的`验证`，查看自己的认证token，

```shell
cpolar authtoken xxxxxxx
```

![20230111103532](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717526.png)

之后将token码复制到openwrt cpolar服务界面的Auth Token里,设置后点击保存

![image-20230426174954224](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717112.png)

保存成功后点击界面上面`打开Web-UI界面`,即可看到cpolar web ui 界面

![image-20230426175417187](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717295.png)

使用我们cpolar官网注册的账号登陆,登陆后即可对隧道进行管理。至此，成功在openwrt上安装cpolar内网穿透，您可以在cpolar web UI管理界面自由创建隧道，实现公网访问内网。

![image-20230426175513987](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717508.png)

### 1.1.7.docker容器安装cpolar #

#### 1. 搜索cpolar镜像,输入命令:

```shell
docker search cpolar
```

![20230117153101](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717554.png)

#### 2. 在出现的列表中,选择第二个:probezy/cpolar下载,输入命令:

```shell
docker pull probezy/cpolar
```

#### 3. 下载好后,在本地镜像库即可看到,输入:

```shell
docker images
```

![20230117153102](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717603.png)

#### 4. 运行容器和cpolar镜像

```shell
docker run -id --network host --name cpolar probezy/cpolar
```

#### 5. 进入容器,输入命令:

```shell
docker exec -it cpolar /bin/bash
```

![20230117153104](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717672.png)

#### 6. 添加cpolar authtoken,打开cpolar官网,复制Authtoken

![20230117153105](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717746.png)

#### 7. 验证authtoken，访问cpolar官网并登录，然后点击左侧的验证，可以查看到你的token码

```shell
cpolar authtoken 复制的token
```

![20230117153106](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717798.png)

> 其中:/usr/local/etc/cpolar/cpolar.yml 是cpolar配置文件路径!

#### 8. 测试暴露8081端口测试,输入:

```shell
cpolar http 8081
```

出现以下信息表示成功

![20230117153107](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717842.png)

#### 9. 如需访问web ui 管理界面,输入http://本机ip:9200 即可访问

![20230117153108](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717920.png)

### 1.2.配置token认证令牌 #

cpolar服务的许多高级功能，需要与您注册的帐户关联才能使用。一旦您注册成功，您需要在后台仪表板上显示的authtoken参数配置cpolar客户端。它将授予您访问自己帐户的功能权限。 cpolar客户端有一个简单的’authtoken’命令，它使用起来很简单。

###### 安装你的认证token

```shell
cpolar authtoken <YOUR_AUTHTOKEN>
```

### 1.3.将本地Web服务器公开到Internet #

cpolar允许您将本地计算机上运行的Web服务器公开到Internet。告诉cpolar您的Web服务器正在侦听哪个端口。

如果您不知道Web服务器正在侦听哪个端口，则可能是80端口，即HTTP的默认端口。

示例：将本地计算机的端口80上的Web服务器公开到Internet

```shell
cpolar http 80
```

启动cpolar时，它将在终端中显示一个UI，其中包含隧道的公共URL以及有关通过隧道建立的连接的其他状态和度量信息。

cpolar控制台用户界面

```shell
cpolar by @bestExpresser

Tunnel Status                 online
Version                       2.0/2.0
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://92832de0.cpolar.io -> localhost:80
Forwarding                    https://92832de0.cpolar.io -> localhost:80

Connnections                  ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```

### 1.4.监听您的消息包流量 #

cpolar客户端提供实时Web UI管理界面，您可以在其中查看隧道上运行的所有HTTP流量。启动cpolar客户端后，只需在Web浏览器中打开http://localhost:4040 即可检查请求详细信息。

尝试向您的公开网址发出请求，完成后，请回顾检查Web UI管理界面。您将看到请求和响应包的所有详细信息，包括时间，持续时间，HEAD标头，查询参数和请求有效负载以及线路上的原始字节。

HTTP请求和响应的详细内容
[![cpolar-inspector](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717147.png)](https://i.loli.net/2018/12/02/5c0347f1cd5d7.png)

### 1.5.重放请求 #

开发外部API发布的Web API通常会要求您做一些工作（如微信公网众号用户发送消息）来触发回调请求，从而减慢开发周期。 cpolar允许您通过单击重放任何Http请求，这样大大加快了开发调试周期。点击网络检查用户界面上任意请求右上角的重放（Replay）按钮进行重放。

只需点击一下即可重放任何针对隧道网络服务器的请求
[![cplar replay](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717354.png)](https://www.cpolar.com/static/img/replay2.png)

### 1.6.请求消息体检查 #

cpolar特别支持Web上使用的最常见的数据交换格式。请求或响应正文中的任何XML或JSON数据都会自动为您打印并检查语法错误。

突出显示JSON语法错误的位置
[![cpolar 支持HTTP消息体语法检查](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717469.png)](https://www.cpolar.com/static/img/syntax.png)

### 2.Cpolar web UI #

> cpolar web UI管理界面，默认端口为`9200`。如需修改默认端口号，请查阅常见问题下的相关教程进行修改。

cpolar本地安装成功后，可通过浏览器访问cpolar web UI管理界面，以【 http://本地ip地址:9200 】形式访问，如

- *http://127.0.0.1:9200/*
- *或http://localhost:9200/*

均可访问到cpolar web ui管理界面【也可在同个局域网下不同的设备上进行访问】，如下图所示，使用cpolar账号登录即可：

![20221212161557](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717582.png)

> 注意：如提示`无法访问此网站，127.0.0.1 拒绝了我们的连接请求`或`Network Error`，请查阅常见问题。

cpolar web UI界面登录成功后，自动转入仪表盘界面。我们可以在这里创建隧道、编辑隧道、查看隧道信息、查看所生成的公网地址，查看系统状态等信息。

![20221212171524](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717697.png)

cpolar默认会安装两个样例隧道，可直接使用，或者编辑、删减：

- 一个是Website隧道指向http 8080端口
- 一个是ssh隧道，指向tcp 22端口（如为windows系统，则为remoteDesktop隧道，指向tcp 3389端口）

### 2.1.创建隧道 #

### 2.1.1.HTTP隧道 #

点击左侧仪表盘的`隧道管理`——`创建隧道`，填写隧道信息

- 隧道名称：可自定义，注意不要与现有隧道名称重复即可
- 协议：选择`http`
- 本地地址：填写所要映射的端口号，如8080
- 域名类型：免费套餐选择`随机域名`
- 地区：可自由选择服务器地区(注意China NAS地区仅供NAS套餐用户使用)

点击创建

![20221212173456](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717699.png)

隧道创建成功后，页面会自动跳转到`隧道列表`页面，可以看到所有数据隧道，包含刚刚创建成功的tunnel-1隧道

![20221212175809](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717807.png)

点击左侧仪表盘的状态——在线隧道列表，可以查看到本地所有在线隧道，以及所生成的公网地址。

可以看到刚刚所创建成功的tunnel-1隧道，已经有生成了相应的公网地址，复制到浏览器访问即可：

- 一个http协议地址
- 一个https协议地址（省去申请/配置证书的繁琐步骤）

![20221212181010](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717911.png)

### 2.1.2.TCP隧道 #

点击左侧仪表盘的`隧道管理`——`创建隧道`，填写隧道信息

- 隧道名称：可自定义，注意不要与现有隧道名称重复即可
- 协议：选择`tcp`
- 本地地址：填写所要映射的端口号，如22
- 域名类型：免费套餐选择`随机域名`
- 地区：可自由选择服务器地区(注意China NAS地区仅供NAS套餐用户使用)

点击创建

![20221214145506](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717971.png)

隧道创建成功后，页面会自动跳转到`隧道列表`页面，可以看到创建成功的tunnel-2隧道，状态正常为`active`

![20221214145616](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717039.png)

点击左侧仪表盘的状态——在线隧道列表，可以看到刚刚创建的tunnel-2隧道，已经有生成了相应的公网地址，本例为`6.tcp.cpolar.top:10577`，注意`tcp://`无需复制

![20221214145833](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717114.png)

### 2.1.3.FTP隧道 #

点击左侧仪表盘的`隧道管理`——`创建隧道`，填写隧道信息

- 隧道名称：可自定义，注意不要与现有隧道名称重复即可
- 协议：选择`ftp`
- 本地地址：填写所要映射的端口号，如21
- 域名类型：免费套餐选择`随机域名`
- 地区：可自由选择服务器地区(注意China NAS地区仅供NAS套餐用户使用)

![20221222145310](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717230.png)

隧道创建成功后，页面会自动跳转到`隧道列表`页面，可以看到创建成功的tunnel-3隧道，状态正常为`active`。

![20221214150359](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717323.png)

点击左侧仪表盘的状态——在线隧道列表，可以看到刚刚创建的tunnel-3隧道，已经有生成了相应的4个公网地址，分别对应信令端口以及数据端口：

| **信令端口（FTP://开头地址）** | **对应数据端口（DATA://开头地址）** |
| :----------------------------- | :---------------------------------- |
| 2.tcp.vip.cpolar.cn:28440      | 2.tcp.vip.cpolar.cn:28441           |
| /                              | 2.tcp.vip.cpolar.cn:28442           |
| /                              | 2.tcp.vip.cpolar.cn:28443           |

![20221214150456](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717388.png)

### 2.2.配置二级子域名 #

使用免费套餐所生成的公网地址为随机临时地址，其24小时内会发生变化。为此，我们可以为其配置一个二级子域名，指定一个容易记忆的地址进行访问，同时该二级子域名为固定不会随机变化。

### 2.2.1 保留二级子域名

> 需升级至基础套餐或以上才支持配置二级子域名

登录[cpolar官网后台](https://dashboard.cpolar.com/)，点击左侧仪表盘的`预留`，找到`保留二级子域名`，为http隧道保留一个二级子域名。

- *地区：选择服务器地区*
- *名称：填写您想要保留的二级子域名（可自定义）*
- *描述：即备注，可自定义填写*

![20221117174126](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717472.png)

本例保留一个名称为`ToDoList`的二级子域名。子域名保留成功后，我们将子域名复制下来，接下来需要将其配置到隧道中去。

![20221117174134](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717697.png)

### 2.2.2 配置二级子域名

在浏览器上访问[127.0.0.1:9200](http://127.0.0.1:9200/#/dashboard)，登录cpolar web ui管理界面。点击左侧仪表盘的`隧道管理`——`隧道列表`，找到需要配置二级子域名的隧道（本例中为website隧道），点击右侧的`编辑`

![20221117174141](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717883.png)

修改隧道信息，将二级子域名配置到隧道中：

- *域名类型：改为选择`二级子域名`*
- *Sub Domain：填写我们刚刚所保留的二级子域名（本例为`ToDoList`）*

修改完成后，点击`更新`

![20221117174151](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717130.png)

隧道更新成功后，点击左侧仪表盘的`状态`——`在线隧道列表`，可以看到website隧道的公网地址，已经更新为二级子域名了，将公网地址复制下来。

![20221117174157](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717439.png)

### 2.2.3 测试访问二级子域名

打开浏览器，我们来测试一下访问配置成功的二级子域名。

![20221117174205](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717462.png)

测试成功，可以正常访问。现在，我们全网唯一的私有二级子域名，就创建好了。

### 2.3.配置自定义域名 #

cpolar支持配置使用您自己的域名，您可以在运行cpolar隧道上使用自己的域名，通过您自己的域名来访问本地服务，而不是比如cpolar.io这样的子域名。

### 2.3.1 保留自定义域名

> 需升级至专业套餐或以上才支持配置自定义域名

登录[cpolar官网后台](https://dashboard.cpolar.com/)，点击左侧的`预留`选项，找到`保留自定义域名`项目。

![20221117174918](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717520.png)

- 在`保留自定义域名`项下，填入我们购买的域名，而这里的域名需要带前缀和后缀，例如本例使用的域名为`www.woxihuanbiancheng.xyz` ，因此就要将全段填入“域名”项；
- `地区`的选择我们需要注意：国内所有自定义域名都需要提前已备案，才能被正常访问，而国外地区的则可不必备案（国内地区包括cn、cn_vip、cn_top；海外有us、hk）；
- `描述`项则是我们区别数据隧道的说明，只要输入方便区分的标识即可。

![20221117174924](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717612.png)

地址保留成功后，系统会默认给出一个CNAME值，这个值是域名供应商端连接cpolar数据端口的端口

![20221117174932](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717754.png)

### 2.3.2 域名解析

在cpolar端口设置完成后，我们再转入域名供应商部分，对我们购买的域名进行设置，让购买的域名能与cpolar连接起来。在这个例子中，我们购买的域名平台为阿里云平台，因此需要转入阿里云的域名控制台。

![20221117174937](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717917.png)

在“域名控制台”下“域名列表”页面，我们可以在打算使用的域名后，找到“解析”键，点击解析，就能对该域名所要连接的网站地址进行设置。

![20221117174942](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717164.png)

![20221117174948](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717273.png)

点击左上的“添加记录”后，会跳出新的页面，我们要在这个页面填入正确的信息，才能成功完成域名的解析工作。而具体的设置内容，如以下所示。

- 第一项“记录类型”，我们在下拉框中选择“CNAME”；

![20221117174954](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717393.png)

- 第二项“主机记录”，填入“www”。需要注意的是，如果所购买的域名有多个内容，则需要填写完整（如所购域名为www.abc.efgh.com），则需要完整填写。

![20221117175000](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717462.png)

- 第三项“主机记录”我们可不必填写，仅使用默认即可。第四项“记录值”，就是cpolar生成的那串复杂的字符，我们直接复制至此空即可。第五项“TTL”我们也保持默认内容。

![20221117175006](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717601.png)

各项内容填写完毕后，就可点击下方“确认”键，进入下一步。此时会自动返回解析设置页面，这是我们只能等待阿里云服务器对输入的信息（特别是记录值）进行分析关联（通常为10分钟）。

![20221117175012](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717725.png)

### 2.3.3 配置自定义域名

浏览器访问本地9200端口，登录cpolar web UI管理界面。点击左侧“隧道管理”——“隧道列表”，从列表中找到需要进行配置的隧道，点击右侧的“编辑”按钮。

![20221117175019](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717867.png)

- 域名类型：改为选择`自定义域名`
- 域名名称：填入刚刚在配置解析的域名

点击`更新`

![20221117175023](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717971.png)

cpolar会自动返回“隧道列表”页面，并显示数据隧道为正常启动状态。

![20221117175028](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717078.png)

点击左侧仪表盘的状态——在线隧道列表，也可以看到公网地址已经更新成为了自己的域名。

测试访问一下，访问成功。

![20221117175034](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717243.png)

### 2.3.4 关于服务器选择以及域名备案的说明

#### 关于服务器地区的选择：

大陆地区有China、China vip、China Top、China VIP Top、China NAS地区

海外地区有United States、Hong Kong、Taiwan、Europe地区

#### 关于自定义域名备案：

如服务器选择大陆地区，所有自定义域名都需要提前已备案，才能部署。

如服务器选择海外地区，则不需要备案，也无需过白名单步骤。

#### 关于域名过白名单：

如果您的域名已备案，可以部署在大陆地区：

- 如选择cn、cn_top、cn_vip_top地区，域名自动过白名单；
- 如选择cn_vip、cn_nas地区，域名需要人工过白名单，请联系官网在线客服人员或者官网QQ，让他帮您提交过白申请。

如果您的域名未备案，请使用海外地区：us、hk、tw、eu地区，不需要域名过白名单步骤，待备案完成后再修改为大陆地区服务器。

### 2.4.配置固定TCP端口地址 #

使用免费套餐所生成的公网地址为随机临时地址，其24小时内会发生变化。对于需要长期远程连接的用户来讲，为其配置一个固定的公网TCP端口地址尤为重要。

### 2.4.1 保留固定TCP地址

> 需升级至专业套餐或以上才支持保留固定TCP端口地址

登录cpolar官网，进入后台，点击左侧仪表盘的[预留](https://dashboard.cpolar.com/reserved)。找到`保留的TCP地址`，本例为远程桌面保留一个固定tcp地址：

- 地区：选择服务器地区
- 描述：远程桌面（可自定义备注）

![20221117173526](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717334.png)

TCP地址保留成功后，会生成相应的公网TCP端口地址，将其复制下来。

![20221117173532](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717427.png)

### 2.4.2 配置固定TCP端口地址

在浏览器上访问[127.0.0.1:9200](http://127.0.0.1:9200/#/dashboard)，登录cpolar web ui管理界面。点击左侧仪表盘的`隧道管理`——`隧道列表`，找到所要配置的TCP隧道，点击`编辑`

![20221117173538](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717557.png)

修改隧道信息

- *端口类型：改为选择`固定TCP端口`*
- *预留的TCP地址：填入刚刚保留成功的公网TCP端口地址*

修改完成后，点击`更新`

![20221117173558](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717686.png)

### 2.4.3 查看公网地址

在左侧仪表盘的`状态`——`在线隧道列表`，可以看到已经更新为固定的公网TCP端口地址了。

![20221117173618](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717827.png)

### 2.5.配置固定FTP端口地址 #

使用免费套餐所生成的公网地址为随机临时地址，其24小时内会发生变化，需要重新配置ftp服务，对于需要长期访问的用户来讲非常不方便，因此，配置固定的公网端口地址就很重要。

### 2.5.1 预留FTP固定公网地址

> 需升级至专业套餐或以上才支持保留固定FTP端口地址

登录cpolar后台，进入预留页面

![20221117181416](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717061.png)

选择`保留的FTP地址`，保留一个固定的FTP公网地址

- *地区：选择China或者China vip*
- *描述：可自定义*

![20221117181423](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717147.png)

```
FTP地址保留成功
```

由于穿透FTP不止需要穿透21端口，还需要穿透数据端口，所以保留成功ftp地址后，除了会生成一个信令端口（公网对应本地的21端口）以外，还会生成一个数据端口段。

![20221117181429](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717237.png)

### 2.5.2 配置固定FTP地址

浏览器访问[127.0.0.1:9200](http://127.0.0.1:9200/#/dashboard)，登录本地cpolar web-ui管理界面

![20221117181435](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717327.png)

点击左侧仪表盘——隧道管理——隧道列表，找到相应的FTP隧道，点击右侧的编辑：

- *端口类型：改为选择固定ftp端口*
- *预留的ftp地址：输入在cpolar后台所保留成功的地址*

点击`更新`

![在这里插入图片描述](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717715.jpeg)

#### 2.5.3 获取固定公网地址

隧道创建成功后，可以看到ftp隧道为激活状态。

![20221117181446](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717860.png)

左侧仪表盘——状态——在线隧道列表，可查看到ftp隧道所生成的4条隧道，分别对应信令端口以及数据端口：

> 信令端口（ftp://开头地址）：
> – ftp://1.tcp.cpolar.cn:25124
>
> 对应数据端口（data://开头地址）（全例为25125——25127）：
> – data://1.tcp.cpolar.cn:25125
> – data://1.tcp.cpolar.cn:25126
> – data://1.tcp.cpolar.cn:25127

![20221117181455](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717049.png)

### 3.命令行创建隧道 #

除了在cpolar web UI管理界面创建隧道外，我们还可以通过一行命令创建隧道将本地服务映射到公网，实现公网访问内网。

> 注意：不要在前台打开多个命令行创建隧道，否则会提示登录实例超过限制。您可以将其配置为后台服务，或者直接通过cpolar web UI管理界面来创建隧道。

### 3.1.HTTP隧道 #

如创建http隧道，指向本地8080端口的web服务

```shell
cpolar http 8080
```

如下图所示，`Forwarding`项为所生成的公网地址，将其复制下来在浏览器访问，即可访问到本地8080端口下的web服务。

![20221226165745](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717278.png)

如需指定服务器地区（默认情况下为China Top），请添加使用`-region`参数，如使用China VIP地区：

```shell
cpolar http -region=cn_vip 80
```

![20221227113404](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717484.png)

`ctrl+C` 关闭隧道，如需长期运行，请将其配置为后台服务。

### 3.2.TCP隧道 #

如创建tcp隧道，指向本地22端口的ssh服务

```shell
cpolar tcp 22
```

如下图所示，`Forwarding`项为所生成的公网地址+公网端口号

![20221226171123](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717604.png)

如需指定服务器地区（默认情况下为China Top），请添加使用`-region`参数，如使用China VIP地区（具体关于服务器地区请参考官网文档——全球基础设施）：

```shell
cpolar tcp -region=cn_vip 22
```

![20221227113720](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717709.png)

公网用户测试SSH连接

```shell
ssh root@2.tcp.vip.cpolar.cn -p 14170
```

> 注意：由于公网端口变成20046，所以请求时，ssh命令要加上-p参数, 值为指定的cpolar公网端口号。

`ctrl+C` 关闭隧道，如需长期运行，请将其配置为后台服务。

### 3.3.将隧道配置为后台服务 #

在前台终端所创建的隧道，页面一旦关闭，隧道也会同步关闭。我们可以将其配置服务，在后台运行。

> 如为windows系统，无需额外配置服务，安装后即默认配置服务
> 如为linux系统，请参考官网文档的入门教程——下载与安装——linux，参考教程安装服务

服务安装并启动成功后，编辑cpolar配置文件

打开cpolar配置文件（默认配置文件路径请参考官网文档——配置文件）

**示例配置文件**

```yaml
authtoken: xxxxxxxxxxxx [[认证token]]

tunnels:
  ssh:              [[隧道名称，表示ssh，名称可以自定义]]
    addr: 22        [[端口号为22]]
    proto: tcp      [[协议tcp]]
    region: cn_vip  [[地区，cn_vip，可选]]:us,hk,cn,cn_vip
  website:          [[隧道名称，用户可以自定义，但多隧道时，不可重复]]
    addr: 8080      [[本地Web站点端口]]
    proto: http     [[协议http]]
    region: cn_vip  [[地区，cn_vip]],可选：us,hk,cn,cn_vip
```

如创建一条http协议隧道：

- *隧道名称：tunnel-1*
- *协议：http协议*
- *本地地址：本地80端口*
- *地区：China VIP*
  编辑配置文件如下所示：

```yaml
authtoken: xxxxxxxxxxxx [[认证token]]

tunnels:
  ssh:
    addr: 22
    proto: tcp
    region: cn_vip
  website:
    addr: 8080
    proto: http
    region: cn_vip
  tunnel-1:
    addr: 80
    proto: http
    region: cn_vip
```

> 注意： 配置文件是yaml格式的，缩进敏感，而且不能有TAB键。

然后按CTRL+X,退出，提示你是否保存，回答Y，确认保存文件路径，回车.

测试启动tunnel-1隧道

```shell
cpolar start tunnel-1
```

![20221227141909](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717826.png)

如上图显示，则为正常，按CTRL+C退出。如果报错，会提示配置文件某行有错误，请重新修改。直到类似上图正确输出。

重新启动cpolar服务

```shell
sudo systemctl restart cpolar
```

### 3.4.配置二级子域名 #

> 注意需要升级至基础套餐或以上才支持配置二级子域名

本例配置一个二级子域名为`todolist`的隧道

#### 1. 保留二级子域名

登录cpolar官网后台——>`预留`——>`保留二级子域名`选项卡下

- 地区：选择`China VIP`
- 二级子域名：填写`todolist`(填写您想要保留的二级子域名，可自定义)
- 描述：即备注，可自定义填写

填写完成后，点击`保留`

![1665737652874](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717889.jpg)

二级子域名保留成功，将子域名`todolist`复制下来，接下来需要将其配置到隧道中去。

![1665737715046](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717948.jpg)

#### 2. 创建一条二级子域名为`todolist`的隧道

请使用`-subdomain`参数

```sehll
cpolar http -subdomain=ToDoList 80
```

![20221227135713](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717049.png)

`ctrl+C` 关闭隧道，如需长期运行，请将其配置为后台服务。

#### 3. 编辑cpolar配置文件

需要在指定隧道下添加`subdomain`参数:

```yaml
authtoken: xxxxxxxxxx
tunnels:
  todolist:
    proto: http
    addr: 80
    subdomain: todolist
    region: cn_vip
```

保存配置文件退出。

重新启动cpolar服务

```shell
sudo systemctl restart cpolar
```

### 3.5.配置自定义域名 #

> 注意需要升级至专业套餐或以上才支持配置自定义域名

#### 1. 保留自定义域名

登录cpolar官网后台，点击左侧的`预留` –> `保留自定义域名` 选项卡下：

- 地区：请根据实际情况选择
- 域名：输入您的域名，本例为`dev.bestexpresser.com`
- 描述：可自定义备注

点击`保留`按钮。

![1.0-QQ20210707-063917@2x](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717171.png)

```shell
1. 关于服务器地区的选择：

   大陆地区有China、China vip、China Top、China VIP Top、China NAS地区;
   海外地区有United States、Hong Kong、Taiwan、Europe地区.

2. 关于自定义域名备案：

   如服务器选择大陆地区，所有自定义域名都需要提前已备案，才能部署;
   如服务器选择海外地区，则不需要备案，也无需过白名单步骤。

3. 关于域名过白名单：

   如您的域名已备案，可以部署在大陆地区：
     – 如选择cn、cn_top、cn_vip_top地区，域名自动过白名单；
     – 如选择cn_vip、cn_nas地区，域名需要人工过白名单，请联系官网在线客服人员或者官网QQ，让他帮您提交过白申请。
   如您的域名未备案，请使用海外地区：us、hk、tw、eu地区，则不需要域名过白名单步骤。
```

自定义域名保留成功后，系统会生成相应的CNAME目标地址，将其复制下来

![2.0-QQ20210707-065455@2x](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717918.png)

#### 2. 自定义域名解析

在您自己的域名提供商，DNS解析中，加入一条CNAME记录。在此示例中：

假设本示例中的域名`bestexpresser.com`是在阿里云注册的，则我们需要登录阿里云，在阿里云的`云解析DNS`里，`bestexpresser.com`域名下，添加一条`CNAME`记录。

![3-QQ20210707-065800@2x](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717414.png)

- 设置记录类型为`CNAME`
- 名称: `dev`
- 记录值: `5983fcc1.cname.cpolar.io`

点击`确认`

![4-QQ20210707-065949@2x](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717960.png)

添加后的效果,解析生效需要等待10分钟

![5-QQ20210707-070041@2x](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717132.png)

大约5-10分钟后，ping您的自定义域名(dev.bestexpresser.com)，看是否已经解析到了cpolar提供cname地址

```shell
ping dev.bestexpresser.com
```

![6-QQ20210707-071908@2x](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717488.png)

如果ping返回的地址中，包含cpolar的cname地址，说明已经解析成功。

#### 3. 创建自定义域名隧道

注意请使用`-hostname`参数，值为您的域名

```shell
cpolar http -hostname=dev.bestexpresser.com 8080
```

配置成功，您现在就可以用dev.bestexpresser.com自定义域名访问本地站点了。

`ctrl+C` 关闭隧道，如需长期运行，请将其配置为后台服务。

#### 4. 编辑cpolar配置文件

需要在指定隧道下添加`hostname`参数:

```yaml
authtoken: xxxxxxxxxx
tunnels:
  website:
    proto: http
    addr: 8080
    hostname: dev.bestexpresser.com
    region: us
```

保存配置文件退出

#### 5. 测试配置文件

```shell
cpolar start website
```

如图，说明配置成功

![20221227154213](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717751.png)

重新启动cpolar服务

```shell
sudo systemctl restart cpolar
```

> 注意：此时通过HTTPS访问自定义域隧道仍然有效，但证书不匹配。接下来，我们来添加证书。

#### 6. 配置SSL证书

当使用自己的自定义域名时，您可能无法访问公开的https服务，因为HTTPS证书不同。 cpolar客户端可以轻松为您完成此操作，使您可以端到端加密通迅，而不必担心本地服务是否具有HTTPS支持。 您只需要在命令行中指定-crt和-key两个参数，以指定TLS证书和密钥的文件系统路径，cpolar客户端将负责为您握手TLS连接。

自定义域名的HTTPS证书，您可以从DNS运营商那里免费获得或购买。

如果您已经有了TLS证书/密钥对，您可以尝试使用按如下命令创建自定义域名隧道。

```shell
cpolar http -hostname=dev.bestexpresser.com -key=/path/to/tls.key -crt=/path/to/tls.crt 8080
参数说明：

hostname: 您的自定义域名
crt: HTTPS证书文件路径（全路径）
key: HTTPS证书密钥文件路径（全路径）
8080:本地web服务器侦听端口
```

配置成功，您即可以使用配置了SSL证书的自定义域名访问本地站点了。

`ctrl+C` 关闭隧道，如需长期运行，请将其配置为后台服务。

#### 7. 编辑cpolar配置文件

需要在指定隧道下添加`crt`以及`key`参数:

```yaml
authtoken: xxxxxxxxxx
tunnels:
  website:
    proto: http
    addr: 8080
    hostname: dev.bestexpresser.com
    region: us
    crt: /path/to/tls.crt
    key: /path/to/tls.key
```

保存配置文件退出

测试配置文件

```shell
cpolar start website
```

如正常显示即为配置成功

重新启动cpolar服务

```shell
sudo systemctl restart cpolar
```

### 3.6.配置固定TCP地址 #

> 注意需要升级至专业套餐或以上才支持配置固定tcp地址

#### 1. 保留固定TCP地址

登录cpolar官网后台——>`预留`——>`保留TCP地址`选项卡下，保留一个固定TCP地址

- 地区：选择China VIP
- 描述：可自定义备注

点击`保留`

![20221227143557](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717868.png)

地址保留成功后，系统会自动生成相应的公网地址+公网端口号，将其复制下来

![20221227143846](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717109.png)

#### 2. 配置固定TCP地址

举例创建一条固定TCP地址的隧道，指向本地22端口，SSH服务。

在调用cpolar时请使用`-remote-addr`选项参数，值为系统分配给您的保留TCP地址

```shell
cpolar tcp -remote-addr=1.tcp.vip.cpolar.cn:23539 22
```

![20221227144308](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717477.png)

`ctrl+C` 关闭隧道，如需长期运行，请将其配置为后台服务。

#### 3. 编辑cpolar配置文件

需要在指定隧道下添加`remote_addr`参数:

```yaml
authtoken: xxxxxxxxxx
tunnels:
  ssh:
    proto: tcp
    addr: 22
    remote_addr: 1.tcp.vip.cpolar.cn:23539
    region: cn_vip
```

> 注意:配置文件中的remote_addr参数为下划线，与命令行中使用的-remote-addr中划线不同。

保存配置文件并退出。

测试配置文件

```shell
cpolar start ssh
```

如正常显示即为配置成功

重新启动cpolar服务

```shell
sudo systemctl restart cpolar
```

### 4.cpolar配置文件 #

有时您的cpolar配置太复杂，无法在命令行选项中表达。 cpolar支持一个可选的，非常简单的YAML配置文件，它为您提供同时运行多个隧道的功能，以及调整一些cpolar更神秘的设置。

### 4.1.指定配置文件路径 #

您可以使用`-config`选项将路径传递给显式配置文件。 建议所有生产部署使用此方法。

明确指定配置文件位置

```shell
cpolar http -config=/opt/cpolar/conf/cpolar.yml 8000
```

### 4.2.默认配置文件路径 #

如果没有为配置文件指定位置，则cpolar会尝试从默认位置`$HOME/.cpolar/cpolar.yml`中读取一个位置。 配置文件是可选的; 如果该路径不存在，则不会发出错误。

在默认路径中，$HOME是操作系统定义的当前用户的主目录。 **它不是环境变量$HOME**, 尽管它们通常是相同的。 对于主要操作系统，如果您的用户名是`john`，则可能会在以下路径中找到默认配置：

| 操作系统 | 默认配置文件路径                 |
| :------- | :------------------------------- |
| OS X     | /Users/john/.cpolar/cpolar.yml   |
| Linux    | /home/john/.cpolar/cpolar.yml    |
| Windows  | C:\Users\john\.cpolar\cpolar.yml |

### 4.3.配置选项 #

### `authtoken`

此选项指定用于在连接到cpolar服务时对此客户端进行身份验证的身份验证令牌。 创建cpolar帐户后，您的后台管理界面将显示分配给您帐户的authtoken。

cpolar.yml指定一个authtoken

```shell
authtoken: 4nq9771bPxe8ctg7LKr_2ClH7Y15Zqe4bWLWF9p
```



### `console_ui`

| `true`  |         | 启用控制台UI界面                              |
| ------- | ------- | --------------------------------------------- |
| `false` |         | 禁用控制台UI界面                              |
| `iftty` | default | 仅当标准输出是TTY（不是文件或管道）时才启用UI |



### `console_ui_color`

| `transparent` |         | 显示控制台UI时不要设置背景颜色 |
| ------------- | ------- | ------------------------------ |
| `black`       | default | 将控制台UI的背景设置为黑色     |



### `http_proxy`

用于建立隧道连接的HTTP代理的URL。 许多HTTP代理具有连接大小和持续时间限制，这将导致cpolar失败。 像许多其他网络工具一样，如果设置了cpolar，它也会尊重环境变量`http_proxy`。

经过身份验证的HTTP代理上的cpolar示例

```shell
http_proxy: http://user:password@proxy.company:3128
```



### `socks5_proxy`

用于建立与cpolar服务器的连接的SOCKS5代理服务的地址。

经过身份验证的socks5代理的cpolar示例

```shell
socks5_proxy: socks5://user:password@proxy.company:1080
```



### `inspect_db_size`

| positive integers |         | 内存上限的大小（以字节为单位），用于分配以通过HTTP隧道保存请求以进行检查和重放。 |
| ----------------- | ------- | ------------------------------------------------------------ |
| `0`               | default | 使用默认分配限制，50MB                                       |
| `-1`              |         | 禁用监听数据库; 这具有禁用所有隧道监听的有效行为             |



### `log_level`

根据日志的记录详细程度，依次可能值是:`crit`,`warn`,`error`,`info`,`debug`



### `log_format`

输出日志记录的格式。

| `logfmt` |         | 人机友好的键/值对                                     |
| -------- | ------- | ----------------------------------------------------- |
| `json`   |         | 换行符分隔的JSON对象                                  |
| `term`   | default | 如果标准输出是TTY，则自定义彩色格式，否则相同`logfmt` |



### `log`

将日志写入此目标。

| `stdout` |         | 写入到标准输出                 |
| -------- | ------- | ------------------------------ |
| `stderr` |         | 写入到标准错误输出             |
| `false`  | default | 禁用日志输出                   |
| 其它值   |         | 将日志记录写入磁盘上的文件路径 |

```shell
log: /var/log/cpolar.log
```



### `metadata`

不透明的，用户提供的字符串，将作为cpolar.com API响应的一部分返回给此客户端启动的所有隧道的List Online Tunnels资源。 这是通过您自己的设备或客户标识符识别隧道的有用机制。 最多4096个字符。

```shell
metadata: bad8c1c0-8fce-11e4-b4a9-0800200c9a66
```



### `region`

选择cpolar客户端将连接的区域来托管其隧道。

| `us` | default | 美国     |
| ---- | ------- | -------- |
| `cn` |         | 中国     |
| `hk` |         | 香港     |
| `eu` |         | 欧洲     |
| `ap` |         | 亚太     |
| `au` |         | 澳大利亚 |



### `root_cas`

根证书颁发机构用于验证与cpolar服务器的TLS连接。

| `trusted` | default | 仅使用cpolar.com隧道服务的可信证书根目录                     |
| --------- | ------- | ------------------------------------------------------------ |
| `host`    |         | 使用主机操作系统信任的根证书。 您可能希望使用此选项连接到第三方cpolar服务器。 |
| 其它值    |         | 指定路径的颁发机构要信任的磁盘上的证书PEM文件                |

### `socks5_proxy`

用于建立与cpolar服务器的连接的SOCKS5代理的URL。

```shell
socks5_proxy: "socks5://localhost:9150"
```



### `tunnels`

隧道定义的名称映射。有关详细信息，请参阅隧道定义



### `update`

| `true`  |         | 如果可用，自动将cpolar更新为最新版本     |
| ------- | ------- | ---------------------------------------- |
| `false` | default | 除非用户手动启动，否则永远不要更新cpolar |



### `update_channel`

更新通道确定要更新的已发布版本的稳定性。 对所有生产部署使用“稳定”。

| `stable` | 默认 | 在可用时更新到稳定版本     |
| -------- | ---- | -------------------------- |
| `beta`   |      | 在可用时更新到新的beta版本 |



### `web_addr`

要绑定的网络地址，用于提供本地Web界面和api。

| 网络地址         |      | 绑定到此网络地址 |
| ---------------- | ---- | ---------------- |
| `127.0.0.1:4040` | 默认 | 默认网络地址     |
| `false`          |      | 禁用Web UI       |

### 4.4.隧道定义 #

配置文件的最常见用途是定义隧道配置。 定义隧道配置很有用，因为您可以从命令行按名称启动预配置的隧道，而不必每次都记住所有正确的参数。

隧道定义为配置文件中`tunnels`属性下的name – > configuration映射。

定义两个名为`httpdev`和`demo`的隧道

```yaml
tunnels:
  httpdev:
    proto: http
    addr: 8000
    subdomain: alan
  demo:
    proto: http
    addr: 9090
    hostname: demo.bestexpresser.com
    inspect: false
    auth: "demo:secret"
```

启动名为`httpdev`的隧道

```shell
cpolar start httpdev
```

您定义的每个隧道都是配置选项名称到值的映射。 配置选项的名称通常与其对应的命令行开关相同。 每个隧道都必须定义`proto`和`addr`。 其他属性可用，许多属性是特定于协议的。
隧道配置属性

等。

| `proto`          | requiredall | 隧道协议名称， `http`, `tcp`可选。                           |
| ---------------- | ----------- | ------------------------------------------------------------ |
| `addr`           | requiredall | 将流量转发到该本地端口号或网络地址                           |
| `region`         | all         | 全球地理区域，默认 `us` 美国区域,可选 `cn` 中国区域          |
| `inspect`        | all         | 启用监听HTTP请求                                             |
| `auth`           | http        | HTTP基本身份验证凭据以强制执行隧道请求                       |
| `host_header`    | http        | 将HTTP Host标头重写为此值，或`保留`以保持不变                |
| `bind_tls`       | http        | bind an HTTPS or HTTP endpoint or both `true`, `false`, or `both` |
| `subdomain`      | httptls     | 要请求的子域名。 如果未指定，则使用隧道名称                  |
| `hostname`       | httptls     | 要求的主机名（需要保留名称和DNS CNAME）                      |
| `crt`            | httptls     | 此路径上的PEM TLS证书可在本地转发之前终止TLS流量             |
| `key`            | httptls     | PEM TLS此路径上的私钥，用于在本地转发之前加密认证TLS数据流   |
| `client_cas`     | httptls     | 此路径上的PEM TLS证书颁发机构将验证传入的TLS客户端连接证书。 |
| `remote_addr`    | tcptlsftp   | 保留的公网TCP地址及端口号                                    |
| `redirect_https` | http        | 为`true` 将Web站点http请求重定向至https协议同域名站点        |

### 4.5.示例配置文件 #

示例配置文件如下所示。 后续部分包含这些示例中显示的所有配置参数的完整文档。

创建一个名为website的Web站点及ssh隧道，域名地址随机变化(24小时-48小时)

```YAML
authtoken: 4nq9771bPxe8ctg7LKr_2ClH7Y15Zqe4bWLWF9p
tunnels:
  website:
    addr: 8080
    proto: http
  ssh:
    addr: 22
    proto: tcp
```



创建两个隧道，website和ssh，随机地址，地区不同

```YAML
authtoken: 4nq9771bPxe8ctg7LKr_2ClH7Y15Zqe4bWLWF9p
tunnels:
  website:
    addr: 8080
    proto: http
    region: cn_vip
  ssh:
    addr: 22
    proto: tcp
    region: cn
```

参数说明：
region: 地区参数，可以为us，cn, cn_vip，hk等，具体请参阅在线文档说明。
不同地区的带宽，网速不同，请根据您的所在地选择合适的地区。



创建二级子域名(dev.cpolar.cn)Web站点（在cn中国区)及ssh隧道

```YAML
authtoken: 4nq9771bPxe8ctg7LKr_2ClH7Y15Zqe4bWLWF9p
tunnels:
  website:
    addr: 8080
    proto: http
    subdomain: dev
    region: cn
  ssh:
    addr: 22
    proto: tcp
```



创建固定TCP端口的ssh隧道

```YAML
authtoken: 4nq9771bPxe8ctg7LKr_2ClH7Y15Zqe4bWLWF9p
tunnels:
  ssh:
    addr: 22
    proto: tcp
    remote_addr: 1.tcp.vip.cpolar.cn:20050
```

客户端ssh访问举例:(需要加-p 参数)

```shell
ssh root@1.tcp.cpolar.cn -p 20050
```

参数说明：
– -p ：参数为指定公网端口（默认ssh为22端口时不用指定，但在使用cpolar时，由于公网端口变为20050，所以需要加上）
– 1.tcp.cpolar.cn:20050地址: 是在`后台`–>`保留`–>`保留TCP地址`创建.
– root：为您服务主机的用户名

如果是树莓派，则上例可改为:

```shell
ssh pi@1.tcp.cpolar.cn -p 20050
```



为多个虚拟托管开发站点运行隧道

```YAML
authtoken: 4nq9771bPxe8ctg7LKr_2ClH7Y15Zqe4bWLWF9p
tunnels:
  app-foo:
    addr: 80
    proto: http
    host_header: app-foo.dev
  app-bar:
    addr: 80
    proto: http
    host_header: app-bar.dev
```



使用您自己的证书在http和https上配置自定义域名

```YAML
authtoken: 4nq9771bPxe8ctg7LKr_2ClH7Y15Zqe4bWLWF9p
tunnels:
  myapp-http:
    addr: 80
    proto: http
    hostname: example.com
    bind_tls: false
  mypp-https:
    addr: 443
    proto: http
    hostname: example.com
```



通过隧道暴露cpolar的Web监查界面

```YAML
authtoken: 4nq9771bPxe8ctg7LKr_2ClH7Y15Zqe4bWLWF9p
tunnels:
  myapp-http:
    addr: 4040
    proto: http
    subdomain: myapp-inspect
    auth: "user:secretpassword"
    inspect: false
```



包含所有选项的示例配置文件

```YAML
authtoken: 4nq9771bPxe8ctg7LKr_2ClH7Y15Zqe4bWLWF9p
region: us
console_ui: true
http_proxy: false
inspect_db_size: 50000000
log_level: info
log_format: json
log: /var/log/cpolar.log
metadata: '{"serial": "00012xa-33rUtz9", "comment": "For customer alan@example.com"}'
root_cas: trusted
socks5_proxy: "socks5://localhost:9150"
update: false
update_channel: stable
web_addr: localhost:4040
tunnels:
  website:
    addr: 8888
    auth: bob:bobpassword
    bind_tls: true
    host_header: "myapp.dev"
    inspect: false
    proto: http
    subdomain: myapp

  custdomain:
    addr: 9000
    proto: http
    hostname: myapp.example.com
    crt: /path/to/example.pem
    key: /path/to/example.key

  ssh-access:
    addr: 22
    proto: tcp
    remote_addr: 1.tcp.cpolar.io:20031
```

### 4.6.同时运行多个隧道 #

您可以将多个隧道名称传递给`cpolar start`，而cpolar将同时运行它们。

从配置文件中启动三个命名隧道

```shell
cpolar start admin ssh metrics
```



```shell
cpolar by @bestexpresser

Tunnel Status                 online
Version                       2.0/2.0
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://admin.cpolar.io -> 10.0.0.1:9001
Forwarding                    http://device-metrics.cpolar.io -> localhost:2015
Forwarding                    https://admin.cpolar.io -> 10.0.0.1:9001
Forwarding                    https://device-metrics.cpolar.io -> localhost:2015
Forwarding                    tcp://0.tcp.cpolar.io:48590 -> localhost:22
...
```

您还可以要求cpolar使用start-all子命令启动配置文件中定义的所有隧道。

启动配置文件中定义的所有隧道

```shell
cpolar start-all
```

### 5.全球基础设施 #

### 5.1.地理位置 #

cpolar在世界各地的数据中心运行着隧道服务器。 给定区域内的数据中心的位置可能在没有通知的情况下改变（例如，欧洲服务器可能从法兰克福转移到伦敦）。

> 需要注意：NAS套餐用户请选择使用Chinar NAS地区，才能够享受到NAS带宽（10M/20M/30M）。

| 服务器                   | 地区          | 英文       | 参数                  | 付费 |
| :----------------------- | :------------ | :--------- | :-------------------- | :--- |
| 大陆地区                 | –             | China      | cn                    | 免费 |
| –                        | China vip     | cn_vip     | 免费                  |      |
| –                        | China Top     | cn_top     | 免费                  |      |
| –                        | China VIP TOP | cn_vip_top | 免费                  |      |
| –                        | China NAS     | cn_nas     | 仅支持NAS套餐用户使用 |      |
| 海外地区                 | 香港          | Hong Kong  | hk                    | 免费 |
| 台湾                     | Taiwan        | tw         | 免费                  |      |
| 美国                     | United States | us         | 免费                  |      |
| 欧洲                     | Europe        | eu         | 免费                  |      |
| 澳大利亚                 | Australia     | au         | 免费                  |      |
| 亚洲/太平洋地区 – 新加坡 | Asia/Pacific  | ap         | 免费                  |      |
| 南非 – 约翰内斯堡        | South Africa  | zaf        | 免费                  |      |

### 5.2.使用方法 #

如果您没有明确选择某个区域，您的隧道将托管在默认区域China Top。 选择离您最近的区域就像指定设置-region命令行标志或在配置文件中设置region属性一样简单。 例如，要在中国地区启动隧道：

```shell
cpolar http -region=cn 8080
```

为特定区域（默认情况下为China Top）分配保留域和保留地址。 保留域或地址时，必须选择目标区域。 您不得绑定在分配给其他区域以外的其他区域中保留的域或地址。 尝试这样做会产生错误并阻止您的隧道会话初始化。

### 6.高级的隧道选项 #

### 6.1.泛域名 #

cpolar允许您将HTTP和TLS隧道绑定到通配符域名（泛域名）。 所有通配符域(泛域名)，甚至是cpolar.io子域的通配符必须首先保留在仪表板上的帐户中。 使用`-hostname`或`-subdomain`时，请指定前导*星号以绑定通配符域。



绑定隧道以接收** example.com`的所有子域的流量**

```shell
cpolar http -hostname=*.example.com 80
```

### 6.2.泛域名规则 #

使用通配符域会在cpolar.com服务的某些方面产生歧义。 以下规则用于解决这些情况，对于了解您是否使用通配符域非常重要。

出于示例的目的，假设您已为您的帐户保留地址`*.example.com`。

- 与嵌套子域的连接（例如`foo.bar.baz.example.com`）将路由到您的通配符隧道。
- 您可以在`example.com`的任何有效子域上绑定隧道，而无需创建其他保留域条目。
- 没有其他帐户可以保留`foo.example.com`或与其他帐户保留的通配符域匹配的任何其他子域。
- 连接将在线路由到最具体的匹配隧道。 如果您为`foo.example.com`和`* .example.com`运行隧道，对`foo.example.com`的请求将始终路由到`foo.example.com`

### 6.3.转发到不同计算机上的服务器（非本地服务） #

cpolar可以转发到未在本地计算机上运行的服务。 而不是指定端口号，只需指定网络地址和端口。



示例：转发到其他计算机上的Web服务器

```shell
cpolar http 192.168.1.1:8080
```

### 6.4.TLS 隧道 #

HTTPS隧道使用cpolar.com证书限制cpolar.com服务器上的所有TLS（SSL）通信。 对于生产级服务，您需要使用自己的TLS密钥和证书对隧道通信进行加密。 cpolar使用TLS隧道使此操作异常简单。

将TLS流量转发到端口443上的本地HTTPS服务器

```shell
cpolar tls -subdomain=encrypted 443
```



隧道运行后，尝试使用curl进行访问。

```shell
curl --insecure https://encrypted.cpolar.io
```

### 6.4.1.加密的TLS连接 #

您尝试公开的服务可能无法限制TLS连接。 cpolar客户端可以为您执行此操作，以便您可以端对端加密通信，而不必担心本地服务是否支持TLS。 同时指定-crt和-key命令行选项，以指定TLS证书和密钥的文件系统路径，然后cpolar客户端将为您限制TLS连接。

```shell
cpolar tls -region=cn -hostname secure.example.com -key=/path/to/tls.key -crt=/path/to/tls.crt 80
```

### 6.4.2.通过TLS隧道运行非HTTP服务 #

cpolar TLS隧道不对正在传输的基础协议进行任何假设。 本文档中的所有示例都使用HTTPS，因为它是最常见的用例，但是您可以在TLS隧道上运行任何TLS封装的协议（例如imaps，smtps，sip等），而无需进行任何更改。

### 6.4.3.没有证书警告的TLS隧道 #

注意前面的curl命令示例中的`--insecure`选项吗？您需要指定，因为您的本地HTTPS服务器没有终止任何cpolar.io子域的通信所需的TLS密钥和证书。如果您尝试在网络浏览器中加载该页面，则会注意到它告诉您该页面可能不安全，因为证书不匹配。

如果要使证书匹配并免受中间人攻击，则需要两件事。首先，您需要为拥有的域名购买SSL（TLS）证书，并将本地Web服务器配置为使用该证书及其私钥来终止TLS连接。如何执行此操作仅针对您的Web服务器和SSL证书提供者，不在本文档的讨论范围之内。为了举例说明，我们假设为您颁发了域`secure.example.com`的SSL证书。

获得密钥和证书并正确安装它们之后，现在该在您自己的自定义域名上运行TLS隧道了。设置的说明与HTTP隧道部分中描述的相同：“自定义域上的隧道”。您注册的自定义域应与SSL证书（`secure.example.com`）中的域相同。设置自定义域后，使用-hostname参数在您自己的域上启动TLS隧道。

通过您自己的自定义域转发TLS流量

```shell
cpolar tls -region=us -hostname=secure.example.com 443
```

### 6.4.4.客户端兼容性 #

TLS隧道通过检查传入TLS连接上的服务器名称信息（SNI）扩展中存在的数据来工作。 并非所有启动TLS连接的客户端都支持设置SNI扩展数据。 这些客户端无法与cpolar的TLS隧道一起正常使用。 幸运的是，几乎所有现代浏览器都使用SNI。 某些现代软件库却没有。 以下客户端列表不支持SNI，并且不适用于TLS隧道：

- **Microsoft Internet Explorer 6.0**
- **Microsoft Internet Explorer 7 & 8 on Windows XP or earlier**
- **Native browser on Android 2.X**
- **Java <=1.6**
- **Python 2.X, 3.0, 3.1 [if required modules are not installed]**(https://stackoverflow.com/questions/18578439/using-requests-with-tls-doesnt-give-sni-support/18579484#18579484 “if required modules are not installed”)
- **可以在Wikipedia上的[Server Name Indiciation](https://en.wikipedia.org/wiki/Server_Name_Indication#No_support)页面上找到更完整的列表。**

### 7.IP白名单隧道访问 #

您可以将帐户中的隧道端点列入白名单。 白名单由cpolar.com服务器强制执行。 它全局应用于所有隧道端点。 检查与任何隧道端点的任何传入连接，以确保连接的源IP地址与白名单中的至少一个条目匹配。 如果连接与白名单不匹配，则会立即终止，并且永远不会转发给cpolar客户端。

作为一种特殊情况，如果您的白名单为空，则允许所有连接。

### 7.1.管理白名单 #

您可以在cpolar仪表板的验证选项卡上管理IP白名单。 在“IP白名单”部分下输入新的IP地址，然后单击**添加白名单条目**。 对IP白名单的更改最多可能需要30秒才能生效。

### 7.2.IP范围 #

有时，您可能希望将整个IP范围列入白名单。 您可以使用[[1] CIDR_notation] [CIDR_notation]指定IP地址块，而不是仅输入单个IP地址。 例如，要允许从10.1.2.0到10.1.2.255的所有IP地址，您需要将10.1.2.0/24添加到白名单。
[CIDR_notation]：https：//en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_notation“CIDR_notation”

### 8.高级功能 #

### 8.1.重写主机Host头域 #

当转发到本地端口时，cpolar不会修改隧道传输的 HTTP 请求，它们会在收到时逐字节复制到您的服务器。一些应用服务器如 WAMP、MAMP 和 pow 使用 `Host` 标头来确定要显示的开发站点。因此，cpolar可以使用修改后的 Host 标头重写您的请求。使用 `-host-header` 开关重写传入的 HTTP 请求。

如果值指定了`rewrite`，`Host` 头将被重写以匹配转发地址的主机名部分。任何其他值都会导致 `Host` 标头被重写为该值。

将主机头重写为`site.dev`

```shell
cpolar http -host-header=rewrite site.dev:80
```



将 `Host`头重写为“example.com”

```shell
cpolar http -host-header=example.com 80
```

### 8.2.http重定向至https #

增加http重定向功能，使http地址跳转至https同域名地址，客户端命令行增加参数`-redirect-https=true`，如果需要在配置文件中配置加参数`redirect_https: true`
例如：

```shell
cpolar http -redirect-https=true -region=cn 8080
```

### 8.3.本地站点使用HTTPS协议通讯 #

cpolar假定要转发到的服务器正在侦听未加密的HTTP流量，但是如果您的服务器正在侦听加密的HTTPS流量怎么办？ 您可以使用`https://`协议指定URL，以请求cpolar对本地服务器使用HTTPS。

通过指定`https://`协议转发到https服务器

```shell
cpolar http https://localhost:8443
```

### 8.4.用密码保护你的隧道 #

任何可以猜测您的隧道URL的人都可以访问您的本地Web服务器，除非您使用密码保护它。 您可以使用`-httpauth`开关确保隧道安全。 这将使用您指定为参数的用户名和密码对所有请求强制执行HTTP Basic Auth。



示例：用密码保护您的隧道

```bash
cpolar http -httpauth="username:password" 8080
```

### 8.5.Websockets #

Websocket端点通过cpolar的http隧道工作，无需任何更改。 但是，目前还没有支持监听初始化连接时，返回临时状态码101时的交换消息。

### 8.6.虚拟主机 (pow, MAMP, WAMP, and others) #

流行的Web服务器（如MAMP和WAMP）依赖于一种通常被称为“虚拟主机”的技术，这意味着它们会查询HTTP请求的“主机”标头，以确定它们应该为多个站点服务。 要公开这样的网站，可以要求cpolar重写所有隧道请求的主机头，以匹配您的Web服务器所期望的。 您可以使用`-host-header`选项（请参阅：重写主机标头）来选择要定位的虚拟主机。 例如，要路由到本地站点`myapp.dev`，您将运行：

```shell
cpolar http -host-header=myapp.dev 80
```

### 8.7.出站代理 #

cpolar通过HTTP或SOCKS5代理正常工作。 cpolar尊重标准的unix环境变量http_proxy。 您还可以在cpolar配置文件中显式设置代理配置：

- [Set the configuration variable http_proxy](https://cpolar.com/docs#config_http_proxy)
- [Set the configuration varible socks5_proxy](https://cpolar.com/docs#config_socks5_proxy)

### 9.常见问题 #

### 9.1.无法访问cpolar web ui #

当在浏览器上访问本地9200端口时，页面提示`无法访问此网站，127.0.0.1 拒绝了我们的连接请求`或`Network Error`，如下图所示：

![20221212162412](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717761.png)

#### 解决方案：请重启cpolar服务

#### 1. Windows系统：

点击开始菜单栏搜索`服务`，并点击打开服务，找到`Cpolar Service`，选中并右键点击`重新启动`，确认cpolar service状态为`正在运行`，再重新在浏览器访问本地9200端口，访问cpolar web UI管理界面。

![20221212163915](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717056.png)

#### 2. Linux系统：

请先确认是否有安装cpolar服务，如`未安装cpolar服务`，请操作安装cpolar服务：

- 向系统添加cpolar服务

```shell
sudo systemctl enable cpolar
```

- 启动cpolar服务

```shell
sudo systemctl start cpolar
```

- 查看服务状态

```shell
sudo systemctl status cpolar
```

如正常显示为`active(running)`，则为启动成功状态

![20221212165139](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717335.png)

如`已安装cpolar服务`，则执行命令重启cpolar服务，再重新尝试访问cpolar web ui管理界面

```shell
sudo systemctl restart cpolar
```

### 9.2.登录失败，用户不一致 #

当在浏览器登录cpolar web ui管理界面时，提示`登录失败，用户不一致`，如下图所示：

![20221223154740](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717391.png)

如已经在一台设备上的cpolar web ui管理界面登录了A账号，但是想要更换登录B账号进行使用，需修改配置文件：

> 需要注意，成功更换账户登录后，原先A账户中所配置的固定地址隧道

#### 1. windows系统

1. 找到cpolar配置文件:c:\Users\用户名.cpolar\cpolar.yml。
2. 选中cpolar.yml文件，右键点击“打开方式”——“记事本”，选择使用记事本打开配置文件。
3. 在配置文件，删除authtoken一行和email一行。
4. 保存cpolar.yml配置文件。
5. 在控制面板—管理工具—服务—cpolar service，重启服务。
6. 重新登录cpolar web UI管理界面即可。

![20221213143022](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717504.png)

#### 2. linux系统

1. 编辑cpolar配置文件

```shell
nano /usr/local/etc/cpolar/cpolar.yml
```

1. 在配置文件，删除authtoken一行和email一行，保存cpolar.yml配置文件。
2. 重启cpolar服务

```shell
sudo systemctl restart cpolar
```

### 9.3.修改cpolar web UI默认端口号 #

如您本地已有服务使用了9200端口，那么您可以将cpolar web UI管理端口修改为其他端口，如`127.0.0.1:9300`

**1. 找到cpolar配置文件**

- windows系统: c:\Users\用户名.cpolar\cpolar.yml，右键点击“打开方式”——“记事本”，选择使用记事本打开
- linux系统：执行命令`nano /usr/local/etc/cpolar/cpolar.yml`

**2. 在配置文件中，增加一行参数：**

```shell
client_dashboard_addr: 127.0.0.1:9300
```

> 注：添加的一行，与authtoken是一个级别的参数。

**3. 保存cpolar配置文件**
**4. 重启cpolar服务**

- windows系统：在控制面板–管理工具—服务—cpolar service，重启服务。
- linux系统：执行命令`sudo systemctl restart cpolar`

**5. 浏览器访问本地9300端口http://127.0.0.1:9300/**

![20221213144331](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717702.png)

### 9.4.远程访问cpolar web UI管理界面 #

我们可以创建一条http协议隧道，指向本地9200端口，在外访问所生成的公网地址，来实现远程访问本地的cpolar web UI。

登录本地cpolar web ui后，点击左侧仪表盘的`隧道管理`——`创建隧道`：

- 隧道名称：cpolar web ui（可自定义）
- 协议：选择http
- 本地地址：9200端口（cpolar web UI默认端口号）
- 域名类型：随机域名
- 地区：China top

点击`创建`

![20221221143326](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717889.png)

隧道创建成功后，在左侧仪表盘的状态——在线隧道列表中，找到刚刚创建成功的cpolar web ui隧道，有生成了相应的公网地址，复制下来到浏览器访问测试一下，访问成功：

![20221223165849](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717012.png)

### 9.5.远程桌面 #

### 9.5.1.测试3389端口是否开放 #

在被控端电脑上，点击左下角开始菜单栏，打开设置，点`应用`设置，打开`系统`

![20221228115950](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717097.png)

在左侧页面下找到“远程桌面”选项，点击“打开”

![20221228115959](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717219.png)

点击左下角开始菜单栏，打开设置，点`应用`设置

![20221228114602](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717297.png)

在“应用”的搜索栏中输入`telnet`，查找Windows内的相关功能

![20221228114613](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717448.png)

在新开启的功能窗口中，找到`telnet`，并将其勾选开启，接着点击`确定`保存设置

![20221228114619](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717530.png)

等待Windows开启功能即可

![20221228114626](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717576.png)

在完成这些设置后，我们就可以打开powershell，进行一个简单的小测试。通常Windows的远程桌面端口为3389，为确保该端口已经正常开放，我们可以打开Windows的powershell软件（该软件可以简单理解为加强版的cmd命令行，通常为系统自带）

![20221228114632](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717780.png)

输入命令测试本地电脑的3389端口是否已经开放

```shell
telnet 127.0.0.1 3389
```

![20221228114639](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717895.png)

命令行窗口不显示任何信息说明远程桌面3389端口处于开启状态。

![20221228114649](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717943.png)

如果3389端口处于关闭状态，命令行窗口会显示连接失败。

### 9.5.2.win10家庭版系统远程桌面 #

需要注意的是，win10家庭版系统不支持微软远程桌面（只有远程协助，而没有是否允许远程桌面的选项），需要为win10专业版、企业版系统。如您的系统为家庭版，您可以将系统升级至专业版再实现远程桌面。

### 9.5.3.设置用户名密码 #

远程桌面需要使用用户名密码，注意您的账号需要是超级管理员权限，或者有远程桌面远程的账号，才能登录。

如果被控端电脑原来没有密码，请设置密码后，再远程桌面。安全第一。

需要注意的是，如果用户名密码为空则会出现`用户账户限制（例如，时间限制）会阻止你登录。请与系统管理员或技术支持联系以获取帮助`的错误提示。请设置密码后，再进行远程桌面。

### 9.5.4.提示无法找到计算机 #

当提示`远程桌面无法找到计算机“cpolar公网地址”。可能是“cpolar公网地址”不属于指定的网络。请验证该计算机名和您尝试连接到的域`时，请先确认所输入的公网地址是否正确。如正确无误，请确认远程桌面隧道是否在线：

- 登录cpolar官网，点击左侧的状态，查看是否有远程桌面隧道所生成的公网地址
- 被控端电脑登录cpolar web UI管理界面，查看远程桌面隧道是否在线，状态是否为`active`

如确认远程桌面隧道不在线，请启动远程桌面隧道，并再次尝试使用所生成的公网地址进行远程桌面。

需要注意，公网刚远程桌面连接的前提是局域网内能够正常远程，否则无法实现远程。如局域网内能够正常远程，仍旧无法使用cpolar所生成的公网地址进行远程桌面，请联系官网在线客服人员或者官方qq群，请求技术支持。

### 9.5.5.远程桌面提示“您的凭证不工作” #

远程桌面电脑时，输入用户密码后如出现”你的凭据不工作”，请先确认凭据是否正确。如确认所输入的凭据无误，但仍提示“你的凭据不工作”，则有可能是由Windows安全设置引起的问题。

#### 方法一、编辑本地组策略

1. 键盘同时按下“Win+R”打开运行窗口

![20230117163134](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717985.png)

1. 输入`gpedit.msc`点击确定

![20230117163700](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071718004.png)

1. 点击计算机配置——Windows设置——安全设置——本地策略——安全选项

![20230117170413](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071718310.png)

1. 找到“网络访问：本地账户的共享和安全模型”并双击打开它，然后将其设置为“经典 – 对本地用户进行身份验证，不改变其本来身份”并单击“确定”。

![20230117171400](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071718315.png)

保存后，再次连接,应该就可以解决了。

#### 方法二、编辑本地安全策略

有可能是因为Windows安全策略阻止非管理员用户登录导致的。因此，我们可以通过修改本地安全策略，允许非管理员用户使用远程桌面连接。

1. 键盘同时按下“Win+R”打开运行窗口

![20230117163134](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071717985.png)

1. 输入`secpol.msc`点击确定

![20230117165510](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071718338.png)

1. 在本地安全策略中展开“本地策略”，然后双击“用户权限分配”选项。

![20230117165555](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071718400.png)

1. 在用户权限详细策略列表中找到“允许通过远程桌面服务登录”，然后选中并双击。

![20230117170054](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071718421.png)

1. 在弹出窗口中点击“添加用户或组”。

![20230117170125](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071718679.png)

1. 输入远程桌面用户名，然后单击“确定”即可。

![20230117170154](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071718700.png)

#### 方法三、添加Windows凭据

点击左下角开始菜单栏，搜索`控制面板`并打开，点击右上角的查看方式，选择小图标显示

![20230117172221](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071718731.png)

点击“凭据管理器”

![20230117172419](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071718782.png)

点击“添加Windows凭据”，输入要登录的服务器的IP地址、用户名、密码，点确定退出。

![20230117172703](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202305071718810.png)

### 9.6.配置自定义域名报错 #

**Q：配置自定义域名提示报错**

提示`Server failed to allocate tunnel rpc error. code = Unknown desc = sal no rows in result set`

**A：保留与配置的域名不一致**

在cpolar官网后台——预留——保留的自定义域名，与您在web-ui中配置的自定义域名，内容不一致，就会报这个错误。请校对下保留与配置的域名是否一致。

### 9.7.预留的TCP地址出现异常 #

**Q：预留的TCP地址出现异常**

提示：`dial tcp 61.160.213.238:4443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond`。

**A：两种情况：**

1. 有可能向238请求的IP，频繁访问238，会被ban拒绝；
2. 有可能宽带的线路，访问238短期内不通。换一个IP就好了。

### 9.8.创建HTTP协议隧道 #

Q:创建一个http协议隧道时，会生成http和https两个公网地址，算是两个隧道吗？

A:不算,还是一个隧道。生成两个公网地址表示支持http协议，也支持https协议（无需配置SSL证书，cpolar.cn已备案）。

### 9.9.隧道在线状态 #

**Q：Web ui里面的在线状态和官网的在线状态隧道是同步的吗？**

**A：以cpolar官网后台在线隧道列表为准。**

正常会与官网同步，实际请登录cpolar官网,点击右侧得到状态,以页面所显示的在线隧道列表为准。如cpolar web ui界面的在线隧道与官网后台在线隧道不一致，请以官网后台为准。可重启本地cpolar服务更新。

> 注：cpolar官网后台所显示的在线隧道列表包含了该账户下所有设备的在线隧道。

### 9.10.SSH TCP链接不通的常见的问题 #

**1. ssh命令，并没有使用-p参数**

ssh命令，默认是连接22端口的，由于我们的本地22端口到了公网被映射到了某个公网端口，如20050，所以，ssh命令需要加-p参数，后面加公网隧道端口号。

因此，请先排查是否有加-p参数。

**2. 确认ssh命令中的用户名是否输入正确**

```shell
ssh -p XXXXX 用户名@1.tcp.vip.cpolar.cn（X为cpolar生成的端口号，用户名替换为主机用户名）
```

需要注意，命令中的用户名请输入被远程主机的用户名，而不是cpolar。

**3. 查看我们的TCP隧道是否在线**

可以访问cpolar官网并登录到后台，点击左侧的状态，查看下ssh隧道是有正常在线，如果没有正常在线， 则没有生成相应的公网地址。

如果隧道没有在线，请重启下服务，再观察隧道是否能够正常启动为在线状态。

**4. 确认TCP在线隧道是端口是否是通的。 **

如果3中的TCP隧道在线，我们可以使用telnet命令，确认一下我们的TCP隧道是否是通的。因为隧道是在公网上，我们就可以直接查验。

命令行执行：

```shell
telnet 1.tcp.cpolar.cn 20050
```

(如果没有telnet，百度 windows 安装telnet客户端）

**telnet公网端口后的结果**

- 如果输出结果为，打不开20050端口号，说明在线隧道的端口不通。
- 结果结束为一个黑屏窗口，什么都没有，说明端口是通的。
- 如果出现黑屏或^，端口通的，但马上又断开了连接，则表示可能公网端口是通的，但您的服务没开启。

如自行排查问题后仍无法连接，请向官网在线客服或者qq客服反馈报错。

### 9.11.WordPress #

要使用Wordpress安装正常工作，通常需要做两件事：

1.您必须确保Wordpress发布相对URL。 您可以通过安装以下插件之一来完成此操作

- https://github.com/optimizamx/odt-relative-urls
- http://wordpress.org/plugins/relative-url/
- http://wordpress.org/plugins/root-relative-urls/

2.您必须确保Wordpress了解它是为了通过隧道主机名提供服务。 您可以通过修改`wp-config`来配置Wordpress以包含以下行：

```PHP
define('WP_SITEURL', 'http://' . _SERVER['HTTP_HOST']);
define('WP_HOME', 'http://' ._SERVER['HTTP_HOST']);
```

### 9.12.我的隧道存储了哪些信息? #

cpolar不会记录或存储通过隧道连接传输的任何数据。 cpolar会记录有关用于调试目的的连接的一些信息，以及隧道名称和连接持续时间等指标。 要获得完整的端到端安全性，请使用`TLS隧道`。

### 9.13.我怎么发音cpolar? #

拼读：si-polar

- see：看见， polar：极点
- 望穿-极点

### 10.故障排除 #

### 10.1.CORS与HTTP基本身份验证 #

是的，但你不能使用cpolar的`-httpauth`选项。 cpolar的http隧道允许您指定基本身份验证凭据以保护您的隧道。 但是，cpolar对**所有请求强制执行此策略**，包括CORS规范要求的预检“OPTIONS”请求。 在这种情况下，您的应用程序必须实现自己的基本身份验证 有关更多详细信息，请参阅[this github issue](https://github.com/inconshreveable/ngrok/issues/196)