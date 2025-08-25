# ansible自动化集群实现部署

---

1. 准备项目hosts主机清单文件和配置

   - 创建项目目录并进入

     ```bash
     [root@manager ~]# mkdir web-cluster && cd web-cluster
     ```

   - 创建hosts主机清单文件

     ```bash
     [root@manager ~]# vim hosts
     
     [webservers]
     172.16.1.7
     172.16.1.8
     172.16.1.9
     
     
     [dbservers]
     172.16.1.41
     
     
     [lbservers]
     172.16.1.5
     172.16.1.6
     ```

   - 拷贝主配置文件到当前目录下

     ```bash
     [root@manager web_cluster]# cp /etc/ansible/ansible.cfg ./
     ```

2. 准备Redis

   - 安装

   - 配置
   - 启动

3. 准备Nginx PhP

4. 业务代码

5. 准备负载均衡

6. 准备接入https

