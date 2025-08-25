# 搭建OpenVPN

## OpenVPN

如果我们的服务都是部署在内网，外网不能访问，但是我们在外边访问不了内网怎么办？公司的服务有很多为了让安全，是只能在内网访问的，这时候我们可以使用开源的OpenVPN来实现。

OpenVPN 是一个功能齐全的 SSL VPN，它使用行业标准 SSL/TLS 协议实现 OSI 第 2 层或第 3 层安全网络扩展，支持基于证书、智能卡和/或用户名/密码凭据的灵活客户端身份验证方法，并允许用户或使用应用于 VPN 虚拟接口的防火墙规则的特定于组的访问控制策略。

## 安装依赖

```bash
yum install -y epel-release 
yum install -y openssl lzo pam openssl-devel lzo-devel pam-devel
复制代码
```

- 安装epel-release 源，因为官方的rpm仓库，为了安全版本往往滞后，甚至有些包就没有，所以使用epel-release源下载。
- openssl包 程序包
- lzo包 用来进行数据压缩

## 安装openvpn和easy-ras

```bash
yum install -y easy-rsa
yum install -y openvpn
复制代码
```

- easy-ras包 提供了一些证书生成脚本、模板配置文件等创建相关目录
- openvpn就是我们今天要安装的包了

## 配置证书密钥

```bash
cp -rf /usr/share/easy-rsa/3.0.8 /etc/openvpn/server/easy-rsa
cd /etc/openvpn/server/easy-rsa



./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa build-server-full server nopass
./easyrsa build-client-full client1 nopass
./easyrsa build-client-full client2 nopass
./easyrsa gen-dh
openvpn --genkey --secret ta.key
复制代码
```

- init-pki：初始化，创建pki文件夹，用来存放即将生成的证书
- build-ca： 会提示输入CA的名称，如下图

![openvpn1.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/239a6cb7ceb34a10823112e01b60978f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image) 根据提示输入，也可以Enter采用默认的。会生成ca.crt等根证书和密钥文件。

- build-server-full client1: 生成服务端/客户端证书，`nopass` 参数设定证书为无密码, 否则还需要设定证书密码
- gen-dh：创建Diffie-Hellman，迪菲·赫尔曼密钥交换 是一种安全协议。它可以让双方在完全没有对方任何预先信息的条件下通过不安全信道创建起一个密钥。这个密钥可以在后续的通讯中作为对称密钥来加密通讯内容。
- --genkey --secret ta.key:可选，生成ta.key用于防御DoS、UDP淹没等恶意攻击。

## 编写身份验证脚本

```bash
vim /etc/openvpn/server/user/checkpsw.sh
复制代码
```

内容如下

```sh
#!/bin/sh
###########################################################
# checkpsw.sh (C) 2004 Mathias Sundman <mathias@openvpn.se>
#
# This script will authenticate OpenVPN users against
# a plain text file. The passfile should simply contain
# one row per user with the username first followed by
# one or more space(s) or tab(s) and then the password.
PASSFILE="/etc/openvpn/server/user/psw-file"
LOG_FILE="/var/log/openvpn/password.log"
TIME_STAMP=`date "+%Y-%m-%d %T"`
###########################################################
if [ ! -r "${PASSFILE}" ]; then
  echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >>  ${LOG_FILE}
  exit 1
fi
CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}' ${PASSFILE}`
if [ "${CORRECT_PASSWORD}" = "" ]; then
  echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=
\"${password}\"." >> ${LOG_FILE}
  exit 1
fi
if [ "${password}" = "${CORRECT_PASSWORD}" ]; then
  echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}
  exit 0
