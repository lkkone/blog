#### Rewrite模块

---

##### if	语句判断

Syntax：if	（判断条件）{动作}

Context：server	location

| 正则 | 含义               |
| ---- | ------------------ |
| ~    | 模糊匹配           |
| ~*   | 不区分大小写的匹配 |
| !~   | 不匹配             |
| =    | 精准匹配           |



---

##### set	设置变量

Syntax：set  $variable  value

Context：server	location	if

---

##### return	返回数据指令

Syntax：return  code；return  code  [text]；return  URL

Context：server	location	if

相关命令：

```shell
curl -L -I -A "firefox" -HHost:hope202091.com http://123.56.243.98/			[[利用curl命令]] -A 伪造火狐浏览器
```

---

##### rewrite重写flag

|           | 关键字  | 正则     | 替换内容    | fiag标记 |
| --------- | ------- | -------- | ----------- | -------- |
| Syntax：  | rewrite | regex    | replacement | [fiag]   |
| Context： | server  | location | if          |          |

flag标记分类

| 语法      | 含义                                                 |
| --------- | ---------------------------------------------------- |
| last      | 本条规则匹配完成后，继续向下匹配新的location URL规则 |
| break     | 本条规则匹配完成后终止，不再匹配后面的任何规则       |
| redirect  | 返回302临时重定向，地址栏会显示跳转后的地址          |
| permanent | 返回301永久重定向，地址栏会显示跳转后的地址          |

---

#### Rewrire生产案例实践

---

需求1：根据用户浏览器请求头中携带的语言调度的不同的页面

```shell
server {
	listen 80;
	server_name url.xx.net;
	root /code;
	
	if ($http_accept_language ~* "zh-CN|zh") {
		set $language /zh;
	}
	if ($http_accept_language ~* "en") {
		set $languag /en;
	}
	rewrite ^/$	/$language;
	
	location / {
	index index.html;
	}
}
```

---

需求2：用户通过手机设备访问  url.xx.net  ，跳转至  url.xx.net/m

```shell
server {
	listen 80;
	server_name url.xx.net;
	root /code;
	
	if ($http_user_agent ~* "android|iphone|ipad") {
		rewrite ^/$ /m;
	}
}
```

---

需求3：用户通过手机设备访问  rul.xx.net  跳转至  m.xx.net

```shell
server {
	listen 80;
	server_name url.xx.net;
	root /code;
	
	location / {
		if ($http_user_agent ~* "android|iphone|ipad") {
			rewrite ^/$ http://m.xx.net	redirect;
			[[return]] 302 http://m.xx.net;
		}
	}
}
```

---

需求4：用户通过http协议请求，能自动跳转至https协议

```shell
server {
	listen 80;
	server_name url.xx.net;
	rewrite ^(.*)$ https://$server_name$1 redirect;
	[[return]] 302 https://$server_name$request_uri;
}
server {
	listen 80;
	server_name url.xx.net;
	root /code;
	ssl on;
}
```

需求5：网站在维护过程中，希望用户访问所有网站重定向至一个维护页面

```shell
server {
	listen 80;
	server_name url.xx.net;
	root /code;
	
	rewrite ^(.*)$ /wh.html break;
	
	location / {
		index index.html;
	}
}
```

需求6：当服务器遇到  403  404  502  等错误时，自动跳转至临时维护的静态页

```shell
server {
	listen 80;
	server_name url.xx.net;
	root /code;
	
	location / {
		index index.html;
	}
	error_page 403 404 502 = @tempdown;
	
	location @tempdown {
		rewrite ^(.*)$ /wh.html break;
	}
}
```

需求7：公司网站在停机维护时，指定的IP能够正常访问，其他的IP跳转至维护页面

```shell
server {
	listen 80;
	server_name url.xx.net;
	root /code;
	
	set $ip 0;
	if ($remote_addr = 10.0.0.1|10.0.0.2) {
		set $ip 1;
	}
	
	if ($ip = 0) {
		rewrite ^(.*)$ /wh.html break;
	}
	
	location / {
		index index,html;
	}
}
```

需求8：公司网站后台  /admin  ，只允许公司的出口公网IP可以访问，其他的IP访问全部返回500，或直接跳转至首页

```shell
location /admin {
	set $ip 0;
	
	if ($remote_addr = "61.149.186.152|139.226.172.254") {
		set $ip 1;
	}
	
	if ($ip = 0) {
		return 500;
		[[rewrite]] /(.*) https://url.xx.net redirect;
	}
}
```

需求9：用户请求  url.xx.net/bbb  跳转至  http://demo:27610/url/bbb

```shell
server {
	listen 80;
	server_name url.xx.net;
	root /code;
	
		location / {
			index index.html;
			
			if (http_host ~* (.*).(.*).(.*)) {
				set $domain $1;
			}
			
			rewrite ^(.*)$ http://demo:27610/$domain$request_uri redirect;
		}
}
```

需求10：现有两台服务器，想要实现http://console.xx.net/index.php?r=sur/index/sid/613192/lang/zh-Hans若访问资源为/index.php?r=survey...则跳转至http://sur.oldxu.net.cn/index.php?=sur/index/sid/613192/lang/zh-Hans

```shell
server {
	listen 80;
	server_name console.xx.net;
	root /code;
	index index.html;
	
	location / {
		if ($args ~ r=survey) {
			rewrite ^/(.*) http://sur.xx.net$sequest_url? redirect;
		}
	}
}
```

需求11：将用户请求http://url.xx.net/?id=2，替换为http://url.xx.net/id/2.html

```shell
server {
	listen 80;
	server_name url.xx.net;
	root /code;
	
	location / {
		index index.html;
		
		if ($args ~ "id") {
			set $ok 1;
		}
		
		if ($args ~ (.*)=(.*)) {
			set $id $1;
			set $number $2；
		}
		
		if ($ok = 1) {
			rewrite ^(.*)$ http://$http_host/$id/$number.html? last;
		}
	}
}
```

需求12：禁止搜索引擎爬取站点

```shell
server {
	listen 80;
	server_name url.xx.net;
	root /code;
	
	location / {
		index index.html;
		
		if ($http_user_agent ~ "YisouSpider|baiduSpider|YoudaoBot") {
			return 403;
		}
	}
}
```

防止站点资源被盗用

```shell
server {
	listen 80;
	server_name img.xx.net;
	root /code;
	
	location / {
		index index.html;
	}
	
	location ~ .*\.(jpg|jpeg|gif|png)$ {
		valid_referers none blocked *.xx.net;
		if ($invallid_referer) {
			return 403;
			rewrite ^(.*)$ /error.jpg break;
		}
	}
}
```

