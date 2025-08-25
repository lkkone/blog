### mysql数据库设置允许root远程连接



1.查看mysql库user表的root用户是否允许外部远程连接

```sql
SELECT User, Password, Host FROM user;
```

2.将root的Host字段更改为任意

`mysql5.7及以下`

```sql
[[新增一个root连接模式]]
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

---
[[解析]]
其中"*.*"代表所有资源所有权限， “'root'@%”其中root代表账户名，%代表所有的访问地址，也可以使用一个唯一的地址进行替换，只有一个地址能够访问。如果是某个网段的可以使用地址与%结合的方式，如10.0.42.%。IDENTIFIED BY '123456'，这个123456是指访问密码。WITH GRANT OPTION允许级联授权
```

---

`mysql8.0及以上`

```sql
[[创建用户]]
CREATE USER 'root'@'%' IDENTIFIED BY '123456';

[[授权]]
grant all privileges on *.* to 'root'@'%' ;
```

---

重新查看root的Host字段

```sql
mysql> SELECT User, Password, Host FROM user;
```

运程连接

```bash
mysql -h 10.0.0.100 -u root -p
```



---

### 补充：

如遇报错

```bash
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

报错说明：当前root用户密码不符合远程远程连接要求

解决方法：

1.修改为复杂密码

```sql
[[查看当前mysql密码规则]]
show variables like 'validate_password%';

---
[[修改密码的mysql用户密码的三种方式]]
1.使用set password命令
mysql> set password for {usernamee} @localhost = password{newpwd}
[[username]] 为要修改密码的用户名，newpwd 为要修改的新密码

2.使用mysqladmini修改
mysqladmin -u{username} -p{oldpwd} password {newpwd}
[[oldpwd为旧密码，另外修改密码的命令中]] -uroot 和 -proot 是整体，不要写成 -u root -p root，-u 和 root 间可以加空格，但是会有警告出现，所以就不要加空格了

3.update直接编辑user表
[[切换到mysql库]]
mysql> use mysql
[[更改mysql库user表的authentication_string字段设置新密码]]
mysql> update mysql.user set authentication_string=password({newpwd}) where user='{username}' and Host = 'localhost';
[[刷新权限]]
flush privileges;
```

2.修改策略

```sql
[[查看当前密码规则]]
show variables like 'validate_password%';

[[密码的长度是由validate_password_length决定的]],但是可以通过以下命令修改
set global validate_password_length=4;

[[validate_password_policy决定密码的验证策略]],默认等级为MEDIUM(中等),可通过以下命令修改为LOW(低)
set global validate_password_policy=0;

[[刷新权限]]
flush privileges;

[[validate_password_policy值]]
0 or LOW     只验证长度
1 or MEDIUM  验证长度、数字、大小写、特殊字符
2 or STRONG  验证长度、数字、大小写、特殊字符、字典文件
```

---

### 相关博文

**相较于上面的方法，下面这篇是直接修改原有root的连接权限**



---

**MysQL设置root密码，并且允许远程连接**

MySQL设置root密码，并且允许远程连接，连接数据库的时候，遇到host is not allowed to connect MySQL，这个说明mysql不允许连接，所以需要修改user表；允许root远程链接比较危险，一般情况下不建议用root，建议新建立一个子账号，如whmblog

一、修改root用户密码
首先，你需要能够登陆到远程主机，直接shell连接就可以了，然后执行：

```bash
mysql -u root [[没有密码的时候]] 

或者

mysql -u root -p [[有密码的时候，一般很可能是root]]
```

然后，登陆上mysql数据，注意，需要是root用户哦。 

修改密码的时候，我们就直接修改mysql数据库中的user表记录就可以了。 

二、首先选择mysql表

```sql
use mysql;
```

三、然后可以查询一下目前的用户

```sql
select user,host,password from user;
```

如果有多个，你可以就看你需要修改哪个了，其中的host字段是允许连接的地址，%表示通配。然后使用sql直接更新就可以了，比如：

```sql
update user set password=password('这里是密码') where user='root';
```

到这里我们就把密码修改了

四、开启远程访问
其实，就是把root的host字段设置成%，表示所有ip都可以连接。

```sql
update user set host='%' where user='root';
```

设置就可以了，但是，还要刷新一下

```sql
flush privileges;

[[别忘记了，不然不会生效的]]
```

---

