#### Nginx常用模块

##### 目录索引

charset utf8，gbk	防止中文乱码

autoindex	on|off	目录索引开闭

autoindex_exact_size	on|off	人性化显示大小

autoindex_localtime	on|off	显示时间

```shell
相关命令	rsync同步	createrepo 创建仓库目录
```



---

##### 访问控制

allow  all|Ip地址	允许

deny  all|Ip地址	拒绝

基于站点	基于rul

---

##### 基础认证

auth_basic	"描述内容"

auth_basic_user_file	密码文件路径

```shell
相关命令	yum  provides  htpasswd搜索命令来自包	yum  install  httpd-tools安装密码软件包
```

创建密码文件	htpasswd -c （创建新密码文件） -b （允许明文密码） 路径	用户名	密码

---

##### 限速限流

###### 请求限制

http层

定义请求限制	limit_req_zone $binary_remote_addr（定义来源ip地址变量） zone=req_one:10m（缓冲空间） rate=1r/s（每秒请求量）;

server层

请求限制的调用	limit_req zone=req_one  burst=5  nodelay

###### 连接限制

http层

定义连接限制	limit_conn_zone $binary_remote_addr（定义来源IP） zone=conn_od（名字可自行定义，http层每个名称只能定义一次）:10m（共享内存区大小）

server层

连接限制的调用	limit_conn conn_od 2（并发连接数）	

限制下载速度	limit_rate_after  100m（达到此速度开始限速）	limit_rate  100k （限速后的下载速度）

应用：超过连接数和后会返回客户端503错误，此时可拦截错误，跳转页面信息，如提醒充值内容

注意*	也可在限制条件后自行定义返回的状态码

limit_req_status 412 ;（写在连接数后，则达到连接数限制条件后返回412）

error_page  503  @temp;	拦截503错误

location  @temp  {			（内部跳转：加@号未内部跳转，外部无法直接请求）

​				default_type text/html;（定义页面）

​				return  200 "跳转后页面内容";（页面输出内容）

}

```shell
相关命令	netstat -nt 查看ip连接状态
```



---

状态监控

location层（常用）

location	/nginx_status {         （名称自定义）

​				stub_status;

​				[[相关策略]]	允许|拒绝什么地址访问

​				allow	172.0.0.1；

​				deny	all；

}

得到数据解释：

Actve connections：2								当前活跃连接数

server accepts handled requsts                 当前总接收的TCP连接数	总共处理连接数数	总的请求数				

​               106      106       406                      因为是长连接，所以一次一次连接有多次请求，可在主配置文件中修改	keepalive_timeout	65;（默认数）可修改为0，则为一次连接一次请求

Reading：0 Writing：1 Waiting：1			  服务端接收到服务端请求的头部信息	服务端响应请求回馈的数据	大量回馈数据需建立管道连接发送，管道数

相关命令	

```shell
echo "当前Nginx的活跃连接数是：$(curl  -s  -HHost：域名  IP地址/ngx_status  |  awk  -F  ':'  'NF==1  {print  $NF}'   |sed  's# ##g')"
```

取出当前连接数并消除数字前空格（-s 屏蔽curl的输出信息）并结合echo输出

```shell
echo "当前Nginx失败的TCP连接数是：$(curl  -s  -HHost：域名  IP地址/ngx_status  |  awk  -F  ':'  'NF==3  {print  $1-$2}')"
```

取出accepts（总的TCP连接数）减去handled（处理的TCP连接数）并结合echo输出，也就是取出了失败连接数

*注意	systemctl  restart  nginx 数据归零	systemctl  reload  nginx 数据不归零

---

###### 资源压缩

location层

