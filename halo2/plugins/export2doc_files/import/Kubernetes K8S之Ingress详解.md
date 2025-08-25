### Kubernetes K8S之Ingress详解与示例

---

>  K8S之Ingress概述与说明，并详解Ingress常用示例 

---

#### 主机配置规划

| 服务器名称(hostname) | 系统版本  | 配置      | 内网IP       | 外网IP(模拟) |
| :------------------- | :-------- | :-------- | :----------- | :----------- |
| k8s-master           | CentOS7.7 | 2C/4G/20G | 172.16.1.110 | 10.0.0.110   |
| k8s-node01           | CentOS7.7 | 2C/4G/20G | 172.16.1.111 | 10.0.0.111   |
| k8s-node02           | CentOS7.7 | 2C/4G/20G | 172.16.1.112 | 10.0.0.112   |

---



#### Ingress概述

Ingress 是对集群中服务的外部访问进行管理的 API 对象，典型的访问方式是 HTTP和HTTPS。

Ingress 可以提供负载均衡、SSL 和基于名称的虚拟托管。

必须具有 ingress 控制器【例如 ingress-nginx】才能满足 Ingress 的要求。仅创建 Ingress 资源无效。

---



#### Ingress 是什么

Ingress 公开了从集群外部到集群内 services 的 HTTP 和 HTTPS 路由。 流量路由由 Ingress 资源上定义的规则控制。

```bash
 internet
     |
[ Ingress ]
--|-----|--
[ Services ]
```

可以将 Ingress 配置为提供服务外部可访问的 URL、负载均衡流量、 SSL / TLS，以及提供基于名称的虚拟主机。Ingress 控制器 通常负责通过负载均衡器来实现 Ingress，尽管它也可以配置边缘路由器或其他前端来帮助处理流量。

Ingress 不会公开任意端口或协议。若将 HTTP 和 HTTPS 以外的服务公开到 Internet 时，通常使用 Service.Type=NodePort 或者 Service.Type=LoadBalancer 类型的服务。

以Nginx Ingress为例，图如下

![img](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/image-20220827115828893.png)

![img](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/1620-20220827114637929.png)

---



#### Ingress示例

---

##### 架构图

![img](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/1620-20220827114637947.png)

---



#### 部署Ingress-Nginx

该Nginx是经过改造的，而不是传统的Nginx。

Ingress-Nginx官网地址

```java
https://kubernetes.github.io/ingress-nginx/
```

Ingress-Nginx GitHub地址

```java
https://github.com/kubernetes/ingress-nginx
```

本次下载版本：nginx-0.30.0

![img](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/1620-20220827114637962.png)

镜像下载与重命名

```bash
docker pull registry.cn-beijing.aliyuncs.com/google_registry/nginx-ingress-controller:0.30.0
docker tag 89ccad40ce8e quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.30.0
docker rmi  registry.cn-beijing.aliyuncs.com/google_registry/nginx-ingress-controller:0.30.0
```

ingress-nginx的yaml文件修改后并启动

```bash
# 当前目录
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress

# 获取NGINX: 0.30.0
[root@k8s-master ingress]# wget https://github.com/kubernetes/ingress-nginx/archive/nginx-0.30.0.tar.gz
[root@k8s-master ingress]# tar xf nginx-0.30.0.tar.gz

# yaml文件在下载包中的位置：ingress-nginx-nginx-0.30.0/deploy/static/mandatory.yaml
[root@k8s-master ingress]# cp -a ingress-nginx-nginx-0.30.0/deploy/static/mandatory.yaml ./
```

```yaml
# yaml文件配置修改
[root@k8s-master ingress]# vim mandatory.yaml

………………
apiVersion: apps/v1
kind: DaemonSet   # 从Deployment改为DaemonSet
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  [[replicas]]: 1   # 注释掉
………………
      nodeSelector:
        kubernetes.io/hostname: k8s-master   # 修改处
      # 如下几行为新加行  作用【允许在master节点运行】
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
………………
          ports:
            - name: http
              containerPort: 80
              hostPort: 80    # 添加处【可在宿主机通过该端口访问Pod】
              protocol: TCP
            - name: https
              containerPort: 443
              hostPort: 443   # 添加处【可在宿主机通过该端口访问Pod】
              protocol: TCP
………………
```

