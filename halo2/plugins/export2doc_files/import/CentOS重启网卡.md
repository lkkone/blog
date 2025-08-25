CentOS重启网卡

- 重载配置文件

```bash
nmcli c reload
```

- 重连网卡

```bash
nmcli d reapply
```

- 或者重新连接

```bash
nmcli connect 
```

- 或

```bash
nmcli c up 
```

