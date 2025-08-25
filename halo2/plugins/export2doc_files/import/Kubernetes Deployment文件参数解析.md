###  Kubernetes Deployment文件参数解析

---

#### 示例1：

```yaml
apiVersion: v1 [[必选，版本号，例如v1]]
kind: Pod [[必选，Pod]]
metadata: [[必选，元数据]]
name: string [[必选，Pod名称]]
namespace: string [[必选，Pod所属的命名空间]]
labels: [[自定义标签]]
- name: string [[自定义标签名字]]
annotations: [[自定义注释列表]]
- name: string
spec: [[必选，Pod中容器的详细定义]]
containers: [[必选，Pod中容器列表]]

    name: string [[必选，容器名称]]
    image: string [[必选，容器的镜像名称]]
    imagePullPolicy: [Always | Never | IfNotPresent] [[获取镜像的策略]] Alawys表示下载镜像 IfnotPresent表示优先使用本地镜像，否则下载镜像，Nerver表示仅使用本地镜像
    command: [string] [[容器的启动命令列表，如不指定，使用打包时使用的启动命令]]
    args: [string] [[容器的启动命令参数列表]]
    workingDir: string [[容器的工作目录]]
    volumeMounts: [[挂载到容器内部的存储卷配置]]
        name: string [[引用pod定义的共享存储卷的名称，需用volumes]][]部分定义的的卷名
        mountPath: string [[存储卷在容器内mount的绝对路径，应少于512字符]]
        readOnly: boolean [[是否为只读模式]]
        ports: [[需要暴露的端口库号列表]]
        name: string [[端口号名称]]
        containerPort: int [[容器需要监听的端口号]]
        hostPort: int [[容器所在主机需要监听的端口号，默认与Container相同]]
        protocol: string [[端口协议，支持TCP和UDP，默认TCP]]
        env: [[容器运行前需设置的环境变量列表]]
        name: string [[环境变量名称]]
        value: string [[环境变量的值]]
        resources: [[资源限制和请求的设置]]
        limits: [[资源限制的设置]]
        cpu: string [[Cpu的限制，单位为core数，将用于docker]] run --cpu-shares参数
        memory: string [[内存限制，单位可以为Mib/Gib，将用于docker]] run --memory参数
        requests: [[资源请求的设置]]
        cpu: string [[Cpu请求，容器启动的初始可用数量]]
        memory: string [[内存请求，容器启动的初始可用数量]]
        livenessProbe: [[对Pod内每个容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可]]
        exec: [[对Pod容器内检查方式设置为exec方式]]
        command: [string] [[exec方式需要制定的命令或脚本]]
        httpGet: [[对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port]]
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
            name: string
            value: string
            tcpSocket: [[对Pod内个容器健康检查方式设置为tcpSocket方式]]
            port: number
            initialDelaySeconds: 0 [[容器启动完成后首次探测的时间，单位为秒]]
            timeoutSeconds: 0 [[对容器健康检查探测等待响应的超时时间，单位秒，默认1秒]]
            periodSeconds: 0 [[对容器监控检查的定期探测时间设置，单位秒，默认10秒一次]]
           successThreshold: 0
           failureThreshold: 0
           securityContext:
           privileged:false
           restartPolicy: [Always | Never | OnFailure]#Pod的重启策略，Always表示一旦不管以何种方式终止运行，kubelet都将重启，OnFailure表示只有Pod以非0退出码退出才重启，Nerver表示不再重启该Pod
           nodeSelector: obeject [[设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定]]
           imagePullSecrets: [[Pull镜像时使用的secret名称，以key：secretkey格式指定]]
        name: string
        hostNetwork:false [[是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络]]
        volumes: [[在该pod上定义共享存储卷列表]]
        name: string [[共享存储卷名称]] （volumes类型有很多种）
        emptyDir: {} [[类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值]]
        hostPath: string [[类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录]]
        path: string [[Pod所在宿主机的目录，将被用于同期中mount的目录]]
        secret: [[类型为secret的存储卷，挂载集群与定义的secre对象到容器内部]]
        scretname: string
        items:
            key: string
            path: string
            configMap: [[类型为configMap的存储卷，挂载预定义的configMap对象到容器内部]]
            name: string
            items:
            key: string
            path: string
```

