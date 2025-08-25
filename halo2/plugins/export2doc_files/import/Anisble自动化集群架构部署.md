## 														Anisble自动化集群架构部署

---

### 服务器地址规划

| 主机名称      | 外网地址-eth0-NAT           | 内网地址-eth1-LAN                 | 角色          |
| ------------- | --------------------------- | --------------------------------- | ------------- |
| DNS-master    | 10.0.0.91 gateway：10.0.0.2 | 172.16.1.91                       | dnsservers    |
| DNS-slave     | 10.0.0.92 gateway：10.0.0.2 | 172.16.1.92                       | dnsservers    |
| Route         | 10.0.0.200（Forward）       | 172.16.1.200                      | routes        |
| LVS-master    |                             | 172.16.1.3 gateway：172.16.1.200  | lbservers     |
| LVS-backup    |                             | 172.16.1.4 gateway：172.16.1.200  | lbservers     |
| HAProxy-node1 |                             | 172.16.1.5 gateway：172.16.1.200  | proxyservers  |
| HAProxy-node2 |                             | 172.16.1.6 gateway：172.16.1.200  | proxyservers  |
| Web-node1     |                             | 172.16.1.7 gateway：172.16.1.200  | webservers    |
| Web-node1     |                             | 172.16.1.8 gateway：172.16.1.200  | webservers    |
| Web-node1     |                             | 172.16.1.9 gateway：172.16.1.200  | webservers    |
| Mysql-master  |                             | 172.16.1.51 gateway：172.16.1.200 | dbservers     |
| Mysql-slave   |                             | 172.16.1.52 gateway：172.16.1.200 | dbservers     |
| Redis-server  |                             | 172.16.1.41 gateway：172.16.1.200 | redisservers  |
| NFS-server    |                             | 172.16.1.32 gateway：172.16.1.200 | nfsservers    |
| Backup        |                             | 172.16.1.31 gateway：172.16.1.200 | backupservers |
| Jmpserver     |                             | 172.16.1.61 gateway：172.16.1.200 | jmsservers    |
| Openvpn       | 10.0.0.60                   | 172.16.1.60                       | vpnservers    |

---

### 基础环境准备

#### 准备配置文件

- 进入ansible/roles目录

```bash
cd /etc/ansible/roles/
```

- 拷贝配置文件至当前目录

```bash
cp /etc/ansible/ansible.cfg ./
```

- 拷贝主机清单文件至当前目录

```bash
cp /etc/ansible/hosts ./
```

#### 修改配置文件

- 修改主机清单文件存放位置&关闭密钥检测&修改和添加facts缓存存储方式

```bash
vim ansible.cfg

inventory      = ./hosts		[[定义主机清单位置]]
host_key_checking = False		[[关闭密钥检测]]
gathering = smart
fact_caching = redis				[[缓存模式]]
fact_caching_timeout = 600			[[缓存时间]]
fact_caching_connection = localhost:6379	[[使用本机的redis]]
```

- 定义主机清单

```bash
vim hosts

[dnsservers]
172.16.1.91
172.16.1.92

[routes]
172.16.1.200

[lbservers]
172.16.1.3
172.16.1.4

[proxyservers]
172.16.1.5
172.16.1.6

[webservers]
172.16.1.7
172.16.1.8
172.16.1.9

[dbservers]
172.16.1.51

[redisservers]
172.16.1.41

[nfsservers]
172.16.1.32

[backupservers]
172.16.1.31

[jmsservers]
172.16.1.61

[vpnservers]
172.16.1.60
```

#### 推送公钥至所有主机

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.1。16.*     #<---这里是指地址规划中的所有主机，逐个下发
```

#### 测试连通性

```bash
ansible all -m ping
```

#### 创建变量目录&文件

```bash
mkdir /etc/ansible/roles/group_vars
touch /etc/ansible/roles/group_vars/all
```

---

### Ansible初始化网络

#### 修改网卡配置

- 编写删除更改NDS&Gateway的playbook

```bash
vim network_init.yml

- hosts: all:!dnsservers:!routes
  tasks:

  - name: Delete DNS
    lineinfile:
      path: /etc/sysconfig/network-scripts/ifcfg-eth1	[[要操作的文件]]
      regexp: 'DNS*'		[[要执行的操作，匹配到DNS]]*的字段
      state: absent			[[要执行的操作，删除匹配到的字段]]

  - name: Add NDS
    lineinfile:
      path: /etc/sysconfig/network-scripts/ifcfg-eth1
      line: "DNS1=223.5.5.5"	[[要执行的操作，添加该内容]]

  - name: Delete Gateway
    lineinfile:
      path: /etc/sysconfig/network-scripts/ifcfg-eth1
      regexp: '^GATEWAY='
      state: absent

  - name: Add Gateway
    lineinfile:
      path: /etc/sysconfig/network-scripts/ifcfg-eth1
      line: "GATEWAY=172.16.1.200"

  - name: Restart Network
    systemd:				[[systemd模块]]
      name: network			[[要操作的服务名称]]
      state: restarted		[[要执行的操作，重启]]
```

#### 执行playbook

```bash
ansible-playbook network_init.yml
```

#### 查看后端配置

- 方法1		-繁琐

```bash
ssh root@172.16.1.4			[[ssh到后端主机]]

cat /etc/sysconfig/network-scripts/ifcfg-eth1			[[查看修改后的网卡信息是否符合预期]]
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
NAME=eth1
DEVICE=eth1
ONBOOT=yes
IPADDR=172.16.1.4
PREFIX=24
DNS1=223.5.5.5						[[由ansible-playbook添加而来的DNS]]
GATEWAY=172.16.1.200				[[由ansible-playbook添加而来的GATEWAY]]

