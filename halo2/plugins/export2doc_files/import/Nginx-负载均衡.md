#### Nginx-负载均衡

---

##### 反向代理

  proxy_pass	http://IP地址：端口

###### 相关配置参数

=======================================================================================================

| Syntax                             | Default                           | Context                |
| ---------------------------------- | --------------------------------- | ---------------------- |
| proxy_set_set_header  field  value | proxy_set_header Host $proxy_host | http  server  location |

注释：**Host**位置决定向后端传递的IP地址值，**X-Real-IP  $remote_addr**  将  **$remote_addr**  的值放进  **X-Real-IP**  中，**remote_add**r  的值为客户端IP，**proxy_set_header  X-Forwarded-For  $proxy_addr_x_forwarded_for**

=======================================================================================================

| Syntax                       | Default                | Context                |
| ---------------------------- | ---------------------- | ---------------------- |
| proxy_http_version 1.0 \|1.1 | proxy_http_version 1.0 | http  server  location |

注释：代理向后端请求时使用的HTTP协议版本，默认1.0版本·

=======================================================================================================

| syntax                      | Default                   | Context                |
| --------------------------- | ------------------------- | ---------------------- |
| proxy_connect_timeout  time | proxy_connect_timeout 60s | http  server  location |

Nginx代理与后端服务器“连接超时”时间（代理连接超时）

=======================================================================================================

| syntax                   | Default                | Context                |
| ------------------------ | ---------------------- | ---------------------- |
| proxy_read_timeout  time | proxy_read_timeout 60s | http  server  location |

Nginx代理等待后端服务器“响应（Header）超时”时间

=======================================================================================================

| syntax                   | Default                | Context                |
| ------------------------ | ---------------------- | ---------------------- |
| proxy_send_timeout  time | proxy_send_timeout 60s | http  server  location |

后端服务器“数据（Data）回传给Nginx代理的超时”时间

=======================================================================================================

| syntax                      | Default                    | Context                |
| --------------------------- | -------------------------- | ---------------------- |
| proxy_buffering  on \| off  | proxy_buffering  on        | http  server  location |
| proxy_buffer_size size      | proxy_buffer_size 4k \| 8k | http  server  location |
| proxy_buffers  number  size | proxy_buffers  8 4k \|8k   | http  server  location |

**peoxy_buffering**  开启或关闭缓冲区	启用缓冲时，Nginx代理服务器将尽快的接受响应Header以及响应报文，并将其保存到  **proxy_buffer_size(Headers)**  和  **proxy_buffres(data)**  设置的缓冲区中

**proxy_buffer_size**  用于控制代理服务器读取后端第一部分响应Header的缓冲区大小

**proxy_buffers**  是代理服务器为单个连接设置响应缓冲区“数量”和“大小”

=======================================================================================================

| Syntax                                       | Overview                        |
| -------------------------------------------- | ------------------------------- |
| proxy_pass http://xx;                        | 重新定向到定义的资源池          |
| proxy_next_upstream error timeout http_500等 | 出现错误、超时和500等错误就定向 |
| proxy_next_upstream_tries  2                 | 尝试请求的次数                  |
| proxy_next_upstream_timeout  3s              | 尝试的超时时间                  |

注解：当资源池中的某台服务器出现错误但没down机，就将请求重新定向到资源池里的其他集群		异常容错机制

###### 代理参数总结



将代理网站常用优化配置写入一个新文件，使用时通过  **include**  调用即可

```shell
[root@proxy01 ~]# vim /etc/nginx/proxy_params
[[协议与头部]]
proxy_http_version 1.1;
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

[[请求连接]] 响应 数据回传时间
proxy_connect_timeout 30;
proxy_send_timeout 60;
proxy_read_timeout 60;

[[缓冲区]]
proxy_buffering on;
proxy_buffer_size 64k;
proxy_buffers 4 64k;
```

**include**  调用方式

```shell
location / {
	proxy_pass http://172.0.0.1:8080;
	include proxy_params;
}
```