#### 示例2：

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata: <Object>
spec: <Object>
  minReadySeconds: <integer> [[设置pod准备就绪的最小秒数]]
  paused: <boolean> [[表示部署已暂停并且deploy控制器不会处理该部署]]
  progressDeadlineSeconds: <integer>
  strategy: <Object> [[将现有pod替换为新pod的部署策略]]
    rollingUpdate: <Object> [[滚动更新配置参数，仅当类型为RollingUpdate]]
      maxSurge: <string> [[滚动更新过程产生的最大pod数量，可以是个数，也可以是百分比]]
      maxUnavailable: <string> #
    type: <string> [[部署类型，Recreate，RollingUpdate]]
  replicas: <integer> [[pods的副本数量]]
  selector: <Object> [[pod标签选择器，匹配pod标签，默认使用pods的标签]]
    matchLabels: <map[string]string> 
      key1: value1
      key2: value2
    matchExpressions: <[]Object>
      operator: <string> -required- [[设定标签键与一组值的关系，In]], NotIn, Exists and DoesNotExist
      key: <string> -required-
      values: <[]string>   
  revisionHistoryLimit: <integer> [[设置保留的历史版本个数，默认是10]]
  rollbackTo: <Object> 
    revision: <integer> [[设置回滚的版本，设置为0则回滚到上一个版本]]
  template: <Object> -required-
    metadata:
    spec:
      containers: <[]Object> [[容器配置]]
      - name: <string> -required- [[容器名、DNS_LABEL]]
        image: <string> [[镜像]]
        imagePullPolicy: <string> [[镜像拉取策略，Always、Never、IfNotPresent]]
        ports: <[]Object>
        - name: [[定义端口名]]
          containerPort: [[容器暴露的端口]]
          protocol: TCP [[或UDP]]
        volumeMounts: <[]Object>
        - name: <string> -required- [[设置卷名称]]
          mountPath: <string> -required- [[设置需要挂载容器内的路径]]
          readOnly: <boolean> [[设置是否只读]]
        livenessProbe: <Object> [[就绪探测]]
          exec: 
            command: <[]string>
          httpGet:
            port: <string> -required-
            path: <string>
            host: <string>
            httpHeaders: <[]Object>
              name: <string> -required-
              value: <string> -required-
            scheme: <string> 
          initialDelaySeconds: <integer> [[设置多少秒后开始探测]]
          failureThreshold: <integer> [[设置连续探测多少次失败后，标记为失败，默认三次]]
          successThreshold: <integer> [[设置失败后探测的最小连续成功次数，默认为1]]
          timeoutSeconds: <integer> [[设置探测超时的秒数，默认1s]]
          periodSeconds: <integer> [[设置执行探测的频率（以秒为单位），默认1s]]
          tcpSocket: <Object> [[TCPSocket指定涉及TCP端口的操作]]
            port: <string> -required- [[容器暴露的端口]]
            host: <string> [[默认pod的IP]]
        readinessProbe: <Object> [[同livenessProbe]]
        resources: <Object> [[资源配置]]
          requests: <map[string]string> [[最小资源配置]]
            memory: "1024Mi"
            cpu: "500m" [[500m代表0]].5CPU
          limits: <map[string]string> [[最大资源配置]]
            memory:
            cpu:         
      volumes: <[]Object> [[数据卷配置]]
      - name: <string> -required- [[设置卷名称]],与volumeMounts名称对应
        hostPath: <Object> [[设置挂载宿主机路径]]
          path: <string> -required- 
          type: <string> [[类型：DirectoryOrCreate、Directory、FileOrCreate、File、Socket、CharDevice、BlockDevice]]
      - name: nfs
        nfs: <Object> [[设置NFS服务器]]
          server: <string> -required- [[设置NFS服务器地址]]
          path: <string> -required- [[设置NFS服务器路径]]
          readOnly: <boolean> [[设置是否只读]]
      - name: configmap
        configMap: 
          name: <string> [[configmap名称]]
          defaultMode: <integer> [[权限设置0]]~0777，默认0664
          optional: <boolean> [[指定是否必须定义configmap或其keys]]
          items: <[]Object>
          - key: <string> -required-
            path: <string> -required-
            mode: <integer>
      restartPolicy: <string> [[重启策略，Always、OnFailure、Never]]
      nodeName: <string>
      nodeSelector: <map[string]string>
      imagePullSecrets: <[]Object>
      hostname: <string>
      hostPID: <boolean>