```bash
[root@k8s-master ingress]# kubectl apply -f mandatory.yaml
namespace/ingress-nginx created
configmap/nginx-configuration created
configmap/tcp-services created
configmap/udp-services created
serviceaccount/nginx-ingress-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-role created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-role-nisa-binding created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
daemonset.apps/nginx-ingress-controller created
limitrange/ingress-nginx created

[root@k8s-master ingress]# kubectl get ds -n ingress-nginx -o wide
NAME                       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                       AGE     CONTAINERS                 IMAGES                                                                  SELECTOR
nginx-ingress-controller   1         1         1       1            1           kubernetes.io/hostname=k8s-master   9m47s   nginx-ingress-controller   quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.30.0   app.kubernetes.io/name=ingress-nginx,app.kubernetes.io/part-of=ingress-nginx

[root@k8s-master ingress]# kubectl get pod -n ingress-nginx -o wide
NAME                             READY   STATUS    RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES
nginx-ingress-controller-rrbh9   1/1     Running   0          9m55s   10.244.0.46   k8s-master   <none>           <none>
```

---



##### deply_service1的yaml信息

yaml文件

```bash
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
```

```yaml
[root@k8s-master ingress]# cat deply_service1.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy1
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      release: v1
  template:
    metadata:
      labels:
        app: myapp
        release: v1
        env: test
    spec:
      containers:
      - name: myapp
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-clusterip1
  namespace: default
spec:
  type: ClusterIP  # 默认类型
  selector:
    app: myapp
    release: v1
  ports:
  - name: http
    port: 80
    targetPort: 80
```

启动Deployment和Service

```bash
[root@k8s-master ingress]# kubectl apply -f deply_service1.yaml 
deployment.apps/myapp-deploy1 created
service/myapp-clusterip1 created
```

查看Deploy状态和信息

```bash
[root@k8s-master ingress]# kubectl get deploy -o wide
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
myapp-deploy1   3/3     3            3           28s   myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,release=v1

[root@k8s-master ingress]# kubectl get rs -o wide
NAME                       DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
myapp-deploy1-5695bb5658   3         3         3       30s   myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=5695bb5658,release=v1

[root@k8s-master ingress]# kubectl get pod -o wide --show-labels
NAME                             READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES   LABELS
myapp-deploy1-5695bb5658-n6548   1/1     Running   0          36s   10.244.2.144   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
myapp-deploy1-5695bb5658-rqcpb   1/1     Running   0          36s   10.244.2.143   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
myapp-deploy1-5695bb5658-vv6gm   1/1     Running   0          36s   10.244.3.200   k8s-node01   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
```

复制

curl访问pod

```bash
[root@k8s-master ingress]# curl 10.244.2.144
Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>

[root@k8s-master ingress]# curl 10.244.2.144/hostname.html
myapp-deploy1-5695bb5658-n6548

[root@k8s-master ingress]# curl 10.244.2.143/hostname.html
myapp-deploy1-5695bb5658-rqcpb

[root@k8s-master ingress]# curl 10.244.3.200/hostname.html
myapp-deploy1-5695bb5658-vv6gm
```

复制

查看Service状态和信息

```bash
[root@k8s-master ingress]# kubectl get svc -o wide
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE     SELECTOR
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP   19d     <none>
myapp-clusterip1   ClusterIP   10.104.146.14   <none>        80/TCP    5m38s   app=myapp,release=v1
```

复制

curl访问svc

```bash
[root@k8s-master ingress]# curl 10.104.146.14
Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>

[root@k8s-master ingress]# curl 10.104.146.14/hostname.html
myapp-deploy1-5695bb5658-n6548

[root@k8s-master ingress]# curl 10.104.146.14/hostname.html
myapp-deploy1-5695bb5658-vv6gm

[root@k8s-master ingress]# curl 10.104.146.14/hostname.html
myapp-deploy1-5695bb5658-rqcpb
```

---



##### deply_service2的yaml信息

yaml文件

```bash
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
```

```yaml
[root@k8s-master ingress]# cat deply_service2.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy2
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      release: v2
  template:
    metadata:
      labels:
        app: myapp
        release: v2
        env: test
    spec:
      containers:
      - name: myapp
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v2
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-clusterip2
  namespace: default
spec:
  type: ClusterIP  # 默认类型
  selector:
    app: myapp
    release: v2
  ports:
  - name: http
    port: 80
    targetPort: 80
```