location  ~*  .*\.(jpg|gif|png)$ {                                               此为压缩图片，压缩文件时更改为（txt|pdf）

​			gzip  on;																	开闭gzip压缩

​			gzip_http_version  1.1;											 压缩支持协议

​			gzip_vary  on														    开闭响应头部添加Vary：Accept-Encoding启用压缩的报文

​			gzip_comp_level  2;												 压缩等级（越高压缩的越小,消耗cpu也越大）

​			gzip_min_lenth  10k;                                                大于规定值启用压缩

​			gzip_types  image/jpeg  imzge/gif  image/png;        压缩规定格式                 （压缩文件时更改为text/plain application/pdf等）

​			gzip_vary  on;

}

*注意	压缩文件格式定义存于  /etc/nginx/mime.types文件中

---

##### Location

| 通配符 | 匹配规则               | 优先级 |
| ------ | ---------------------- | ------ |
| =      | 精准匹配               | 1      |
| ^~     | 以某个字符开头         | 2      |
| ~      | 区分大小写的正则匹配   | 3      |
| ~*     | 不区分大小写的正则匹配 | 4      |
| /      | 通用匹配               | 5      |

在location找不到页面文件时根据错误代码定向页面或使用@重定向接收后定义到内部页面

=======================================================================================================

指定页面

error_page  403  404  400 /新页面														 接收错误代码并转到指定页码

=======================================================================================================

@重定向

error_page  404  @error_404;			 													接受代码后转交@error_404

location  @error_404  {																			定义@error_404

​				dafault_type  text/html			  												引用内部页面

​				return  302  http://$server_name;		 									重定向至首页

}

=======================================================================================================

---

##### 日志模块

定义日志格式

###### 访问日志

access_log  on|off	访问日志的开闭

log_format	main|test				变量集成

常用变量

| 变量                  | 含义                                         |
| --------------------- | -------------------------------------------- |
| $remote_addr          | 记录客户端IP地址                             |
| $remote_user          | 记录客户端用户名                             |
| $time_local           | 记录通用的本地时间                           |
| $time_ios8601         | 记录ISO8601标准格式下的本地时间              |
| $request              | 记录请求的方法以及请求的http协议             |
| $status               | 记录请求状态码（用于定位错误信息）           |
| $body_bytes_sent      | 发送给客户端的资源字节数（不包括响应头的大小 |
| $bytes_sent           | 发送给客户端的总字节数                       |
| $msec                 | 日志写入时间，单位为秒，精度为毫秒           |
| $http_referer         | 记录客户端是从哪个页面连接访问过来的         |
| $http_user_agent      | 记录客户端浏览器相关信息                     |
| $http_x_forwarded_for | 记录客户端IP地址                             |
| $request_length       | 请求的长度（包括请求行，请求头和请求正文）   |
| $request_time         | 请求花费的时间，单位为秒                     |

日志格式引用

access_log  /var/log/nginx/access.log  main|test;				规定日志路径并引用

=======================================================================================================

###### 错误日志

error_log /dev/null	关闭错误日志（存放路径打到空）

常见错误日志级别	<高	debug | info | notice | warn | error | crit | alert | emerg	低>

级别越高，记录的信息越多	不定义默认级别为warn

应用层级	main	http	server	location

=======================================================================================================

###### 日志过滤

 常用于location层过滤ico图标文件请求日志，或者规定缓存客户端内存

loaction  /favicon,ico  {

access_log  off;	                            关闭日志

return  200；									重定向至200状态码

}

====================================================

loaction  /favicon,ico  {

access_log  off;	                            关闭日志



expires  3650d；							  缓存10年

}

=======================================================================================================

*注意	日志引用可定义到所有层，http层定义为整个主机，每个server中定义为单独站点的日志（非常有必要），location中没有必要定义 

=======================================================================================================

Realilp模块获取真实IP

| Ryntax                            | Overview                                                     |
| --------------------------------- | ------------------------------------------------------------ |
| set_real_ip_from [代理IP]         | 真实服务器上一级代理的IP地址或IP地址段，可以写多行           |
| real_ip_header    X-Forwarded-For | 从哪个Header头检索出需要的IP地址                             |
| real_ip_recursive    on           | 递归排除set_real_ip_from里面出现的IP，其余没有出现的认为是用户真实IP |