status: <Object>
```

#### 示例3：

**Deployment 使用**

[Kubernetes](https://so.csdn.net/so/search?q=Kubernetes&spm=1001.2101.3001.7020)提供了一种更加简单的更新RC和Pod的机制，叫做Deployment。通过在Deployment中描述你所期望的集群状态，Deployment Controller会将现在的集群状态在一个可控的速度下逐步更新成你所期望的集群状态。Deployment主要职责同样是为了保证pod的数量和健康，90%的功能与Replication Controller完全一样，可以看做新一代的Replication Controller。但是，它又具备了Replication Controller之外的新特性：Replication Controller全部功能：Deployment继承了上面描述的Replication Controller全部功能。

- 事件和状态查看：可以查看Deployment的升级详细进度和状态。
- 回滚：当升级pod镜像或者相关参数的时候发现问题，可以使用回滚操作回滚到上一个稳定的版本或者指定的版本。
- 版本记录: 每一次对Deployment的操作，都能保存下来，给予后续可能的回滚使用。
- 暂停和启动：对于每一次升级，都能够随时暂停和启动。
- 多种升级方案：Recreate----删除所有已存在的pod,重新创建新的; RollingUpdate----滚动升级，逐步替换的策略，同时滚动升级时，支持更多的附加参数，例如设置最大不可用pod数量，最小升级间隔时间等等。

```yaml
[[创建命令：kubectl]] create -f deployment.yaml --record
[[使用rollout]] history命令，查看Deployment的历史信息：kubectl rollout history deployment cango-demo
[[使用rollout]] undo回滚到上一版本: kubectl rollout undo deployment cango-demo
[[使用]]–to-revision可以回滚到指定版本:  kubectl rollout undo deployment hello-deployment --to-revision=2
```



**ConfigMap 配置**

在几乎所有的应用开发中，都会涉及到配置文件的变更，比如说在web的程序中，需要连接数据库，缓存甚至是队列等等。
一些大公司专门开发了自己的一套配置管理中心，kubernetes也提供了自己的一套方案，即ConfigMap。kubernetes通过ConfigMap来实现对容器中应用的配置管理。

```yaml
apiVersion: v1   [[接口版本]]
kind: ConfigMap  [[接口类型]]
metadata:
  name: special-config  [[Config名称]]
  namespace: cango-prd  [[命名空间]]
data:                   [[键值对]]      
  special.how: very    
  special.type: charm
 
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-volume-config
  namespace: cango-prd
data:
  backup-script: | xxxx自定义文件内容
  log-script: | xxxx自定义文件内容
```



**StorageClass**

Kubernetes 集群存储 PV 支持 Static 静态配置以及 Dynamic 动态配置，动态卷配置 (Dynamic provisioning) 可以根据需要动态的创建存储卷。

- 静态配置方式，集群管理员必须手动调用云/存储服务提供商的接口来配置新的固定大小的 Image 存储卷，然后创建 PV 对象以在 Kubernetes 中请求分配使用它们。
- \#通过动态卷配置，能自动化完成以上两步骤，它无须集群管理员预先配置存储资源，而是使用 StorageClass 对象指定的供应商来动态配置存储资源。

Example: 定义Ceph作为PV存储池进行分配，每个云或存储厂商品牌配置参数都不一样，但是调用方式都是一致的

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rbd
provisioner: kubernetes.io/rbd
parameters:
  monitors: 10.142.21.21:6789,10.142.21.22:6789,10.142.21.23:6789
  adminId: admin
  adminSecretName: ceph-secret-admin
  adminSecretNamespace: kube-system
  pool: rbd
  userId: admin
  userSecretName: ceph-secret-admin
  fsType: ext4
  imageFormat: "1"
```



