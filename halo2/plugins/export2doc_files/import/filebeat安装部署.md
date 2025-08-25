## 下载

```` bash
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.6.2-linux-x86_64.tar.gz
````

## 配置

```` bash
filebeat.inputs:
- type: log
  id: my-filestream-id
  enabled: true
  paths:
    - /opt/filebeat/logs/*.ndjson
output.logstash:
  hosts: ["10.0.0.104:5044"]

````

## 启动

```` bash
nohup ./filebeat -e -c /opt/filebeat/filebeat.yml >/dev/null 2>&1 &
````

