## 准备安装包

```` bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.6.2-linux-x86_64.tar.gz

mv kibana-8.6.2-linux-x86_64.tar.gz /opt

tar xf kibana-8.6.2-linux-x86_64.tar.gz

mv kibana-8.6.2 kibana

mkdir kibana/config/certs
````

## 准备证书

在es节点生成证书

```` bash
./elasticsearch-certutil csr -name kibana
unzip csr-bundle.zip
scp /opt/elastic/kibana/* root@10.0.0.104:/opt/kibana/config/certs/
scp /opt/elastic/config/certs/elasticsearch-ca.pem root@10.0.0.104:/opt/kibana/config/certs/
````

## 修改配置

```` bash
server.port: 5601
server.host: "10.0.0.104"
server.name: "kibana"
server.ssl.enabled: true
server.ssl.certificate: "/opt/elastic/kibana/config/certs/kibana.crt"
server.ssl.key: "/opt/elastic/kibana/config/certs/kibana.key"
elasticsearch.hosts: ["https://10.0.0.101:9200", "https://10.0.0.102:9200", "https://10.0.0.103:9200"]
elasticsearch.username: "elastic"
elasticsearch.password: "123456"
elasticsearch.ssl.certificateAuthorities: [ "/opt/elastic/kibana/config/certs/elasticsearch-ca.pem" ]
i18n.locale: "zh-CN"
````

## 启动文件

```` bash
vi /usr/lib/systemd/system/kibana.service

[Unit]
Description=kibana 8.6.2
After=network.target

[Service]
Type=simple
User=elastic
ExecStart=/opt/elastic/kibana/bin/kibana
PrivateTmp=true

[Install]
WantedBy=multi-user.target
````