\#创建从StorageClass的关联的Ceph PV存储rdb中申请一块1G大小的PVC命名为rdb-pvc1

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: rbd-pvc1
  namespace: kube-system
spec:
  storageClassName: rbd
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi   
```



**Deployment部署文件详解**

```yaml
apiVersion: extensions/v1beta1   [[接口版本]]
kind: Deployment                 [[接口类型]]
metadata:
  name: cango-demo               [[Deployment名称]]
  namespace: cango-prd           [[命名空间]]
  labels:
    app: cango-demo              [[标签]]
spec:
  replicas: 3
   strategy:
    rollingUpdate:  ##由于replicas为3,则整个升级,pod个数在2-4个之间
      maxSurge: 1      [[滚动升级时会先启动1个pod]]
      maxUnavailable: 1 [[滚动升级时允许的最大Unavailable的pod个数]]
  template:         
    metadata:
      labels:
        app: cango-demo  [[模板名称必填]]
    sepc: [[定义容器模板，该模板可以包含多个容器]]
      containers:                                                                   
        - name: cango-demo                                                           [[镜像名称]]
          image: swr.cn-east-2.myhuaweicloud.com/cango-prd/cango-demo:0.0.1-SNAPSHOT [[镜像地址]]
          command: [ "/bin/sh","-c","cat /etc/config/path/to/special-key" ]    [[启动命令]]
          args:                                                                [[启动参数]]
            - '-storage.local.retention=$(STORAGE_RETENTION)'
            - '-storage.local.memory-chunks=$(STORAGE_MEMORY_CHUNKS)'
            - '-config.file=/etc/prometheus/prometheus.yml'
            - '-alertmanager.url=http://alertmanager:9093/alertmanager'
            - '-web.external-url=$(EXTERNAL_URL)'
    [[如果command和args均没有写，那么用Docker默认的配置。]]
    [[如果command写了，但args没有写，那么Docker默认的配置会被忽略而且仅仅执行]].yaml文件的command（不带任何参数的）。
    [[如果command没写，但args写了，那么Docker默认配置的ENTRYPOINT的命令行会被执行，但是调用的参数是]].yaml中的args。
    [[如果如果command和args都写了，那么Docker默认的配置被忽略，使用]].yaml的配置。
          imagePullPolicy: IfNotPresent  [[如果不存在则拉取]]
          livenessProbe:       [[表示container是否处于live状态。如果LivenessProbe失败，LivenessProbe将会通知kubelet对应的container不健康了。随后kubelet将kill掉container，并根据RestarPolicy进行进一步的操作。默认情况下LivenessProbe在第一次检测之前初始化值为Success，如果container没有提供LivenessProbe，则也认为是Success；]]
            httpGet:
              path: /health [[如果没有心跳检测接口就为/]]
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 60 ##启动后延时多久开始运行检测
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 5
            readinessProbe:
          readinessProbe:
            httpGet:
              path: /health [[如果没有心跳检测接口就为/]]
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 30 ##启动后延时多久开始运行检测
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 5
          resources:              ##CPU内存限制
            requests:
              cpu: 2
              memory: 2048Mi
            limits:
              cpu: 2
              memory: 2048Mi
          env:                    ##通过环境变量的方式，直接传递pod=自定义Linux OS环境变量
            - name: LOCAL_KEY     [[本地Key]]
              value: value
            - name: CONFIG_MAP_KEY  [[局策略可使用configMap的配置Key，]]
              valueFrom:
                configMapKeyRef:
                  name: special-config   [[configmap中找到name为special-config]]
                  key: special.type      [[找到name为special-config里data下的key]]
          ports:
            - name: http
              containerPort: 8080 [[对service暴露端口]]
          volumeMounts:     [[挂载volumes中定义的磁盘]]
          - name: log-cache
            mount: /tmp/log
          - name: sdb       [[普通用法，该卷跟随容器销毁，挂载一个目录]]
            mountPath: /data/media    
          - name: nfs-client-root    [[直接挂载硬盘方法，如挂载下面的nfs目录到/mnt/nfs]]
            mountPath: /mnt/nfs
          - name: example-volume-config  [[高级用法第1种，将ConfigMap的log-script]],backup-script分别挂载到/etc/config目录下的一个相对路径path/to/...下，如果存在同名文件，直接覆盖。
            mountPath: /etc/config       
          - name: rbd-pvc                [[高级用法第2中，挂载PVC]](PresistentVolumeClaim)
 
