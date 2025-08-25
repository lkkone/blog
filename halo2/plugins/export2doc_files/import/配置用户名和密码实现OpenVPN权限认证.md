# [ 配置用户名和密码实现OpenVPN权限认证](https://www.linuxprobe.com/openvpn-privilege-authentication.html)

安装部署：

[CentOS](https://www.linuxprobe.com/) 6.5

软件FQ官网下载

**同步系统时间**

```
yum install chrony -y

service chronyd start && chronyc sources && chkconfig chronyd on

或

yum install ntpdate -y

crontab添加

*/1 * * * * /usr/sbin/ntpdate 0.rhel.pool.ntp.org > /dev/null 2>&1

service crond restart
```

**安装依赖包**

```
yum install epel-release -y && echo "sslverify=false">>/etc/yum.conf

yum install openssl openssl-devel lzo lzo-devel pam pam-devel pam_mysql automake pkgconfig gcc gcc-c++
```

**安装OpenVPN**

```
cd /usr/local/src/

tar -zxvf openvpn-2.4.4.tar.gz

cd openvpn-2.4.4

./configure --prefix=/opt/openvpn make && make install

cp -a sample/sample-config-files/server.conf /opt/openvpn/         

[[最好放在/opt/openvpn/下]]

cp -a distro/rpm/openvpn.init.d.rhel /etc/init.d/openvpn           

[[创建启动脚本]]

ln -s /opt/openvpn/sbin/openvpn /usr/sbin/openvpn                  

[[启动脚本中会用到，也可以不执行此命令，直接在启动脚本中修改]]

vi /etc/init.d/openvpn                        

[[在85行，修改为：work]]=/opt/openvpn

cd /opt/openvpn/ && mv server.conf server.conf.bak
vi server.conf    [[修改配置文件]]; ';'为注释

port 1195				[[使用1195端口]]
proto tcp				[[使用tcp传输模式]]
dev tun				[[使用tun虚拟网卡设备（还有一种是Tap）]]
ca keys/ca.crt			[[指定server端证书路径]]
cert keys/server.crt		[[指定server端证书路径]]
key keys/server.key		[[Thisfile]] should be kept secret
dh keys/dh2048.pem
tls-auth keys/ta.key 0
cipher AES-256-CBC
server 10.8.0.0 255.255.255.0		[[openvpn使用的网络]]
push "route 10.8.0.0 255.255.0.0"	[[添加openvpn路由]]
[[push]] "route 0.0.0.0 0.0.0.0"                          
ifconfig-pool-persist ipp.txt			[[客户端连入后使用的IP地址池]]
push "dhcp-option DNS 61.134.1.4"		[[客户端连入后使用的DNS]]
push "dhcp-option DNS 223.5.5.5"
keepalive 10 120					[[保持VPN会话]]
comp-lzo							[[开启Lzo数据压缩]]
user nobody
group nobody
auth-user-pass-verify /opt/openvpn/checkpsw.sh via-env
script-security 3
client-cert-not-required
[[不请求客户的CA证书，使用User/Pass验证，如果同时启用证书和密码认证，注释掉该行]]
username-as-common-name
persist-key
persist-tun
verb 3
link-mtu 1500					[[设置MTU连接数值]]
status logs/openvpn-status.log
log         logs/openvpn.log
log-append  logs/openvpn.log
 

mkdir logs                          [[创建日志目录]]

mkdir keys                          [[创建key目录]]
[root@vpn ~]# openvpn --help | grep -A 5 script-security

--script-security level mode : mode='execve' (default) or 'system', level=

0 -- strictly no calling of external programs

1 -- (default) only call built-ins such as ifconfig

2 -- allow calling of built-ins and scripts

3 -- allow password to be passed to scripts via env

--shaper n : Restrict output to peer to n bytes per second.
```

**安装easy-rsa，用来生成证书和密钥**

```
cd /usr/local/src/

wget http://download.freenas.org/distfiles/easy-rsa-2.2.0_master.tar.gz

[[http]]://download.freenas.org/distfiles/

tar -zxvf easy-rsa-2.2.0_master.tar.gz

cp -a easy-rsa-2.2.0_master/easy-rsa /opt/openvpn/

cd /opt/openvpn/easy-rsa/2.0/

mv vars vars.bak
vi vars                             [[修改配置文件]]

export EASY_RSA="`pwd`"
export OPENSSL="openssl"
export PKCS11TOOL="pkcs11-tool"
export GREP="grep"
export KEY_CONFIG=`$EASY_RSA/whichopensslcnf $EASY_RSA`
export KEY_DIR="$EASY_RSA/keys"
echo NOTE: If you run ./clean-all, I will be doing a rm -rf on $KEY_DIR
export PKCS11_MODULE_PATH="dummy"
export PKCS11_PIN="dummy"
export KEY_SIZE=2048                   [[修改为2048]]
export CA_EXPIRE=3650
export KEY_EXPIRE=3650
export KEY_COUNTRY="CN"                 [[以下根据自己情况修改]]
export KEY_PROVINCE="ShaanXi"
export KEY_CITY="XA"
export KEY_ORG="yjz"
export KEY_EMAIL="xx@yjz.cn"
export KEY_CN=yjz
export KEY_NAME=yjz
export KEY_OU=yjz
 
ln -s openssl-1.0.0.cnf openssl.cnf
source  vars                        

[[全局变量]]

##生成证书，以下命令全部一直回车

./clean-all                         

[[清空所有证书（keys目录下）]]

./build-ca                          

[[生成服务器ca证书]]

./build-key-server server

[[生成服务端证书]]

./build-dh                          

[[生成DH验证文件（dh2048]].pem）

openvpn --genkey --secret ta.key    

[[降低DDoS风险]]

./build-key client

[[生成客户端证书（建议以使用者命名）]]
```

**设置外网访问**

```
vim /etc/sysctl.conf               [[将net]].ipv4.ip_forward = 0 改为 1

sysctl -p

配置nat表将vpn网段IP转发到server内网：很重要

iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE              

[[注意接口（eth0）是内网的接口，其它选项不要修改]]

iptables -A INPUT -p TCP --dport 1195 -j ACCEPT              

[[开启防火墙1195端口]]

service iptables restart                  [[POSTROUTING需要保存并重启服务才能生效]]

chkconfig iptables on
```

**启动服务**

```
[[拷贝证书到/opt/openvpn/keys目录下]]

cd /opt/openvpn/easy-rsa/2.0/keys/ cp -a ca.crt  server.crt dh2048.pem server.key /opt/openvpn/keys    

cd .. && cp ta.key  /opt/openvpn/keys            /etc/init.d/openvpn start chkconfig openvpn on
```

**配置脚本+密码文件控制方式**

下载脚本，根据具体配置修改红色部分

```
http://openvpn.se/files/other/checkpsw.sh

#!/bin/sh

###########################################################

# checkpsw.sh (C) 2004 Mathias Sundman 

#

# This script will authenticate OpenVPN users against

# a plain text file. The passfile should simply contain

# one row per user with the username first followed by

# one or more space(s) or tab(s) and then the password.



PASSFILE="/opt/openvpn/psw-file"

LOG_FILE="/opt/openvpn/logs/openvpn-password.log"

TIME_STAMP=`date "+%Y-%m-%d %T"`



###########################################################



if [ ! -r "${PASSFILE}" ]; then

echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >> ${LOG_FILE}

exit 1

fi



CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}' ${PASSFILE}`



if [ "${CORRECT_PASSWORD}" = "" ]; then

echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}

