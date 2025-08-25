## 时间同步服务-Chrony

---

### 服务端

#### 安装

```bash
yum install chrony -y
```

#### 配置

```bash
主配置文件：/etc/chrony.conf
允许内网网段同步此服务器
allow 172.16.1.0/24
```

#### 启动

```bash
服务端程序：/usr/bin/chronyd
启动服务
systemctl start chronyd
修改配置后需重载
systemctl reload chronyd
```

#### 优化

```bash
将外部服务器修改为国内时间同步服务器
server ntp.aliyun.com iburst
开启断网继续同步
local stratum 10
```

---

### 客户端

#### ntpdate

安装ntpdate命令

```bash
yum install ntpdate -y
```

单条命令实现

```bash
ntpdate 172.16，1.62
```

定时任务实现

```bash
crontab -e
*/5 * * * * ntpdate 172.16.1.62 $>/dev/null 
```

#### chrony

##### 安装

```bash
yum install chrony -y
```

##### 配置

```bash
vim /etc/chrony.ocnf
指向服务端
server 172.16.1.62 iburst
```

##### 启动

```bash
systemctl restart chronyd
```

##### 查看

```bash
chronyc sources
```