[[使用volume将ConfigMap作为文件或目录直接挂载，其中每一个key-value键值对都会生成一个文件，key为文件名，value为内容，]]
  volumes:  # 定义磁盘给上面volumeMounts挂载
  - name: log-cache
    emptyDir: {}
  - name: sdb  [[挂载宿主机上面的目录]]
    hostPath:
      path: /any/path/it/will/be/replaced
  - name: example-volume-config  # 供ConfigMap文件内容到指定路径使用
    configMap:
      name: example-volume-config  [[ConfigMap中名称]]
      items:
      - key: log-script           [[ConfigMap中的Key]]
        path: path/to/log-script  [[指定目录下的一个相对路径path/to/log-script]]
      - key: backup-script        [[ConfigMap中的Key]]
        path: path/to/backup-script  [[指定目录下的一个相对路径path/to/backup-script]]
  - name: nfs-client-root         [[供挂载NFS存储类型]]
    nfs:
      server: 10.42.0.55          [[NFS服务器地址]]
      path: /opt/public           [[showmount]] -e 看一下路径
  - name: rbd-pvc                 [[挂载PVC磁盘]]
    persistentVolumeClaim:
      claimName: rbd-pvc1         [[挂载已经申请的pvc磁盘]]
```



参考文献：

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

https://www.jianshu.com/p/6bc8e0ae65d1

http://ju.outofmemory.cn/entry/208362

---

#### 示例4：

因为kubernetes在v1.18.0版本之后废弃了–replicas 参数，因此这里就不涉及命令行方式创建deployment的了，而推荐使用yaml的方式

一、使用配置文件创建deployment
1.1 编写配置文件
如下，创建deployment-nginx.yaml文件，编辑内容如下，即创建一个dev命名空间，并在dev命名空间中创建三个nginx的pod，并且创建一个deployment控制器

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:1.17.1
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
```

1.2 创建资源

使用如下命令创建资源

```bash
[root@master demo]# kubectl apply -f deployment-nginx.yaml
namespace/dev created
deployment.apps/nginx created
[root@master demo]#
```

1.3 查看创建的资源

如下，同时查看deployment和pod

```bash
[root@master demo]# kubectl get deployment,pod -n dev
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           68s

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-66ffc897cf-787xt   1/1     Running   0          68s
pod/nginx-66ffc897cf-cxxjs   1/1     Running   0          68s
pod/nginx-66ffc897cf-k88qt   1/1     Running   0          68s
[root@master demo]#
```

1.4 删除其中一个pod

如下删除其中一个pod

```bash
[root@master demo]# kubectl delete  pod/nginx-66ffc897cf-787xt -n dev
pod "nginx-66ffc897cf-787xt" deleted
[root@master demo]#
```

再次查看deployment和pod，如下，可以发现此时还是三个pod，仔细观察可以发现，上面的删除操作确实成功了，只不过这里又增加了一个新的pod，这就是deployment控制器在发挥作用，即当一个pod被删除后，deployment控制器会自动的创建一个新的pod来满足要求