exit 1

fi



if [ "${password}" = "${CORRECT_PASSWORD}" ]; then

echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}

exit 0

fi



echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}

exit 1
 

touch /opt/openvpn/logs/openvpn-password.log

chown nobody:nobody /opt/openvpn/logs/openvpn-password.log

 

密码存放方式

在psw-file里按”用户名[空格或者tab]密码“这种规则方式存放

touch /opt/openvpn/logs/psw-file

chown nobody:nobody /opt/openvpn/psw-file

cat /opt/openvpn/psw-file

test test

ipad ipad
```

**Windows客户端配置**

```
下载：openvpn-install-2.4.4-I601.exe 点击安装，一直next，默认目录安装即可 一般会安装到 C:/Program Files/OpenVPN/ 目录下

创建client.ovpn文件： client dev tun proto tcp-client remote x.x.x.x 1195 [[vpn服务端ip，这里为内网对应的公网IP，路由器映射至内网主机]]

remote-random

resolv-retry infinite

nobind

persist-key persist-tun

ca ca.crt

auth-user-pass

auth-nocache

remote-cert-tls server

tls-auth ta.key 1

cipher AES-256-CBC [[保持服务端和客户端一致]]

comp-lzo

status openvpn-status.log
将client.ovpn文件放到C:/Program Files/OpenVPN/config目录下
```

从VPN服务端下载ca.crt，ta.key证书 将ca.crt，ta.key证书放到C：/Program Files/OpenVPN/config目录下

点击桌面openvpn图标，输入相应的用户名密码即可

> 原文来自：https://my.oschina.net/HeAlvin/blog/2051348
>
> 本文地址：https://www.linuxprobe.com/openvpn-privilege-authentication.html

本文原创地址：https://www.linuxprobe.com/openvpn-privilege-authentication.html