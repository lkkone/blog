## chronyd时间同步服务器：

> server: 10.35.10.39
> client: 10.35.10.40、10.35.10.41

### 安装chrony、设置开机启动

**注：若机器可联网，做到这步即可。**

```
yum install -y chrony
systemctl start chronyd
systemctl enable chronyd
```

### server配置：

​	vim /etc/chrony.conf  添加以下两项：

	local stratum 10	：即使server端无法从互联网同步时间，也同步本机时间至client
	allow all			：允许所有主机从server端同步时间

### client配置：
​	vim /etc/chrony.conf  注释原server配置，改为prod1机器IP地址

	server 10.35.10.39 iburst

client端重启chronyd

```
systemctl restart chronyd
```

### 查看服务状态：

查看server端在线情况：
	chronyc activity
	
查看时间同步情况：
	chronyc tracking