启动Deployment和Service

```bash
[root@k8s-master ingress]# kubectl apply -f deply_service2.yaml 
deployment.apps/myapp-deploy2 created
service/myapp-clusterip2 created
```

查看Deploy状态和信息

```bash
[root@k8s-master ingress]# kubectl get deploy myapp-deploy2 -o wide
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
myapp-deploy2   3/3     3            3           9s    myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v2   app=myapp,release=v2

[root@k8s-master ingress]# kubectl get rs  -o wide
NAME                       DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES                                                      SELECTOR
myapp-deploy1-5695bb5658   3         3         3       7m23s   myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=5695bb5658,release=v1   # 之前创建的
myapp-deploy2-54f48f879b   3         3         3       15s     myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v2   app=myapp,pod-template-hash=54f48f879b,release=v2   # 当前deploy创建的

[root@k8s-master ingress]# kubectl get pod -o wide --show-labels -l "release=v2"
NAME                             READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES   LABELS
myapp-deploy2-54f48f879b-7pxwp   1/1     Running   0          25s   10.244.3.201   k8s-node01   <none>           <none>            app=myapp,env=test,pod-template-hash=54f48f879b,release=v2
myapp-deploy2-54f48f879b-lqlh2   1/1     Running   0          25s   10.244.2.146   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=54f48f879b,release=v2
myapp-deploy2-54f48f879b-pfvnn   1/1     Running   0          25s   10.244.2.145   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=54f48f879b,release=v2
```

复制

查看Service状态和信息

```bash
[root@k8s-master ingress]# kubectl get svc -o wide  
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE    SELECTOR
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP   19d    <none>
myapp-clusterip1   ClusterIP   10.104.146.14   <none>        80/TCP    8m9s   app=myapp,release=v1
myapp-clusterip2   ClusterIP   10.110.181.62   <none>        80/TCP    61s    app=myapp,release=v2
```

复制

curl访问svc

```bash
[root@k8s-master ingress]# curl 10.110.181.62
Hello MyApp | Version: v2 | <a href="hostname.html">Pod Name</a>

[root@k8s-master ingress]# curl 10.110.181.62/hostname.html
myapp-deploy2-54f48f879b-lqlh2

[root@k8s-master ingress]# curl 10.110.181.62/hostname.html
myapp-deploy2-54f48f879b-7pxwp

[root@k8s-master ingress]# curl 10.110.181.62/hostname.html
myapp-deploy2-54f48f879b-pfvnn
```

---



#### Ingress HTTP代理访问

yaml文件【由于自建的service在默认default名称空间，因此这里也是default名称空间】

```bash
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
```

```yaml
[root@k8s-master ingress]# cat ingress-http.yaml

apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-http
  namespace: default
spec:
  rules:
    - host: www.zhangtest.com
      http:
        paths:
        - path: /
          backend:
            serviceName: myapp-clusterip1
            servicePort: 80
    - host: blog.zhangtest.com
      http:
        paths:
        - path: /
          backend:
            serviceName: myapp-clusterip2
            servicePort: 80
```

复制

启动ingress http并查看状态

```bash
[root@k8s-master ingress]# kubectl apply -f ingress-http.yaml 
ingress.networking.k8s.io/nginx-http created

[root@k8s-master ingress]# kubectl get ingress -o wide
NAME         HOSTS                                  ADDRESS   PORTS   AGE
nginx-http   www.zhangtest.com,blog.zhangtest.com             80      9s
```

复制

查看nginx配置文件

```bash
[root@k8s-master ~]# kubectl get pod -A | grep 'ingre'
ingress-nginx          nginx-ingress-controller-rrbh9               1/1     Running   0          27m

[root@k8s-master ~]# kubectl exec -it -n ingress-nginx nginx-ingress-controller-rrbh9 bash
bash-5.0$ cat /etc/nginx/nginx.conf
…………
##### 可见server www.zhangtest.com 和 server blog.zhangtest.com的配置
```

---



##### 浏览器访问

hosts文件修改，添加如下信息