```bash
[root@master demo]# kubectl get deployment,pod -n dev
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           3m21s

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-66ffc897cf-cxxjs   1/1     Running   0          3m21s
pod/nginx-66ffc897cf-k88qt   1/1     Running   0          3m21s
pod/nginx-66ffc897cf-ztpbm   1/1     Running   0          38s
[root@master demo]#
```

1.5 删除资源

使用如下命令删除资源

```bash
[root@master demo]# kubectl delete -f deployment-nginx.yaml
namespace "dev" deleted
deployment.apps "nginx" deleted
[root@master demo]#
```

---

#### 示例5：

工作负载控制器是什么
工作负载控制器(Workload Controllers)是k8s的一个抽象概念，用于更高层次对象，部署和管理pod
常用工作负载控制器：

Deployment：无状态应用部署
StatefulSet：有状态应用部署
DaemonSet：确保所有Node运行同一个Pod
Job：一次性任务
Cronjob：定时任务
控制器的作用：

管理Pod对象
使用标签与Pod关联
控制器实现了Pod 的运维，例如滚动更新、伸缩、副本管理、维护Pod状态等
直接去创建pod有很多功能使用不了的，都是需要控制器去来管理，所以实际生产环境Pod用作测试比较多

Deployment
Deployment可以管理Pod和ReplicaSet，具有上线部署、副本设定、滚动升级、回滚等功能，他的应用场景主要在网站、API、微服务等

Deployment使用流程如下

第一步：部署镜像
部署使用的镜像，镜像来源与两个位置(docker hub、私有仓库)
在这里只是简单的测试一下Deployment的使用流程，以nginx版本为例去进行测试，生产环境中的部署方式类似

```bash
# 部署nginx镜像，尽量使用yaml去部署，yaml可以通过kubectl create命令去生成后修改，也可以自己去手写
# 生成一个deployment的yaml，然后去部署nginx
[root@k8s-master ~]# kubectl create deployment web --image=nginx --dry-run=client -o yaml > deployment.yaml
kubectl apply -f deployment.yaml
# 生成一个service的yaml
[root@k8s-master ~]# kubectl expose deployment web --port=80 --target-port=80 --type=NodePort --dry-run=client -o yaml > services.yaml
[root@k8s-master ~]# kubectl apply -f services.yaml
```

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/1713e1b97cec4a4ea3a5198a4a4898e2.png)

第二步：滚动升级
滚动发布是指每次只升级一个或多个服务，升级完成后加入生产环境，不断执行这个过程，直到集群中的全部旧版本升级新版本
应用的升级有三种方式

修改yaml，然后kubectl apply -f进行升级
使用命令去升级，kubectl set image deployment/web nginx=nginx:1.18
使用系统编辑器打开，kubectl edit deployment/web
第一种方式，修改yaml文件进行升级

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/4456ac708f5241f5b3d389f7059b0488.png)

问nginx页面，nginx版本号变更

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/323445c7f5e6412483ba2ddd8fd07aa6.png)

第二种方式，使用命令升级

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/75998cb833ae465bb4a916ec3bce2ee1.png)

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/33df0f236db2477fa17ddf0b11fd5f3a.png)

第三种方式，使用系统的编辑器打开修改

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/a4f7fb84977743ca9813619423667c3b.png)

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/0eb6a026fc0d45bcafaa2f04ea5f6748.png)

深入理解下k8s是如何实现滚动升级的
master另开两个终端持续查看在滚动更新时情况

先将副本数改为3，然后重新更新一下

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/6f5b1c57df964a30af9c82bee410ffef.png)

```bash
# 一台机器查看nginx请求头信息，这样子可以看到nginx版本号为多少
[root@k8s-master ~]# for i in {1..1000};do curl -I 10.96.93.68 ;sleep 1;don
```

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/511324e86ec8444482833f4e56c93ccf.png)

