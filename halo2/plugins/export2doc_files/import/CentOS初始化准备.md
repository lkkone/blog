### CentOS初始化准备

---

##### 关闭防火墙

systemctl disable --now firewalld

##### 关闭selinux

```bash
vim  /etc/selinux/config;reboot
SELINUX=disabled
```

- 文件带点为未关闭selinux

- getenforce检查关闭

##### 安装源

- Dase
- AppStream
- EPEL
- extras
- PowerTools

##### 安装基本软件

- vim
- tree
- lrzsz
- wget
- curl
- tree
- zip
- unzip
- xz

##### 时间同步

- 检查 chronyc sources -v