exit								[[试完毕退回ansible]] bash
```

- 方法2		-简单

```bash
ansible all -m shell -a "cat /etc/sysconfig/network-scripts/ifcfg-eth1 | egrep 'DNS|GATEWAY'"
```

---

### Ansible初始化环境

#### 准备基础环境路径

```bash
mkdir base/{tasks,handlers,files,templates} -p        [[注意：]]{}内的，必须为英文！
```

#### 关闭防火墙

```bash
vim base/tasks/firewall.yml

- name: Disable Selinux Firwall
  selinux:
    state: disabled

- name: Diable Firewalld
  systemd:
    name: firewalld
    state: stopped
    enabled: no
```

#### 创建统一用户

- 编写创建用户&组的playbook

```bash
vim base/tasks/user.yml

- name: Create Group User
  group:
    name: "{{ all_group }}"
    gid: "{{ all_gid }}"

- name: Create User
  user: 
    name: "{{ all_user }}"
    uid: "{{ all_uid }}"
    group: "{{ all_group }}"
    create_home: no 
    shell: /sbin/nologin
```

- 添加对应变量

```bash
vim group_vars/all
	
[[在base栏添加以下变量]]
all_group: www
all_user: www
all_uid: 666
all_gid: 666
```

#### 添加仓库

```bash
vim base/tasks/yum_repository.yml

- name: Add Base Yum Repository
  yum_repository:
    name: base
    description: Base Aliyun Repositoey
    baseurl: http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
    gpgcheck: yes
    gpgkey: http://mirrors.aliyun.com/centos/RPM-GPG-KEY-Centos-7

- name: Add Epel Yum Repository
  yum_repository:
    name: epel
    description: Epel Aliyun Repositoey
    baseurl: http://mirrors.aliyun.com/epel/7/$basearch/
    gpgcheck: no

- name: Add Nginx Yum Repository
  yum_repository:
    name: nginx
    description: Nginx Repository
    baseurl: http://nginx.org/packages/centos/7/$basearch/
    gpgcheck: no

- name: Add PHP Yum Repository
  yum_repository:
    name: php71w
    description: php Repository
    baseurl: http://us-east.repo.webtatic.com/yum/el7/x86_64/
    gpgcheck: no
    
- name: Add Haproxy Yum Repository
  yum_repository:
    name: haproxy
    description: haproxy repository
    baseurl: https://repo.ius.io/archive/7/$basearch/
    gpgcheck: yes
    gpgkey: https://repo.ius.io/RPM-GPG-KEY-IUS-7
```

#### 安装基础软件包

```bash
vim base/tasks/yum_pkg.yml

- name: Installed Packages All
  yum:
    name: "{{ item }}"
    state: present
  loop:
    - rsync
    - nfs-utils
    - net-tools
    - wget
    - tree
    - lrzsz
    - vim
    - unzip
    - httpd-tools
    - bash-completion
    - iftop
    - iotop
    - glances
    - gzip
    - psmisc
    - MySQL-python			[[安装数据库时用到]]
    - bind-utils
```

#### 调整文件描述符

```bash
vim base/tasks/limits.yml

- name: Change Linux /etc/security/limit.conf
  pam_limits:
    domain: "*"
    limit_type: "{{ item.limit_type }}"
    limit_item: "{{ item.limit_item }}"
    value: "{{ item.value }}"
  loop:
    - { limit_type: 'soft', limit_item: 'nofile', value: '100000' }
    - { limit_type: 'hard', limit_item: 'nofile', value: '100000' }
```

#### 内核参数调整

```bash
vim base/tasks/kernel.yml

- name: Change Port Range
  sysctl:
    name: net.ipv4.ip_local_port_range			[[端口数范围]]
    value: '1024 65000'
    sysctl_set: yes
    state: present

- name: Enabled Forward							[[开启forward转发]]
  sysctl:
    name: net.ipv4.ip_forward
    value: '1'
    sysctl_set: yes
    state: present

- name: Enabled tcp_reuse
  sysctl:
    name: net.ipv4.tcp_tw_reuse					[[端口重用]]
    value: '1'
    sysctl_set: yes
    state: present

- name: Chanage tcp tw_buckets
  sysctl:
    name: net.ipv4.tcp_max_tw_buckets			[[LIME_WAIT状态回收阈值]]
    value: '5000'
    sysctl_set: yes
    state: present

- name: Chanage tcp_syncookies
  sysctl:
    name: net.ipv4.tcp_syncookies				[[防止SYN]] Flood攻击，开启后max_syn_backlog理论上没有最大值
    value: '1'
    sysctl_set: yes
    state: present

- name: Chanage tcp max_syn_backlog
  sysctl:
    name: net.ipv4.tcp_max_syn_backlog			[[SYN半连接队列可储存的最大值]]
    value: '8192'
    sysctl_set: yes
    state: present

- name: Chanage tcp Established Maxconn
  sysctl:
    name: net.core.somaxconn					[[SYN全连接队列可储存的最大值]]
    value: '32768'
    sysctl_set: yes
    state: present

- name: Chanage tcp_syn_retries
  sysctl:
    name: net.ipv4.tcp_syn_retries				[[发送SYN包重试次数，默认6]]
    value: '2'
    sysctl_set: yes
    state: present

- name: Chabage net.ipv4.tcp_synack_retries			[[返回syn]]+ack重试次数，默认5
  sysctl:
    name: net.ipv4.tcp_synack_retries
    value: '2'
    sysctl_set: yes
    state: present
```

### Ansible基础环境测试

#### 汇总调用YML

```bash
vim base/tasks/main.yml

- name: Firewalld
  include: firewall.yml

- name: Kernel Parameters
  include: kernel.yml

- name: Flie Limits
  include: limits.yml

- name: Establish User&group
  include: user.yml

- name: Yum Repository
  include: yum_repository.yml

- name: Yum packages
  include: yum_pkg.yml
```

#### 编写调用角色playbook

```bash
vim /etc/ansible/roles/top.yml

- hosts: all 
  roles:
    - role: base
