# elasticsearch安装

## 一、下载安装包

```` bash
mkdir /elastic
wget -O /elastic https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.6.2-linux-x86_64.tar.gz
````

## 二、解压

```` bash
cd /elastic
tar xvf elasticsearch-8.6.2-linux-x86_64.tar.gz
mv elasticsearch-8.6.2 /opt/elastic
````

## 三、配置

```` bash
# 创建用户
useradd elastic
passwd elastic

# 创建目录
mkdir /opt/elastic/to/logs -p
mkdir /opt/elastic/to/data -p
mkdir /opt/elastic/config/certs -p
# 授权
chown -R elastic:elastic /opt/elastic

# 切换用户
su - elastic 

vim /opt/elastic/config/elasticsearch.yml

node.name: node-1
path.data: /opt/elastic/to/data
cluster.name: my-application
path.logs: /opt/elastic/to/logs
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["10.0.0.101", "10.0.0.102", "10.0.0.103"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]

[[开启xpack]]
xpack.security.enabled: true


[[开启集群中https传输]]
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.client_authentication: required
xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12

[[开启api接口https传输]],配置HTTP层TLS/SSL加密传输
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: certs/http.p12
xpack.security.http.ssl.truststore.path: certs/http.p12
  
[[允许跨域]]
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization,X-Requested-With,Content-Type,Content-Length





vim /opt/elastic/config/jvm.options
-Xms256m
-Xmx256m
````

#### 系统配置

```` bash
vim /etc/security/limits.conf
elastic soft nofile 65536
elastic hard nofile 65536
elastic soft nproc  65536
elastic hard nproc  65536


vim /etc/sysctl.conf
vm.max_map_count=655360
````



#### 生成证书

```` bash
cd /opt/elastic/bin
./elasticsearch-certutil ca
./elasticsearch-certutil cert --ca elastic-stack-ca.p12
mv /opt/elastic/elastic-stack-ca.p12 /opt/elastic/elastic-certificates.p12 /opt/elastic/config/certs/
./elasticsearch-certutil http 
------------
n
y
/opt/elastic/config/certs/elastic-stack-ca.p12
n
node-1
node-2
node-3
y
10.0.0.101
10.0.0.102
10.0.0.103
y
n
mv /opt/elastic/elasticsearch-ssl-http.zip /opt/elastic/config/certs/
cd /opt/elastic/config/certs
unzip elasticsearch-ssl-http.zip
mv elasticsearch/* ./
````

#### 生成密码

```` bash
./elasticsearch-setup-passwords interactive
./elasticsearch-reset-password -u elastic -i  --上边的不行用这个
````

#### 信任证书

```` bash
./elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
./elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
./elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
./elasticsearch-keystore add xpack.security.http.ssl.truststore.secure_password
````

#### 把整个elastic目录打包，拉到其他es

## 四、启动文件

```` bash
vi /etc/systemd/system/elasticsearch.service

[Unit]
Description=elasticsearch 8.6.2
After=network.target

[Service]
Type=forking
User=elastic
ExecStart=/opt/elastic/bin/elasticsearch -d
PrivateTmp=true
# 指定此进程可以打开的最大文件数
LimitNOFILE=65535
# 指定此进程可以打开的最大进程数
LimitNPROC=65535
# 最大虚拟内存
LimitAS=infinity
# 最大文件大小
LimitFSIZE=infinity
# 超时设置 0-永不超时
TimeoutStopSec=0
# SIGTERM是停止java进程的信号
KillSignal=SIGTERM
# 信号只发送给给JVM
KillMode=process
# java进程不会被杀掉
SendSIGKILL=no
# 正常退出状态
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target



systemctl daemon-reload
systemctl enable elasticsearch.server
systemctl start elasticsearch.server
systemctl status elasticsearch.server
````

## 五、查看集群状态

```` bash
curl -k -u elastic:123456 -XGET https://10.0.0.102:9200/_cat/nodes?v

cluster：集群名称
status：集群状态 green 表示集群一切正常；yellow 表示集群不可靠但可用(单节点状态)；red 集群不可用，有故障。
node.total：节点总数量
node.data：数据节点的数量
shards：存活的分片数量
pri：主分片数量
relo：迁移中的分片数量
init：初始化中的分片数量
unassign：未分配的分片
pending_tasks：准备中的任务
max_task_wait_time：任务最长等待时间
active_shards_percent：激活的分片百分比
````