fi
echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=
\"${password}\"." >> ${LOG_FILE}
exit 1
复制代码
```

修改文件的权限.我这里写的755.估计降低权限，只给用户组读执行的权限也可。

```bash
chmod 755 /etc/openvpn/server/user/checkpsw.sh
chown openvpn:openvpn /etc/openvpn/server/user/checkpsw.sh
复制代码
```

## 创建认证的用户文件

tee命令用于读取标准输入的数据，并将其内容输出成文件。

```bash
tee /etc/openvpn/server/user/psw-file << EOF
复制代码
```

输入用户名密码,以空格隔开，务必写在一行，否则认为两个用户

```bash
test test
复制代码
```

结束输入

```
EOF
复制代码
```

修改权限

```bash
chmod 600 /etc/openvpn/server/user/psw-file
chown openvpn:openvpn /etc/openvpn/server/user/psw-file
复制代码
```

## 配置server端

```bash
mkdir -p /var/log/openvpn/
mkdir -p /etc/openvpn/server/user
chown openvpn:openvpn /var/log/openvpn
复制代码
```

- 第一个文件夹用来存放opencv的日志文件
- 第二个文件夹，存放账号密码的用户文件以及检查登录认证的脚本
- 第三个命令，将日志文件夹的权限赋给openvpn用户组下的openvpn用户，因为启动之后的openvpn是以openvpn用户的权限启动的。

```bash
vim /etc/openvpn/server/server.conf
复制代码
```

新文件，在文件里写下

```sh
#################################################
# This file is for the server side              #
# of a many-clients <-> one-server              #
# OpenVPN configuration.                        #
#                                               #
# Comments are preceded with '#' or ';'         #
#################################################
port 1194
proto tcp-server
## Enable the management interface
# management-client-auth
# management localhost 7505 /etc/openvpn/user/management-file
dev tun     # TUN/TAP virtual network device
user openvpn
group openvpn
ca /etc/openvpn/server/easy-rsa/pki/ca.crt
cert /etc/openvpn/server/easy-rsa/pki/issued/server.crt
key /etc/openvpn/server/easy-rsa/pki/private/server.key
dh /etc/openvpn/server/easy-rsa/pki/dh.pem
tls-auth /etc/openvpn/server/easy-rsa/ta.key 0
## Using System user auth.
# plugin /usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so login
## Using Script Plugins
auth-user-pass-verify /etc/openvpn/server/user/checkpsw.sh via-env
script-security 3
# client-cert-not-required  # Deprecated option
verify-client-cert
username-as-common-name
## Connecting clients to be able to reach each other over the VPN.
client-to-client
## Allow multiple clients with the same common name to concurrently connect.
duplicate-cn
# client-config-dir /etc/openvpn/server/ccd
# ifconfig-pool-persist ipp.txt
server 10.8.0.0 255.255.255.0
push "dhcp-option DNS 114.114.114.114"
push "dhcp-option DNS 1.1.1.1"
push "route 192.168.1.0 255.255.255.0"
# comp-lzo - DEPRECATED This option will be removed in a future OpenVPN release. Use the newer --compress instead.
compress lzo
# cipher AES-256-CBC
ncp-ciphers "AES-256-GCM:AES-128-GCM"
## In UDP client mode or point-to-point mode, send server/peer an exit notification if tunnel is restarted or OpenVPN process is exited.
# explicit-exit-notify 1
keepalive 10 120
persist-key
persist-tun
verb 3
log /var/log/openvpn/server.log
log-append /var/log/openvpn/server.log
status /var/log/openvpn/status.log
复制代码
```

- port:端口号，可修改
- server 10.8.0.0，vpn的虚拟网关
- push "route 192.168.1.0 255.255.255.0"指定内网的网关，这样我们就可以通过openvpn访问该网关下的内网。
- ca 指定刚才生成的根证书和密钥文件。
- cert 指定服务端的证书
- key 指定服务端的密钥
- dh指定密匙交换文件，即刚才生成的迪菲·赫尔曼密钥交换文件
- log指定log文件
- status指定状态日志文件
- auth-user-pass-verify指定身份验证脚本

## 创建软连接

```bash
cd /etc/openvpn/server/
ln -sf server.conf .service.conf
复制代码
```

## 配置防火墙

```bash
firewall-cmd --permanent --add-masquerade
firewall-cmd --permanent --add-service=openvpn
firewall-cmd --zone=public --add-port=1194/tcp --permanent
firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
firewall-cmd --reload
复制代码
```

- add-masquerade: 允许防火墙伪装IP,要想开启端口转发，需要先开启此功能
- add-service：永久开放此服务
- add-port：开放端口号
- direct： 设置NET规则,将ipv4转发到openvpn通道
- reload：刷新配置，使上述配置生效

## 启动服务端

```bash
systemctl start openvpn-server@.service.service
复制代码
```

无报错信息，则启动成功，服务器端配置完成。

## 安装客户端

下载地址：[官网](https://link.juejin.cn?target=https%3A%2F%2Fopenvpn.net%2Fvpn-client%2F)下载,并安装，可在第一步指定文件的安装目录。比如我安装在`D:\openvpn`

## 下载客户端所需要的证书文件

均在服务器上

- ca.crt :在`/etc/openvpn/server/easy-rsa/pki/`文件夹下
- ta.key:在`/etc/openvpn/server/easy-rsa/`文件夹下
- client1.key:在`/etc/openvpn/server/easy-rsa/pki/private/`文件夹下
- client1.crt:在`/etc/openvpn/server/easy-rsa/pki/issued/`文件夹下

下载上述四个文件到客户端的`D:\openvpn\config`下，并创建文件`client.ovpn`

输入以下内容

```vbnet
client
proto tcp-client
dev tun
auth-user-pass
remote Openvpn的公网地址 1194
ca ca.crt
cert client1.crt
key client1.key
tls-auth ta.key 1
remote-cert-tls server
auth-nocache
persist-tun
persist-key
compress lzo
verb 4
mute 10
复制代码
```

修改remote openvpn的公网地址，即可。

## 启动客户端

以管理员身份运行，连接时输入我们上述创建的用户名test和密码test，连接成功后，在本地访问远程服务器上的内网ip，成功访问。

![openvpn2.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2b59a6b792d4f329e42b3c6ab358e59~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)



作者：WangScaler
链接：https://juejin.cn/post/6976159670608068639
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。