

容器启动：

```bash
docker run -d --name ss -p 10086:8388 -e PASSWORD=YourPassword -e METHOD=aes-256-gcm shadowsocks/shadowsocks-libev
```

Shadowsocks使用BBR加速

安装 BBR

```bash
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
```

执行权限

```bash
chmod +x bbr.sh
```

启动BBR安装

```bash
./bbr.sh
```

重启服务器

重新启动之后，lsmod | grep bbr 如果看到 tcp_bbr 就说明 BBR 已经启动了

另外一种开启bbr加速

ubuntu

```bash
echo net.core.default_qdisc=fq >> /etc/sysctl.conf

echo net.ipv4.tcp_congestion_control=bbr >> /etc/sysctl.conf
```

执行

```bash
sysctl -p
sysctl net.ipv4.tcp_available_congestion_control
```

检测是否如下所示：

```bash
sysctl net.ipv4.tcp_available_congestion_control

net.ipv4.tcp_available_congestion_control = bbr cubic reno
```

执行以下命令检测 BBR 是否开启：

```bash
lsmod | grep bbr
```

出现类似以下的情况就是成功开启BBR

```bash
tcp_bbr 24576 3
```