```

#### 测试基础环境角色

```bash
ansible-playbook top.yml                       [[必须在roles中执行]]
```

---

### Ansible应用编配-NFS

#### 创建NFS角色

```bash
mkdir nfs-server/{tasks,bandlers,templates} -p
```

- 进入templates目录中

```bash
cd /etc/ansible/roles/nfs-server/templates/
```

#### 拷贝配置文件

```bash
scp root@172.16.1.32:/etc/exports ./expots.j2       [[exports意为出口]]	[[expots意为样品]]
```

#### 编辑配置文件

```bash
[root@manager templates]# vim expots.j2 

{{ nfs_share_blog }} {{ nfs_allow_ip }}(rw,all_squash,anonuid={{ all_uid }},anongid={{ all_gid }})
{{ nfs_share_zrlog }} {{ nfs_allow_ip }}(rw,all_squash,anonuid={{ all_uid }},anongid={{ all_gid }})
```

#### 添加对应变量

```bash
vim /etc/ansible/roles/group_vars/all

[[nfs栏添加以下变量]]					[[自行按需求分类，方便查看]]
nfs_share_blog: /data/blog
nfs_share_zrlog: /data/zrlog
nfs_allow_ip: 172.16.1.0/24
```

#### 编辑角色main.yml

```bash
vim nfs-server/tasks/main.yml

- name: Configre NFS Server
  temlate:
    src: exports.j2
    dest: /etc/exports
    owner: root
    group: root
    mode: '0644'
  notify: Restart NFS Server			[[如果文件发生改变则触发，交由bandlers触发]]

- name: Create NFS dir 
  file:
    path: "{{ item }}"					[[引用循环]]
    state: directory
    owner: "{{ all_user }}"				[[引用定义的用户变量]]
    group: "{{ all_group }}"              [[引用定义的组变量]]
    mode: '0755'
    recurse: yes				[[递归]]
  loop:
    - "{{ nfs_share_blog }}"			[[循环内容]]
    - "{{ nfs_share_zrlog }}"

- name: Start NFS Server
  systemd:
    name: nfs
    state: started
```

#### 编辑notify的bandlers

```bash
vim  /etc/ansible/roles/nfs-server/bandlers/main.yml

- name: Restart NFS Server
  systemd:
    name: nfs
    state: restarted
```

#### 测试NFS应用

- 添加角色引用到主yml内

```bash
vim /etc/ansible/roles/top.yml 

[[-]] hosts: all 						把测试过的环境注释掉，节省时间
#  roles:
#    - role: base

- hosts: nfsservers
  roles:
    - role: nfs-server
```

- 执行ansible-playbook


```bash
ansible-playbook top.yml                     [[必须在roles中执行]]
```

#### 检测对端主机应用情况

- 检查对端配置文件

```nash
ansible nfsservers -m shell -a "cat /etc/exports"
```

- 检查对端服务启动情况

```bash
ansible nfsservers -m shell -a "systemctl status nfs"
```

---

### Ansible应用编配-MySQL

**注意：**MariaDB5.5安装后默认无密码，5.7或其他版本安装后可能存在默认密码，有密码环境下需删除数据库密码以便Ansible执行

**或者：**在Ansible的mariadb编配mysql_user模块中添加原有本地主机user和pass

#### 删除数据库密码操作

##### 登录数据库主机

- 进入数据库

```bash
mysql -uroot -p123
```

- 查看密码

```bash
select Host,User,Password from mysql.user;
```

- 删除本地用户密码

```bash
update mysql.user set Password='' where Host="localhost" and user="root";
```

- 检查删除

```bash
select Host,user,Password from mysql.user;
```

- 使更改生效并退出

```bash
flush privileges		[[不重启服务并使所作更改生效]]
exit
```

- 无密码登录检测

```bash
mysql		[[正常进入则更改成功]]
```

**扩展应用：**删除无用数据库用户

```bash
select Host,user from mysql.user	[[查看用户表]]
delete from mysql.user where User="Username" and Host="Host"
```

#### 创建MariaDB角色

```bash
mkdir mariadb/{tasks,handlers,templates} -p
```

#### 编辑角色main.yml

```bash
- name: Installd Mariadb
  yum:
    name: "{{ item }}"
    state: present
  loop:
    - mariadb-server
    - mariadb

- name: Start Mariadb Server
  systemd:
    name: mariadb
    state: started
    enabled: yes

- name: Removes all anonymous user accounts
  mysql_user:
    name: ''
    host_all: yes
    state: absent

- name: Create Super User {{ mysql_super_user }}
  mysql_user:
    [[login_user]]: root                [[有密码条件下添加]]
    [[login_pass]]: 123
    name: "{{ mysql_super_user }}"
    password: "{{ mysql_super_pass  }}"
    priv: "{{ mysql_super_user_priv }}"
    host: "{{ msyql_allow_ip }}"
    state: present
```

#### 添加对应变量

```bash
vim /etc/ansible/roles/group_vars/all

[[mysql栏添加以下变量]]
mysql_super_user: ansible_app
mysql_super_pass: oldxu.net123
mysql_super_user_priv: '*.*:all'
msyql_allow_ip: '172.16.1.%'
```

#### 测试MariaDB应用

```bash
vim /etc/ansible/roles/top.yml

- hosts: dbservers
  roles:
    - role: mariadb
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml                     [[必须在roles中执行]]
```

#### 检测对端主机应用情况

- 检查是否安装启动

```bash
systemctl status mariadb
```

- 检查远程连接情况

```bash
mysql -h 172.16.1.52 -uansible_app -poldxu.net123	[[在非数据库主机且安装有mariadb的主机上测试，可两台数据库之间互测]]
```

---

### Ansible应用编配-Redis

#### 创建Redis角色

```bash
mkdir redis/{tasks,handlers,templates} -p
```

#### 编写角色main.yml

```bash
vim /etc/ansible/roles/redis/tasks/main.yml