```bash
# 一台机器查看副本数状态
[root@k8s-master ~]# kubectl get replicaset -w
```

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/9b73bcd17d7b413792be4b6e4e6fc82d.png)



开始更新版本

```bash
[root@k8s-master ~]# kubectl set image deployment web nginx=nginx:1.15
```

更新好后他会一个一个去替换掉

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/2cb9d4ef35274b23b7aaf8f46196f829.png)

还可以拿这条命令去查看升级情况

![在这里插入图片描述](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/f3aa8178a8a043609e6105d9847762cd.png)

滚动更新策略
maxSurge：滚动更新过程中最大Pod副本数，确保在更新时启动的Pod数量比期望(replicas)Pod数量最大多出25%
maxUnavailable：滚动更新过程中最大不可用Pod副本数，确保在更新时最大25%Pod数量不可用，即确保75%Pod数量是可用状态
其实默认就是25%的

```yaml
# 案例
spec:
  replicas: 3
  revisionHistoryLimit: 10 # RS历史版本保存数量
  selector:
    matchLabels:
      app: web
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
```

第三步：水平扩容
他是通过replicas参数来控制Pod数量的(也是由ReplicaSet来控制的)，有两种方法

修改yaml里的replicas值，在apply
使用命令，kubectl scale deployment web --replicas=10
第四步：回滚
发布的版本不合适需要退回到上个版本或者指定版本，可以使用以下命令来实现

kubectl rollout history deployment web # 查看历史发布版本，web是项目所在deployment的名称，我这里的名称就为web
kubectl rollout undo deployment web # 回滚到上一个版本
kubectl rollout undo deployment web --to-revison=2 # 回滚历史指定版本
可以使用以下命令来查看历史版本和镜像版本的对应关系，不过信息太多需要使用正则语句去匹配出来

kubectl describe rs | grep -C 9 revision [[找到关键字revision后向下查询9行]]
第五步：应用下线
删除即可

kubectl delete deploy web
kubectl delete svc web
ReplicaSet控制器
ReplicaSet：副本集，主要维护Pod副本数量，不断对比当前Pod数量与期望Pod数量
ReplicaSet用途：Deployment每次发布都会创建一个RS作为记录，用于实现滚动升级和回滚

kubectl get rs # 查看RS记录
kubectl rollout history deployment web # 版本对应RS记录
DaemonSet
DaemonSet应用场景：网络插件(kube-proxy、calico)、其他Agent
DaemonSet功能：

在每一个Node上运行一个Pod
新加入的Node也同样会自动运行一个Pod
DaemonSet和deployment的使用方式类似，只是不支持副本集，因为默认每个节点都有一个

Job
Job分为普通任务(Job)和定时任务(CronJob)

一次性执行
应用场景：离线数据处理，视频解码等业务

```yaml
# 案例
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restarPollicy: Never
```

CronJob

CronJob用于实现定时任务，像Linux的Crontab一样

- 定时任务

应用场景：通知，备份

```yaml
# 案例
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"  # 每分钟运行一次
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello hahaha
          restartPolicy: OnFailure
```

---

#### 示例6：

```bash
通过kubectl创建deployment
# kubectl create -f deployment.yaml --record
–record参数，使用此参数将记录后续创建对象的操作，方便管理与问题追溯

查看deployment具体信息
# kubectl describe deployment frontend

通过deployment修改Pod副本数量（需要修改yaml文件的spec.replicas字段到目标值，然后替换旧的yaml文件）
# kubectl edit deployment hello-deployment

使用rollout history命令，查看Deployment的历史信息
kubectl rollout history deployment hello-deployment

使用Deployment可以回滚到上一版本，但要加上–revision参数，指定版本号
kubectl rollout history deployment hello-deployment --revision=2

使用rollout undo回滚到上一版本
kubectl rollout undo deployment hello-deployment 

使用–to-revision可以回滚到指定版本
kubectl rollout undo deployment hello-deployment --to-revision=2
```