---



##### Nginx七层负载均衡

**syntax**  

proxy_upstream_module

通过upsteam关键字定义虚拟资源池

proxy_modules

通过proxy_pass关键字调用虚拟资源池

```shell
upstream backend {
	server backend1.xx.com;
	server backend2.xx.com;
	server backend3.xx.com;
}

server {
	location / {
		proxy_pass http://backend;
	}
}
```

---

**调度算法相关参数**

| name                   | syntax                          |
| ---------------------- | ------------------------------- |
| 轮询                   | server 172.16.1.7:80;           |
| 加权轮询        weight | server 172.16.1.7:80  weight=5; |
| ip_hash调度算法        | ip_hash;                        |
| 一致性hash调度算法     | hash  $remote_addr  consistent; |
| url_hash调度算法       | hash  $request_uri  consistent; |
| least_conn调度算法     | least_conn;                     |

---

**后端状态相关参数**

| State                 | Overview                                     |
| --------------------- | -------------------------------------------- |
| down                  | 当前的server暂时不参与负载均衡               |
| backup                | 预留的备份服务器                             |
| max_fails = [size]    | 允许请求失败的次数  默认 1次                 |
| fail_timeout = [size] | 经过max_fails失败后，服务器暂停时间  默认10s |
| max_conns = [size]    | 限制最大的连接数                             |
| keepalive  [size]     | 提升吞吐量，限制长连接最大空闲连接数         |

---



##### Nginx会话共享-Redis

会话保持分类

| name          | Overview                                                     |
| ------------- | ------------------------------------------------------------ |
| 粘性session   | 指Nginx每次都将同一用户的所有请求转发至同一台服务器上，及Nginx的IP_hash |
| session复制   | 每次session发生变化，就广播给集群中的服务器，使所有的服务器上的session相同  tomcat应用 |
| session共享   | 换chunsession至内存数据库中，使用Redis，memcashed实现        |
| session持久化 | 将session存储至数据库中，像操作数据一样操作session  wordpress应用 |
| cookie植入    |                                                              |

---

1.下载安装启动Redis服务器

```shell
[root@redis ~]# yum install -y redis
[root@redis ~]# systemctl emable redis
[root@redis ~]# systemctl start redis
```

2.修改redis配置并重载

```shell
[root@redis ~]# vim /etc/redis.conf
/搜索bind	添加本地IP地址
[root@redis ~]# systemctl reload redis
```

3.修改所有服务器php配置文件

```shell
[root@redis ~]# vim /etc/php.ini
/搜索session修改
```

*注意：php修改配置redis模板从redis模块文件  **/etc/php.d/redis.ini**  中来			语法为下      另外php.ini中注释符为；

*注意：默认的session存储方式为fire，存储位置为  **/etc/lib/php/session**

| Syntax                                                      | Overview                                            |
| ----------------------------------------------------------- | --------------------------------------------------- |
| session.save_handler = redis                                | 定义session缓存方式                                 |
| session.save_path = tcp://xxIP?weight=xx&timeout=xx&auth=xx | 定义redis缓存地址、权重、超时时间、密码，可定义多台 |

4.修改配置文件  **/etc/php-fpm.d/www.cof**

```shell
[root@web01 ~]# vim /etc/php-fpm.d/www.conf 
/搜索session
注释掉定义session存储类型和路径的两行
;php_calue[session.save_handler] = files
;php_value[session.save_path] = /var/lib/php/session
```

5.重启php-fpm

```shell
[root@web01 ~]# systemctl restart php-fpm.service 
```

---

相关命令

**Redis**

| Syntax    | Overview     |
| --------- | ------------ |
| redis-cli | 登录redis    |
| keys  *   | 查看缓存数据 |

**mysql**

| Syntax            | Overview   |
| ----------------- | ---------- |
| mysql -u -p       | 登录数据库 |
| show  databases； | 查看数据库 |
| use    xx；       | 进入       |
| show  tables；    | 查看目录   |
| desc              |            |