- name: Installed Redis Server
  yum:
    name: redis
    state: present

- name: Configure Redis Server
  template:
    src: redis.conf.j2
    dest: /etc/redis.conf
    owner: redis
    group: root
    mode: '0640 '
  notify: Restart Redis Server

- name: Server Redis Server
  systemd:
    name: redis
    state: started
    enabled: yes
```

#### 进入templates目录中

```bash
cd /etc/cd /etc/ansible/roles/redis/templates/
```

#### 拷贝配置文件

```bash
yum install redis -y			[[下载redis]]
cp /etc/redis.conf ./redis.conf.j2				[[拷贝配置文件至当前目录下]]
```

#### 编辑配置文件

```bash
vim ./redis.j2				[[修改bind网端为变量]]
bind 127.0.0.1 {{ ansible_eth1.ipv4.address }}			[[/搜索bind关键字]]
```

#### 添加对应变量

```bash

```

#### 编辑notify的handlers

```bash
vim redis/handlers/main.yml

- name: Restart Redis Server
  systemd:
    name: redis
    state: restarted
```

#### 测试Redis应用

- 添加角色引用到主yml内

```bash
vim /etc/ansible/roles/top.yml

- hosts: redisservers
  roles:
    - role: redis
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml                     [[必须在roles中执行]]
```

#### 检测对端主机应用情况

- 连接远程Redis

```bash
redis-cli -h 172.16.1.41		[[连接远程主机Redis]]
```

- 登录对端主机检查配置文件

```bash
ssh root@172.16.1.41			[[登录后记得退出]]
vim /etc/redis.conf
```

---

### Ansible应用编配-Nginx

#### 创建Nginx角色

```
mkdir nginx/{tasks,handlers,templates} -p
```

#### 编辑角色main.yml

```bash
vim /etc/ansible/roles/nginx/tasks/main.yml

- name: Install Nginx Server
  yum:
    name: nginx
    enablerepo: nginx
    start: present

- name: Configure Nginx nginx.conf
  template:
    src:
    dest: /etc/nginx/nginx.conf
  notify: Restart Nginx Server

- name: Start Nginx Server
  systemd:
    name: nginx
    state: started
    enabled: yes
```

#### 编辑notify的handlers

```bash
vim nginx/handlers/main.yml

- name: Restart Nginx Server
  systemd:
    name: nginx
    state: restarted
```

- 进入templates目录中

```bash
cd /etc/ansible/roles/nginx/templates/
```

#### 拷贝配置文件

```bash
scp root@172.16.1.7:/etc/nginx/nginx.conf ./nginx.conf.j2
```

#### 编辑配置文件

```bash
user  {{ all_user }};		[[基础环境定义的用户变量]]
worker_processes  {{ ansible_processor_vcpus }};		[[全局变量]]

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  {{ ansible_processor_vcpus * 1024 }};			[[全局变量]]
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    [[tcp_nopush]]     on;

    keepalive_timeout  65;

    [[gzip]]  on;
    include {{ nginx_include_path }};			[[需定义路径变量]]
}
```

#### 添加对应变量

```bash
vim /etc/ansible/roles/group_vars/all