```java
文件位置：C:\WINDOWS\System32\drivers\etc\hosts
添加信息如下：

# K8S ingress学习
10.0.0.110  www.zhangtest.com  blog.zhangtest.com
```

浏览器访问[www.zhangtest.com](http://www.zhangtest.com/)

```java
http://www.zhangtest.com/
http://www.zhangtest.com/hostname.html
```

![img](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/1620-20220827114637982.png)

浏览器访问blog.zhangtest.com

```java
http://blog.zhangtest.com/
http://blog.zhangtest.com/hostname.html
```

![img](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/1620-20220827114638002.png)

当然：除了用浏览器访问外，也可以在Linux使用curl访问。前提是修改/etc/hosts文件，对上面的两个域名进行解析。

---



#### Ingress HTTPS代理访问

##### [SSL证书](https://cloud.tencent.com/product/ssl?from=10680)创建

```bash
[root@k8s-master cert]# pwd
/root/k8s_practice/ingress/cert

[root@k8s-master cert]# openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/C=CN/ST=BJ/L=BeiJing/O=BTC/OU=MOST/CN=zhang/emailAddress=ca@test.com"
Generating a 2048 bit RSA private key
......................................................+++
........................+++
writing new private key to 'tls.key'
-----

[root@k8s-master cert]# kubectl create secret tls tls-secret --key tls.key --cert tls.crt
secret/tls-secret created
```

---



##### 创建ingress https

yaml文件【由于自建的service在默认default名称空间，因此这里也是default名称空间】

```bash
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
```

```yaml
[root@k8s-master ingress]# cat ingress-https.yaml 
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-https
  namespace: default
spec:
  tls:
    - hosts:
      - www.zhangtest.com
      - blog.zhangtest.com
      secretName: tls-secret
  rules:
    - host: www.zhangtest.com
      http:
        paths:
        - path: /
          backend:
            serviceName: myapp-clusterip1
            servicePort: 80
    - host: blog.zhangtest.com
      http:
        paths:
        - path: /
          backend:
            serviceName: myapp-clusterip2
            servicePort: 80
```

复制

启动ingress https并查看状态

```bash
[root@k8s-master ingress]# kubectl apply -f ingress-https.yaml 
ingress.networking.k8s.io/nginx-https created

[root@k8s-master ingress]# kubectl get ingress -o wide
NAME          HOSTS                                  ADDRESS   PORTS     AGE
nginx-https   www.zhangtest.com,blog.zhangtest.com             80, 443   8s
```

---



##### 浏览器访问

hosts文件修改，添加如下信息

```java
文件位置：C:\WINDOWS\System32\drivers\etc\hosts
添加信息如下：

# K8S ingress学习
10.0.0.110  www.zhangtest.com  blog.zhangtest.com
```

复制

浏览器访问[www.zhangtest.com](http://www.zhangtest.com/)

```java
https://www.zhangtest.com/
https://www.zhangtest.com/hostname.html
```

![img](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/1620-20220827114638030.png)

浏览器访问blog.zhangtest.com

```java
https://blog.zhangtest.com/
https://blog.zhangtest.com/hostname.html
```

![img](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/1620-20220827114638047.png)

---



#### Ingress-Nginx实现BasicAuth认证

官网地址：

```java
https://kubernetes.github.io/ingress-nginx/examples/auth/basic/
```

准备工作

```bash
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress

[root@k8s-master ingress]# yum install -y httpd

[root@k8s-master ingress]# htpasswd -c auth foo
New password: [[输入密码]]
Re-type new password: [[重复输入的密码]]
Adding password for user foo   ##### 此时会生成一个 auth文件

[root@k8s-master ingress]# kubectl create secret generic basic-auth --from-file=auth
secret/basic-auth created
```

```yaml
[root@k8s-master ingress]# kubectl get secret basic-auth -o yaml

apiVersion: v1
data:
  auth: Zm9vOiRhcHIxJFpaSUJUMDZOJDVNZ3hxdkpFNWVRTi9NdnZCcVpHaC4K
kind: Secret
metadata:
  creationTimestamp: "2020-08-17T09:42:04Z"
  name: basic-auth
  namespace: default
  resourceVersion: "775573"
  selfLink: /api/v1/namespaces/default/secrets/basic-auth
  uid: eef0853b-a52b-4684-922a-817e4cd9e9ca
type: Opaque
```

ingress yaml文件

```bash
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
```

```yaml
[root@k8s-master ingress]# cat nginx_basicauth.yaml

apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-with-auth
  annotations:
    # type of authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    # name of the secret that contains the user/password definitions
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # message to display with an appropriate context why the authentication is required
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - foo'
spec:
  rules:
  - host: auth.zhangtest.com
    http:
      paths:
      - path: /
        backend:
          serviceName: myapp-clusterip1
          servicePort: 80
```

启动ingress并查看状态

```bash
[root@k8s-master ingress]# kubectl apply -f nginx_basicauth.yaml 
ingress.networking.k8s.io/ingress-with-auth created


[root@k8s-master ingress]# kubectl get ingress -o wide
NAME                HOSTS                                  ADDRESS   PORTS     AGE
ingress-with-auth   auth.zhangtest.com                               80        6s
```

---

##### 浏览器访问

hosts文件修改，添加如下信息

```bash
文件位置：C:\WINDOWS\System32\drivers\etc\hosts
添加信息如下：

# K8S ingress学习
10.0.0.110  www.zhangtest.com  blog.zhangtest.com auth.zhangtest.com
```

浏览器访问auth.zhangtest.com

```javascript
http://auth.zhangtest.com/
```

![img](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/1620-20220827114638073.png)

---



#### Ingress-Nginx实现Rewrite重写

官网地址：

```java
https://kubernetes.github.io/ingress-nginx/examples/rewrite/
```

重写可以使用以下注解控制：

| 名称                                           | 描述                                                         | 值     |
| :--------------------------------------------- | :----------------------------------------------------------- | :----- |
| nginx.ingress.kubernetes.io/rewrite-target     | 必须重定向的目标URL                                          | String |
| nginx.ingress.kubernetes.io/ssl-redirect       | 指示位置部分是否只能由SSL访问(当Ingress包含证书时，默认为True) | Bool   |
| nginx.ingress.kubernetes.io/force-ssl-redirect | 即使Ingress没有启用TLS，也强制重定向到HTTPS                  | Bool   |
| nginx.ingress.kubernetes.io/app-root           | 定义应用程序根目录，Controller在“/”上下文中必须重定向该根目录 | String |
| nginx.ingress.kubernetes.io/use-regex          | 指示Ingress上定义的路径是否使用正则表达式                    | Bool   |

ingress yaml文件

```bash
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
```

```yaml
[root@k8s-master ingress]# cat nginx_rewrite.yaml 

apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: https://www.baidu.com
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.zhangtest.com
    http:
      paths:
      - backend:
          serviceName: myapp-clusterip1
          servicePort: 80
```

启动ingress并查看状态

```bash
[root@k8s-master ingress]# kubectl apply -f nginx_rewrite.yaml 
ingress.networking.k8s.io/rewrite created

[root@k8s-master ingress]# kubectl get ingress -o wide
NAME                HOSTS                   ADDRESS          PORTS     AGE
rewrite             rewrite.zhangtest.com                    80        13s
```

---



##### 浏览器访问

hosts文件修改，添加如下信息

```bash
文件位置：C:\WINDOWS\System32\drivers\etc\hosts
添加信息如下：
# K8S ingress学习
10.0.0.110  www.zhangtest.com  blog.zhangtest.com auth.zhangtest.com  rewrite.zhangtest.com
```

浏览器访问rewrite.zhangtest.com

```java
http://rewrite.zhangtest.com/
```

![img](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/1620-20220827114638106.png)

之后，可见重定向到了[https://www.baidu.com 百度页面](https://www.baidu.xn--com-vy2fl66hqi5b8tb/)

---

#### 相关阅读

1、[k8s 官方 Ingress](https://kubernetes.io/zh/docs/concepts/services-networking/ingress/)

2、[Ingress-Nginx官网地址](https://kubernetes.github.io/ingress-nginx/)

3、[Ingress-Nginx GitHub地址](https://github.com/kubernetes/ingress-nginx)

4、[Ingress-Nginx实现BasicAuth认证](https://kubernetes.github.io/ingress-nginx/examples/auth/basic/)

5、[Ingress-Nginx实现Rewrite重写](https://kubernetes.github.io/ingress-nginx/examples/rewrite/)

---





 
