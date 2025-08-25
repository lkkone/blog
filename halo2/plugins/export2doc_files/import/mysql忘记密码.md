### mysql忘记密码

---

#### 修改密码

使用root登录，修改mysql数据库的user表
使用password()函数进行密码加密
注意修改完成后需要刷新权限

```bash
use mysql;

update user set password=password('新密码') where user='用户名';

例：

update user set password=password('123') where user='root';

刷新权限：flush privileges;
```

#### 忘记 root 账户密码

1、配置mysql登录时不需要密码，修改配置文件

Centos中：配置文件位置为/data/server/mysql/my.cnf
Windows中：配置文件位置为C:\Program Files (x86)\MySQL\MySQL Server 5.1\my.ini
修改，找到mysqld，在它的下一行，添加skip-grant-tables

```sql
[mysqld]
skip-grant-tables
```

2、重启mysql，免密码登录，修改mysql数据库的user表

```sql
use mysql;

update user set password=password('新密码') where user='用户名';

例：
update user set password=password('123') where user='root';

刷新权限：flush privileges;
```

3、还原配置文件，把刚才添加的skip-grant-tables删除，重启