[[nginx栏添加以下变量]]
nginx_conf_path: /etc/nginx/nginx.conf
nginx_include_dir: /etc/nginx/conf.d
nginx_include_path: /etc/nginx/conf.d/*.conf
```

#### 测试Nginx应用

- 添加角色引用到主yml内

```bash
vim /etc/ansible/roles/top.yml

- hosts: webservers
  roles:
    - role: nginx
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml			[[必须在roles中执行]]
```

#### 检测对端主机应用情况

- 检查主配置文件

```bash
ansible webservers -m shell -a "vim /etc/nginx/nginx.conf"
```

- 检查nginx启动情况

```bash
ansible webservers -m shell -a "systemctl status nginx"
```

---

### Ansible应用编配-PHP

#### 创建PHP角色

```bash
mkdir php-fpm/{tasks,handlers,templates} -p
```

#### 编辑角色main.yml

```bash
vim /etc/ansible/roles/php-fpm/tasks/main.yml

- name: Installed PHP-FPM Server
  yum:
    name: "{{ item }}"
    enablerepo: php71w
    state: present
  loop:
      - php71w
      - php71w-cli
      - php71w-common
      - php71w-devel
      - php71w-embedded
      - php71w-gd
      - php71w-mcrypt
      - php71w-mbstring
      - php71w-pdo
      - php71w-xml
      - php71w-fpm
      - php71w-mysqlnd
      - php71w-opcache
      - php71w-pecl-memcached
      - php71w-pecl-redis
      - php71w-pecl-mongodb

- name: Configure PHP php.ini php-fpm
  template:
    src: "{{ item.src }}"
    dest: "{{ item.dest }}"
  loop:
    - { src: php.ini.j2 , dest: "{{php_ini_path}}" }
    - { src: fpm-www.conf.j2 , dest: "{{php_fpm_path}}" }
  notify: Restart PHP Server

- name: Start PHP-FPM Server
  systemd:
    name: php-fpm
    state: started
    enabled: yes
```

- 进入templates目录中

```bash
cd /etc/ansible/roles/php-fpm/templates/
```

#### 拷贝配置文件

```bash
scp root@172.16.1.7:/etc/php.ini ./php.ini.j2
scp root@172.16.1.7:/etc/php-fpm.d/www.conf ./fpm-www.conf.j2
```

#### 	编辑配置文件

- 修改php.ini.j2

```bash
vim /etc/ansible/roles/php-fpm/templates//php.ini.j2
/session				[[搜索session]]

session.save_handler = {{ session_method }}			[[修改缓存session类型变量]]
session.save_path = "{{ session_redis_path }}"		[[添加redis路径变量]]
```

- 修改fpm-www.conf.j2

```bash
[[排除]];与空格的行显示基础配置
egrep -v "^;|^$" fpm-www.conf.j2

[[复制基础配置]]

[[全删除后粘贴到配置文件中]]

[[修改为变量]]
[www]
user = {{ all_user }}
group = {{ all_group }}
listen = 127.0.0.1:9000
listen.allowed_clients = 127.0.0.1
pm = dynamic
pm.max_children = {{ fpm_max_process }}
pm.start_servers = {{ fpm_start_process }}
pm.min_spare_servers = {{ fpm_min_spare_servers }}
pm.max_spare_servers = {{ fpm_max_spare_servers }}
slowlog = /var/log/php-fpm/www-slow.log
php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
php_value[soap.wsdl_cache_dir]  = /var/lib/php/wsdlcache
```

#### 添加对应变量

```bash
[[在php栏添加以下变量]]        
php_ini_path: /etc/php.ini    
php_fpm_path: /etc/php-fpm.d/www.conf

session_method: redis
session_redis_path: "tcp://172.16.1.41:6379?weight=1&timeout=2.5"

fpm_max_process: 100
fpm_start_process: 10
fpm_min_spare_servers: 10
fpm_max_spare_servers: 50
```

#### 测试PHP应用

- 添加角色引用到主yml内

```bash
vim /etc/ansible/roles/top.yml

- hosts: webservers
  roles:
    - role: php-fpm
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml			[[必须在roles中执行]]
```

#### 检测对端主机应用情况

- 检查对端配置文件

```bash
[[检查/etc/php]].ini
ansible webservers -m shell -a "cat /etc/php.ini |grep session.save_handler"
ansible webservers -m shell -a "cat /etc/php.ini |grep  session.save_path"

[[检查/etc/php-fpm]].d/www.conf
ansible webservers -m shell -a "cat /etc/php-fpm.d/www.conf"
```

- 检查启动情况

```bash
ansible webservers -m shell -a "systemctl status php-fpm"
```

- 检查端口情况

```bash
ansible webservers -m shell -a "netstat -lntp |grep 9000"
```

---

### Ansible应用编配-Haproxy

#### 创建Haproxy角色

```bash
mkdir haproxy/{tasks,handlers,templates} -p
```

#### 编辑角色main.yml

```bash
vim /etc/ansible/roles/haproxy/tasks/main.yml

- name: Install Haproxy Server
  yum:
    name: haproxy22
    enablerepo: haproxy
    state: present

- name: Configure Haproxy Server
  template:
    src: haproxy.cfg.j2
    dest: /etc/haproxy/haproxy.cfg
  notify: Restart Haproxy Server

- name: Create Haproxy Include Dir
  file:
    path: "{{ haproxy_include_path }}"
    state: directory

- name: Change Service Configure Add
  lineinfile:
    path: /usr/lib/systemd/system/haproxy.service
    insertafter: '^\[Service\]'
    line: 'Environment="CONFIG_D={{ haproxy_include_path }}"'

- name: Change Service Configure Delete
  lineinfile:
    path: /usr/lib/systemd/system/haproxy.service
    regexp: '^ExecStart='
    line: 'ExecStart=/usr/sbin/haproxy -f $CONFIG -f $CONFIG_D -c -q $OPTIONS'

- name: Change Service Configure ExecStartpre
  lineinfile:
    path: /usr/lib/systemd/system/haproxy.service
    regexp: 'ExecStartPre='
    line: 'ExecStartPre=/usr/sbin/haproxy -f $CONFIG -f $CONFIG_D -c -q $OPTIONS'

- name: Start Haproxy Server
  systemd:
    name: haproxy
    state: started
    enabled: yes
```

#### 编辑notify的handlers

```bash
vim /etc/ansible/roles/haproxy/handlers/main

- name: Restart Haproxy Server
  systemd:
    name: haproxy
    state: restarted
```

- 进入templates目录中

```bash
cd /etc/ansible/roles/haproxy/templates/
```

#### 拷贝配置文件

```bash
scp root@172.16.1.5:/etc/haproxy/haproxy.cfg ./haproxy.cfg.j2
```

#### 编辑配置文件

```bash
vim haproxy.cfg.j2

global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats level admin

    [[nbproc]] 4
	nbthread 8
    cpu-map 1 0
    cpu-map 2 1
    cpu-map 3 2
    cpu-map 4 3

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000


listen haproxy-stats_2
	bind *:9999
	stats enable
	stats refresh 1s
	stats hide-version
	stats uri /haproxy?stats
	stats realm "HAProxy statistics"
	stats auth admin:123456
	stats admin if TRUE
```

#### 添加对应变量

```bash
vim /etc/ansible/roles/group_vars/all

[[在haproxy栏添加以下变量]]
haproxy_include_path = /etc/haproxy/conf.d
```

#### 测试Haproxy应用

- 添加角色引用到主yml内

```bash
vim /etc/ansible/roles/top.yml

- hosts: proxyservers
  roles:
    - role: haproxy
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml			[[必须在roles中执行]]
```

#### 检测对端主机应用情况

- 检查对端配置文件

```bash
ansible proxyservers -m shell -a "systemctl cat haproxy"
```

---

### Ansible应用编配-Keepalived

#### 创建Keepalived角色

```bash
mkdir keepalived/{tasks,handlers,templates} -p
```

#### 编辑角色main.yml

```bash
vim keepalived/tasks/main.yml

- name: Install Keeplived Server
  yum:
    name: keepalived
    state: present

- name: Configure Keepalived Server
  template:
    src: keepalived.conf.j2
    dest: /etc/keepalived/keepalived.conf 
  notify: Restart Keepalived Server

- name: Started Keepalived Server
  systemd:
    name: keepalived
    state: started
    enabled: yes
```

#### 编辑角色notify的handlers

```bash
vim /etc/ansible/roles/keepalived/handlers/main.yml

- name: Restart Keepalived Server
  systemd:
    name: keepalived
    state: restarted
```

- 进入templates目录中

```bash
cd /etc/ansible/roles/keepalived/templates
```

#### 拷贝配置文件

```bash
scp root@172.16.1.5:/etc/keepalived/keepalived.conf ./keepalived.conf.j2
```

#### 编辑配置文件

```bash
vim ./keepalived.conf.j2

global_defs {
        router_id {{ ansible_hostname }}
}

vrrp_instance VI_1 {

{% if ansible_hostname == "proxy01" %}
        state MASTER
        priority 200
{% elif ansible_hostname == "proxy02" %}
        state BACKUP
        priority 100
{% endif %}

        interface eth1
        virtual_router_id 49
        advert_int 3
        authentication {
                auth_type PASS
                auth_pass 1111
        }

virtual_ipaddress {
        {{ proxy_vip }}
}
```

#### 添加对应变量

```bash
vim /etc/ansible/roles/group_vars/all

[[在keepalived栏添加以下内容]]
proxy_vip:172.16.1.100
```

#### 测试keepalived应用情况

- 添加角色引用到主yml内

```bash
vim /etc/ansible/roles/top.yml

- hosts: proxyservers
  roles:
    - role: keepalivedm 
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml			[[必须在roles中执行]]
```

#### 检测对端主机应用情况

- 检查配置文件

```bash
ansible-playbook lbservers -m shell -a "cat /etc/keepalived/keepalived.conf"
```

- 检查启动情况

```bash
ansible lbserver -m shell -a "systemctl status keepalived"
```

- 检查IP地址

```bash
ansible lbservers -m shell -a "ip addr |grep 172.16.1.100"
```

- 检查地址漂移

```bash
ansible 172.16.1.5 -m shell -a "systemctl stop keepalived"			[[关闭proxy01的keepalivved]]
ansible lbservers -m shell -a "ip addr |grep 172.16.1.100"			[[查看地址位置]]
```

---

### Ansible应用编配-LVS

#### 创建lvs角色

```bash
mkdir /etc/ansible/roles/lvs/{tasks,handlers,templates,meta} -p
```

#### 编辑依赖main.yml

```bash
vim lvs/meta/main.yml

dependencies:
  - { role: keepalived }
```

#### 编辑角色main.yml

```bash
vim lvs/tasks/main.yml

- name: Install Ipvsadm Packages
  yum:
    name: ipvsadm
    state: present

- name: Configure LVS Keepalived
  template:
    src: keepalived.conf.j2
    dest: /etc/keepalived/keepalived.conf
  notify: Restart Keepalived Server

- name: start LVS Keepalived
  systemd:
    name: keepalived
    state: started
    enabled: yes
```

- 进入templers目录

```bash
cd /etc/ansible/roles/lvs/templates
```

#### 拷贝配置文件

```bash
scp root@172.16.1.3:/etc/keepalived/keepalived.conf  ./keepalived.conf.j2
```

#### 编辑配置文件

```bash
vim keepalived.conf.j2

global_defs {
    router_id {{ ansible_hostname }}
}

vrrp_instance VI_1 {

{% if ansible_hostname == "lvs01" %}
    state MASTER
    priority 200
{% elif ansible_hostname == "lvs02" %}
    state BACKUP
    priority 100
{% endif %}

    interface eth1
    virtual_router_id 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
 }
    virtual_ipaddress {
        {{ lvs_vip }}
    }
}

[[Http]]
virtual_server {{ lvs_vip }} {{ lvs_port_http }} {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    protocol TCP

{% for host in groups['proxyservers'] %}
    real_server {{ host }} {{ lvs_port_http }} {
        weight 1
        TCP_CHECK {
            connect_port 80
            connect_timeout  3
            nb_get_retry 2
            delay_beefore_retry 3
        }
    }
{% endfor %}
}

[[Https]]
virtual_server {{ lvs_vip }} {{ lvs_port_https }} {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    protocol TCP

{% for host in groups['proxyservers'] %}
    real_server {{ host }} {{ lvs_port_https }} {
        weight 1
        TCP_CHECK {
            connect_port 80
            connect_timeout  3
            nb_get_retry 2
            delay_beefore_retry 3
        }
    }
{% endfor %}
}
```

#### 添加相应变量

```bash
vim /etc/ansible/roles/group_vars/all

[[在lvs栏添加以下变量]]
lvs_vip: 172.16.1.100
lvs_port_http: 80
lvs_port_https: 443
```

#### 测试LVS应用情况

- 添加角色引用到主yml中

```bash
vim /etc/ansible/roles/top.yml

- hosts: lbservers
  roles:
    - role: lvs
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml			[[必须在roles中执行]]
```

#### 检测对端主机应用情况

- 检查配置文件

```bash
ansible-playbook lbservers -m shell -a "cat /etc/keepalived/keepalived.conf"
```

- 检查启动情况

```bash
ansible lbserver -m shell -a "systemctl status keepalived"
```

- 检查IP地址

```bash
ansible lbservers -m shell -a "ip addr |grep 172.16.1.100"
```

- 检查地址漂移

```bash
ansible 172.16.1.5 -m shell -a "systemctl stop keepalived"			[[关闭proxy01的keepalivved]]
ansible lbservers -m shell -a "ip addr |grep 172.16.1.100"			[[查看地址位置]]
```

---

### Ansible应用编配-LVS-devel

#### 创建Lvs-devel角色

```bash
mkdir lvs-devel/{tasks,handlers,templates} -p
```

#### 编辑角色main.yml

```bash
vim lvs-devel/tasks/main.yml

- name: Configure VIP lo:0
  template:
    src: ifcfg-lo:0.j2
    dest: /etc/sysconfig/network-scripts/ifcfg-{{ lvs_rs_network }}
  notify: Restart Network

- name: Configure Arp_Ignore
  sysctl:
    name: "{{ item }}"
    value: '1'
    sysctl_set: yes
  loop:
    - net.ipv4.conf.default.arp_ignore
    - net.ipv4.conf.all.arp_ignore
    - net.ipv4.conf.lo.arp_ignore

- name: Configure Arp_Announce 
  sysctl:
    name: "{{ item }}"
    value: '2'
    sysctl_set: yes
  loop:
    - net.ipv4.conf.default.arp_announce
    - net.ipv4.conf.all.arp_announce
    - net.ipv4.conf.lo.arp_announce
```

#### 编辑角色notify的handlers

```bash
vim lvs-devel/handlers/main.yml

- name: Restart Network
  service:
    name: network
    state: restarted
    args: lo:0
```

- 进入templaers目录

```bash
cd /etc/ansible/roles/keepalived/templates
```

#### 创建配置文件

```bash
vim lvs-devel/templates/ifcfg-lo:0.j2

DEVICE={{ lvs_rs_network }}
IPADDR={{ lvs_vip }}
NETMASK=255.0.0.0
ONBOOT=yes
NAME=loopback
```

#### 添加相应变量

```bash
vim /etc/ansible/roles/group_vars/all

[[在lvs-devel栏添加以下变量]]
lvs_rs_network: lo:0
```

#### 测试LVS-devel应用情况

- 添加角色引用到主yml中

```bash
vim /etc/ansible/roles/top.yml

- hosts: proxyservers
  roles:
    - role: lvs-devel
```

- 执行ansible-plzybook

```bash
ansible-playbook top.yml			[[必须在roles中执行]]
```

**注意：**再次执行ansible时可能出现ssh不通问题，此时将对端**lo:0**down掉

#### 检测对端主机应用情况

- 检查网卡文件

```bash
ansible proxyservers -m shell -a "cat /etc/sysconfig/network-scripts/ifcfg-lo:0"
```

- 检查启动情况

```bash
ansible proxyservers -m shell -a "systemctl status keepalived"
```

- 检查IP地址

```bash
ansible proxyservers -m shell -a "ip addr |grep 172.16.1.100"
```

---

### Ansible应用编配-Route

#### 创建Route角色

```bash
mkdir route/tasks -p
```

#### 编辑角色mai.yml

```bash
vim route/tasks/main.yml

- name: Iptables SNAT Share Network
  iptables:
    table: nat 
    chain: POSTROUTING 
    source: 172.16.1.0/24 
    jump: SNAT 
    to_source: "{{ ansible_eth0.ipv4.address }}"


- name: Iptables DNAT Http 80 Port
  iptables:
    table: nat 
    chain: PREROUTING
    protocol: tcp 
    destination: "{{ ansible_eth0.ipv4.address }}"
    destination_port: '{{ lvs_port_http|int }}'
    jump: DNAT 
    to_destination: "{{ lvs_vip }}:{{ lvs_port_http }}"

- name: Iptables DNAT Http 443 Port
  iptables:
    table: nat 
    chain: PREROUTING
    protocol: tcp 
    destination: "{{ ansible_eth0.ipv4.address }}" 
    destination_port: '{{ lvs_port_https|int }}' 
    jump: DNAT 
    to_destination: "{{ lvs_vip }}:{{ lvs_port_https }}"
```

#### 测试Route应用情况

- 添加角色引用到主yml中

```bash
vim /etc/ansible/roles/top.yml

- hosts: routes
  roles:
    - role: route
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml			[[必须在roles中执行]]
```

#### 检测对端主机应用情况

- 检查iptables

```bash
ansible routes -m shell -a "iptables -t nat -L -n"
```

---

### Ansible应用编配-DNS

#### 创建DNS角色

```bash
mkdir dns/{tasks,handlers,templates} -p
```

#### 编辑角色main.yml

```bash
vim dns/tasks/main.yml

- name: Install Bind Server
  yum:
    name: "{{ item }}"
    state: present
  loop:
    - bind-utils
    - bind

- name: Configure named.conf
  template:
    src: named.conf.j2
    dest: /etc/named.conf
    owner: root
    group: named
    mode: '0640'
  notify: Restart Bind Server

- name: Configure oldxu.net zone
  template:
    src: oldxu.net.zone.j2
    dest: "{{ dns_zone_path}}/{{ oldxu_net_zone_name }}"
  when: ( ansible_hostname == "dns-master" )
  notify: Restart Bind Server

- name: Start BIND Server
  systemd:
    name: named
    state: started
    enabled: yes
```

#### 编辑角色notify的handlers

```bash
vim /dns/handlers/main.yml

- name: Restart Bind Server
  systemd:
    name: named
    state: restarted
```

- 进入templates目录

```bash
cd /etc/ansible/roles/dns/templates
```

#### 编辑配置文件

```bash
vim named.conf.j2

options {
	listen-on port 53 { any; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	recursing-file  "/var/named/data/named.recursing";
	secroots-file   "/var/named/data/named.secroots";
	allow-query     { any; };


{% if ansible_hostname == "dns-master" %}
 	 allow-transfer {172.16.1.92;};   //允许哪些`IP`地址能同步Master配置信息；
	 also-notify {172.16.1.92;};       //Master主动通知Slave域名变发生了变更      
{% elif ansible_hostname == "dns-slave" %}
	masterfile-format text;
{% endif %}

	recursion yes;
	dnssec-enable yes;
	dnssec-validation yes;
	bindkeys-file "/etc/named.root.key";
	managed-keys-directory "/var/named/dynamic";
	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

    zone "." IN {
        type hint;
        file "named.ca";
    };

{% if ansible_hostname == "dns-master" %}
    zone "oldxu.net" IN {
	type master;
	file "{{ oldxu_net_zone_name }}";
    };
{% elif ansible_hostname == "dns-slave" %}
    zone "oldxu.net" IN {
	type slave;
	file "slaves/{{ oldxu_net_zone_name }}";
	masters { {{ dns_master_ip }}; };
    };
{% endif %}

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```

```bash
vim oldxu.net.zone.js

$TTL 600;

oldxu.net.	IN	SOA	ns.oldxu.net.	qq.oldxu.net. (
	202201011
	1800
	900
	604800
	86400
)

oldxu.net.	IN	NS	ns1.oldxu.net.
oldxu.net.	IN	NS	ns2.oldxu.net.

ns1.oldxu.net.	IN	A	{{ dns_master_ip }}
ns2.oldxu.net.	IN	A	{{ dns_slave_ip }}


www.oldxu.net.	IN	A	1.1.1.1
blog.oldxu.net.	IN	A	10.0.0.200

```

#### 添加对应变量

```bash
vim /etc/ansible/roles/group_vars/all

[[在dns栏添加以下变量]]
dns_master_ip: 172.16.1.91
dns_slave_ip: 172.16.1.92
dns_zone_path: /var/named
oldxu_net_zone_name: oldxu.net.zone
```

#### 测试DNS应用

- 添加角色引用到主yml中

```bash
vim /etc/ansible/roles/top.yml

- hosts: dnsservers
  roles:
    - role: dns
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml			[[必须在roles中执行]]
```

#### 检测对端主机应用情况

- 检查配置文件

```bash
ansible dnsservers -m shell -a "cat /etc/named.conf"
ansible dnsservers -m shell -a "cat /var/named/oldxu.net.zone"
```

- 检查解析

```bash
dig www.oldxu.net @172.16.1.91:
```

---

### 接入wordpress业务

#### 创建wordpress角色

```bash
mkdir wordpress-web/{tasksk,handlers,templates,meta,tiles} -p
```

#### 编辑依赖main.yml

```bash
vim wordpress-web/meta/main.yml

dependencies:
  - { role: nginx }
  - { role: php-fpm }
```

#### 编辑角色main.yml

```bash
- name: Create Wordpress Configure
  template:
    src: blog.oldxu.net.conf.j2
    dest: "{{ nginx_include_dir }}/{{ word_nginx_name }}"
    owner: root
    group: root
    mode: '0644'
  notify: Restart Nginx Server

- name: Create Code Directory
  file:
    path: "{{ word_code_dir }}"
    state: directory
    owner: "{{ all_user }}"
    group: "{{ all_group }}"
    recurse: yes

- name: Import Wordpress Code
  unarchive:
    src: latest-zh_CN.tar.gz
    dest: "{{ word_code_dir }}"
    owner: "{{ all_user }}"
    group: "{{ all_group }}"
    creates: "{{ word_code_path }}/wp-config.php"


- name: Copy Wordpress Connection MySQL FIle
  template:
    src: wp-config.php.j2
    dest: "{{ word_code_path }}/wp-config.php"
```

#### 添加对应变量

```bash
vim /etc/ansible/roles/group_vars/all

[[在mysql栏添加以下变量]]
mysql_server_ip: 172.16.1.51

[[在wordperss栏添加以下变量]]
word_domain: wordpress.oldxu.net
word_http_path: 80
word_code_path: /code/wordpress
word_code_der: /code
fastcgi_https: off
```

#### 编辑配置文件

- 进入角色files

```bash
cd /etc/ansible/roles/wordpress-web/files
```

- 下载wordpress

```bash
wget https://cn.wordpress.org/latest-zh_CN.tar.gz
```

#### 测试wordpress

- 添加角色引用到主yml中

```bash
vim /etc/ansible/roles/top.yml

- hosts: webservers
  roles:
    - role: webservers-web
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml			[[必须在roles中执行]]
```

#### 检测对端主机应用情况

- 检查推送情况

```bash
ansible webservers -m shell -a "ls /code/wordpress"
```

- 检查配置文件

```bash
ansible webservers -m shell -a "cat /etc/nginx/conf.d/blog.oldxu.net.conf"
ansible webservers -m shell -a "cat /code/wordpress/wp-config.php"
```

- 检查nginx

```bash
在宿主机hosts添加web地址测试网页
```

---

### 加入haproxy

#### 创建wordpress-proxy

```bash
mkdir wordpress-proxy/{handlers,templates,tasks,mete} -p
```

#### 编辑依赖main.yml

```bash
vim wordpress-proxy/mete/main.yml

dependencies:
 - { role: haproxy }
```

#### 编辑角色main.yml

```bash
vim wordpress-proxy/tasks/main.yml

- name: Wordpress Haproxy Configure
  template:
    src: wordpress.cfg.j2
    dest: "{{ haproxy_include_path }}/wordpress.cfg"
  notify: Restart Haproxy Server
```

#### 添加对应变量

```bash
vim /etc/ansible/roles/group_vars/all

[[haproxy]]
haproxy_port: 80
```

#### 测试wordpress-proxy

- 添加角色引用到主ynl中

```bash
vim /etc/ansible/roles/top.yml

- hosts: proxyservers
  roles:
    - wordpress-proxy
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml			[[必须在roles中执行]]
```

#### 总体测试

- 取消所有主yml注释并排列

```bash
vim /etc/ansible/roles/top.yml

- hosts: all 
  roles:
    - role: base

- hosts: nfsservers
  roles:
    - role: nfs-server

- hosts: dbservers
  roles:
    - role: mariadb

- hosts: redisservers
  roles:
    - role: redis

- hosts: webservers
  roles:
    - role: nginx
    - role: php-fpm
    - role: wordpress-web

- hosts: proxyservers
  roles:
#    - role: haproxy
#    - role: keepalived
    - role: lvs-devel
    - wordpress-proxy

- hosts: lbservers
  roles:
    - role: lvs

- hosts: routes
  roles:
    - role: route

- hosts: dnsservers
  roles:
    - role: dns
```

- 执行ansible-playbook

```bash
ansible-playbook top.yml			[[必须在roles中执行]]
```

 
