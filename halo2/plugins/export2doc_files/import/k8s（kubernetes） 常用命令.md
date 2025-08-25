# k8s（kubernetes） 常用命令

**查看所有** **pod** **列表， -n** **后跟** **命名空间，** **查看指定的命名空间**

```bash
kubectl get pod 
kubectl get pod -n kube-system    [[查看指定命名空间的pod]]
kubectl get pod -o wide    [[查看更详细的信息，比如pod所在节点]]
kubectl get pod --show-labels    [[获取pod并查看pod的标签]]
```

**查看** **RC** **和** **service** **列表，** **-o wide** **查看详细信息**

```bash
kubectl get rc,svc
kubectl get pod,svc -o wide
kubectl get pod <pod name> -o yaml
```

**显示** **Node** **的详细信息**

```bash
kubectl describe node 192.168.0.212 [[可以跟Node]] IP或者主机名
```

**显示** **Pod** **的详细信息，** **特别是查看** **pod** **无法创建的时候的日志**

```bash
kubectl describe pod <pod-name>
eg:
kubectl describe pod redis-master-tqds9
```

**根据** **yaml** **创建资源， apply** **可以重复执行，create** **不行**

```bash
kubectl create -f pod.yaml
kubectl apply -f pod.yaml
```

**基于 pod.yaml** **定义的名称删除指定资源**

```bash
kubectl delete -f pod.yaml 
```

**删除所有包含某个** **label** **的pod** **和** **service**

```bash
kubectl delete pod,svc -l name=<label-name>
```

**删除默认命名空间下的所有** **Pod**

```bash
kubectl delete pod --all
```

**执行** **pod** **命令**

```bash
kubectl exec <pod-name> -- date
kubectl exec <pod-name> -- bash
kubectl exec <pod-name> -- ping 10.24.51.9
```

**通过bash获得** **pod** **中某个**[**容器**](https://cloud.tencent.com/product/tke?from=10680)**的TTY，相当于登录容器**

```bash
kubectl exec -it <pod-name> -c <container-name> -- bash
eg:
kubectl exec -it redis-master-cln81 -- bash
```

**查看容器的日志**

```bash
kubectl logs <pod-name>
kubectl logs -f <pod-name> # 实时查看日志
kubectl log  <pod-name>  -c <container_name> # 若 pod 只有一个容器，可以不加 -c 

kubectl logs -l app=frontend # 返回所有标记为 app=frontend 的 pod 的合并日志。
```

**查看节点标签**

```bash
kubectl get node --show-labels
```

**重启** **pod**

```bash
kubectl get pod <POD名称> -n <NAMESPACE名称> -o yaml | kubectl replace --force -f -
```

**创建命令**

```bash
kubectl apply -f ./my-manifest.yaml           # 创建资源
kubectl apply -f ./my1.yaml -f ./my2.yaml     # 使用多个文件创建
kubectl apply -f ./dir                        # 基于目录下的所有清单文件创建资源
kubectl apply -f https://git.io/vPieo         # 从 URL 中创建资源
kubectl create deployment nginx --image=nginx # 启动单实例 nginx
kubectl explain pods,svc                      # 获取 pod 清单的文档说明

# 从标准输入创建多个 YAML 对象
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000000"
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep-less
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000"
EOF

# 创建有多个 key 的 Secret
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: $(echo -n "s33msi4" | base64 -w0)
  username: $(echo -n "jane" | base64 -w0)
EOF
```

**查看和查找资源**

```bash
# get 命令的基本输出
kubectl get services                          # 列出当前命名空间下的所有 services
kubectl get pods --all-namespaces             # 列出所有命名空间下的全部的 Pods
kubectl get pods -o wide                      # 列出当前命名空间下的全部 Pods，并显示更详细的信息
kubectl get deployment my-dep                 # 列出某个特定的 Deployment
kubectl get pods                              # 列出当前命名空间下的全部 Pods
kubectl get pod my-pod -o yaml                # 获取一个 pod 的 YAML

# describe 命令的详细输出
kubectl describe nodes my-node
kubectl describe pods my-pod

# 列出当前名字空间下所有 Services，按名称排序
kubectl get services --sort-by=.metadata.name

# 列出 Pods，按重启次数排序
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# 列举所有 PV 持久卷，按容量排序
kubectl get pv --sort-by=.spec.capacity.storage

# 获取包含 app=cassandra 标签的所有 Pods 的 version 标签
kubectl get pods --selector=app=cassandra -o \
  jsonpath='{.items[*].metadata.labels.version}'

# 获取所有工作节点（使用选择器以排除标签名称为 'node-role.kubernetes.io/master' 的结果）
kubectl get node --selector='!node-role.kubernetes.io/master'

# 获取当前命名空间中正在运行的 Pods
kubectl get pods --field-selector=status.phase=Running

# 获取全部节点的 ExternalIP 地址
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# 列出属于某个特定 RC 的 Pods 的名称
# 在转换对于 jsonpath 过于复杂的场合，"jq" 命令很有用；可以在 https://stedolan.github.io/jq/ 找到它。
sel=${$(kubectl get rc my-rc --output=json | jq -j '.spec.selector | to_entries | .[] | "\(.key)=\(.value),"')%?}
echo $(kubectl get pods --selector=$sel --output=jsonpath={.items..metadata.name})

# 显示所有 Pods 的标签（或任何其他支持标签的 Kubernetes 对象）
kubectl get pods --show-labels

# 检查哪些节点处于就绪状态
JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"

# 列出被一个 Pod 使用的全部 Secret
kubectl get pods -o json | jq '.items[].spec.containers[].env[]?.valueFrom.secretKeyRef.name' | grep -v null | sort | uniq
```

```bash
# 列举所有 Pods 中初始化容器的容器 ID（containerID）
# Helpful when cleaning up stopped containers, while avoiding removal of initContainers.
kubectl get pods --all-namespaces -o jsonpath='{range .items[*].status.initContainerStatuses[*]}{.containerID}{"\n"}{end}' | cut -d/ -f3

# 列出事件（Events），按时间戳排序
kubectl get events --sort-by=.metadata.creationTimestamp

# 比较当前的集群状态和假定某清单被应用之后的集群状态
kubectl diff -f ./my-manifest.yaml
```

**更新资源**

```bash
kubectl set image deployment/frontend www=image:v2               # 滚动更新 "frontend" Deployment 的 "www" 容器镜像
kubectl rollout history deployment/frontend                      # 检查 Deployment 的历史记录，包括版本 
kubectl rollout undo deployment/frontend                         # 回滚到上次部署版本
kubectl rollout undo deployment/frontend --to-revision=2         # 回滚到特定部署版本
kubectl rollout status -w deployment/frontend                    # 监视 "frontend" Deployment 的滚动升级状态直到完成
kubectl rollout restart deployment/frontend                      # 轮替重启 "frontend" Deployment

cat pod.json | kubectl replace -f -                              # 通过传入到标准输入的 JSON 来替换 Pod

# 强制替换，删除后重建资源。会导致服务不可用。
kubectl replace --force -f ./pod.json

# 为多副本的 nginx 创建服务，使用 80 端口提供服务，连接到容器的 8000 端口。
kubectl expose rc nginx --port=80 --target-port=8000

# 将某单容器 Pod 的镜像版本（标签）更新到 v4
kubectl get pod mypod -o yaml | sed 's/\(image: myimage\):.*$/\1:v4/' | kubectl replace -f -

kubectl label pods my-pod new-label=awesome                      # 添加标签
kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq       # 添加注解
kubectl autoscale deployment foo --min=2 --max=10                # 对 "foo" Deployment 自动伸缩容
```

**部分更新资源**

```bash
# 部分更新某节点
kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}' 

# 更新容器的镜像；spec.containers[*].name 是必须的。因为它是一个合并性质的主键。
kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"kubernetes-serve-hostname","image":"new image"}]}}'

# 使用带位置数组的 JSON patch 更新容器的镜像
kubectl patch pod valid-pod --type='json' -p='[{"op": "replace", "path": "/spec/containers/0/image", "value":"new image"}]'

# 使用带位置数组的 JSON patch 禁用某 Deployment 的 livenessProbe
kubectl patch deployment valid-deployment  --type json   -p='[{"op": "remove", "path": "/spec/template/spec/containers/0/livenessProbe"}]'

# 在带位置数组中添加元素 
kubectl patch sa default --type='json' -p='[{"op": "add", "path": "/secrets/1", "value": {"name": "whatever" } }]'
```

**删除资源**

```bash
kubectl delete -f ./pod.json                                              # 删除在 pod.json 中指定的类型和名称的 Pod
kubectl delete pod,service baz foo                                        # 删除名称为 "baz" 和 "foo" 的 Pod 和服务
kubectl delete pods,services -l name=myLabel                              # 删除包含 name=myLabel 标签的 pods 和服务
kubectl delete pods,services -l name=myLabel --include-uninitialized      # 删除包含 label name=myLabel 标签的 Pods 和服务
kubectl -n my-ns delete po,svc --all                                      # 删除在 my-ns 名字空间中全部的 Pods 和服务
# 删除所有与 pattern1 或 pattern2 awk 模式匹配的 Pods
kubectl get pods  -n mynamespace --no-headers=true | awk '/pattern1|pattern2/{print $1}' | xargs  kubectl delete -n mynamespace pod
```

**Pod常用操作**

```bash
kubectl logs my-pod                                 # 获取 pod 日志（标准输出）
kubectl logs -l name=myLabel                        # 获取含 name=myLabel 标签的 Pods 的日志（标准输出）
kubectl logs my-pod --previous                      # 获取上个容器实例的 pod 日志（标准输出）
kubectl logs my-pod -c my-container                 # 获取 Pod 容器的日志（标准输出, 多容器场景）
kubectl logs -l name=myLabel -c my-container        # 获取含 name=myLabel 标签的 Pod 容器日志（标准输出, 多容器场景）
kubectl logs my-pod -c my-container --previous      # 获取 Pod 中某容器的上个实例的日志（标准输出, 多容器场景）
kubectl logs -f my-pod                              # 流式输出 Pod 的日志（标准输出）
kubectl logs -f my-pod -c my-container              # 流式输出 Pod 容器的日志（标准输出, 多容器场景）
kubectl logs -f -l name=myLabel --all-containers    # 流式输出含 name=myLabel 标签的 Pod 的所有日志（标准输出）
kubectl run -i --tty busybox --image=busybox -- sh  # 以交互式 Shell 运行 Pod
kubectl run nginx --image=nginx -n mynamespace      # 在指定名字空间中运行 nginx Pod
kubectl run nginx --image=nginx                     # 运行 ngins Pod 并将其规约写入到名为 pod.yaml 的文件
  --dry-run=client -o yaml > pod.yaml

kubectl attach my-pod -i                            # 挂接到一个运行的容器中
kubectl port-forward my-pod 5000:6000               # 在本地计算机上侦听端口 5000 并转发到 my-pod 上的端口 6000
kubectl exec my-pod -- ls /                         # 在已有的 Pod 中运行命令（单容器场景）
kubectl exec my-pod -c my-container -- ls /         # 在已有的 Pod 中运行命令（多容器场景）
kubectl top pod POD_NAME --containers               # 显示给定 Pod 和其中容器的监控数据
```

**节点操作**

```bash
kubectl cordon my-node                                                # 标记 my-node 节点为不可调度
kubectl drain my-node                                                 # 对 my-node 节点进行清空操作，为节点维护做准备
kubectl uncordon my-node                                              # 标记 my-node 节点为可以调度
kubectl top node my-node                                              # 显示给定节点的度量值
kubectl cluster-info                                                  # 显示主控节点和服务的地址
kubectl cluster-info dump                                             # 将当前集群状态转储到标准输出
kubectl cluster-info dump --output-directory=/path/to/cluster-state   # 将当前集群状态输出到 /path/to/cluster-state

# 如果已存在具有指定键和效果的污点，则替换其值为指定值
kubectl taint nodes foo dedicated=special-user:NoSchedule
```

**格式化输出**

要以特定格式将详细信息输出到终端窗口，可以将 -o 或 --output 参数添加到支持的 kubectl 命令

| 输出格式                    | 描述                                                         |
| :-------------------------- | :----------------------------------------------------------- |
| -o=自定义列=<规范>          | 使用逗号分隔的自定义列来打印表格                             |
| -o=自定义列文件=<文件名>    | 使用 <filename> 文件中的自定义列模板打印表格                 |
| -o=json                     | 输出 JSON 格式的 API 对象                                    |
| -o=jsonpath=<template>      | 打印 jsonpath 表达式中定义的字段                             |
| -o=jsonpath-file=<filename> | 打印在 <filename> 文件中定义的 jsonpath 表达式所指定的字段。 |
| -o=名称                     | 仅打印资源名称而不打印其他内容                               |
| -o=宽                       | 以纯文本格式输出额外信息，对于 Pod 来说，输出中包含了节点名称 |
| -o=yaml                     | 输出 YAML 格式的 API 对象                                    |

使用 -o=custom-columns 的示例：

```bash
# 集群中运行着的所有镜像
kubectl get pods -A -o=custom-columns='DATA:spec.containers[*].image'

 # 除 "k8s.gcr.io/coredns:1.6.2" 之外的所有镜像
kubectl get pods -A -o=custom-columns='DATA:spec.containers[?(@.image!="k8s.gcr.io/coredns:1.6.2")].image'

# 输出 metadata 下面的所有字段，无论 Pod 名字为何
kubectl get pods -A -o=custom-columns='DATA:metadata.*'
```

---

# k8s常用命令汇总



# 1.查看类命令



```csharp
# 获取节点和服务版本信息
 kubectl get nodes
# 获取节点和服务版本信息，并查看附加信息
 kubectl get nodes -o wide
# 获取pod信息，默认是default名称空间
kubectl get pod
# 获取pod信息，默认是default名称空间，并查看附加信息【如：pod的IP及在哪个节点运行】
kubectl get pod -o wide
# 获取指定名称空间的pod
kubectl get pod -n <namespaces>
# 获取指定名称空间中的指定pod
kubectl get pod podName -n <namespaces>
# 获取所有名称空间的pod
kubectl get pod -A 
# 查看pod的详细信息，以yaml格式或json格式显示
kubectl get pods -o yaml
kubectl get pods -o json

# 查看pod的标签信息
kubectl get pod -A --show-labels 
# 根据Selector（label query）来查询pod
kubectl get pod -A --selector="k8s-app=kube-dns"

# 查看运行pod的环境变量
kubectl exec podName env
# 查看指定pod的日志
kubectl logs -f --tail 500 -n <namespaces> <podname>
 
# 查看所有名称空间的service信息
kubectl get svc -A
# 查看指定名称空间的service信息
kubectl get svc -n <namespaces>

# 查看componentstatuses信息
kubectl get cs
# 查看所有configmaps信息
kubectl get cm -A
# 查看所有serviceaccounts信息
kubectl get sa -A
# 查看所有daemonsets信息
kubectl get ds -A
# 查看所有deployments信息
kubectl get deploy -A
# 查看所有replicasets信息
kubectl get rs -A
# 查看所有statefulsets信息
kubectl get sts -A
# 查看所有jobs信息
kubectl get jobs -A
# 查看所有ingresses信息
kubectl get ing -A
# 查看有哪些名称空间
kubectl get ns

# 查看pod的描述信息
kubectl describe pod <podName>
kubectl describe pod -n <namespaces> <podname>

# 查看node或pod的资源使用情况
# 需要heapster 或metrics-server支持
kubectl top node
kubectl top pod 

# 查看指定命令空间下指定pod下的容器信息
kubectl describe pod <podname> -n <namespaces> |grep container

# 查看集群信息
kubectl cluster-info   或  kubectl cluster-info dump
# 查看各组件信息【172.16.1.110为master机器】
kubectl -s https://172.16.1.110:6443 get componentstatuses
```

# 2.操作类命令



```bash
# 创建资源
kubectl create -f xxx.yaml
# 应用资源
kubectl apply -f xxx.yaml
# 应用资源，该目录下的所有 .yaml, .yml, 或 .json 文件都会被使用
kubectl apply -f <directory>
# 创建test名称空间
kubectl create namespace test

# 删除资源
kubectl delete -f xxx.yaml
kubectl delete -f <directory>
# 删除指定的pod
kubectl delete pod podName
# 删除指定名称空间的指定pod
kubectl delete pod -n <namespaces> podName
# 删除其他资源
kubectl delete svc svcName
kubectl delete deploy deployName
kubectl delete ns nsName
# 强制删除
kubectl delete pod podName -n nsName --grace-period=0 --force
kubectl delete pod podName -n nsName --grace-period=1
kubectl delete pod podName -n nsName --now

# 编辑资源
kubectl edit pod podName -n <namespaces>
```

# 3.进阶命令操作



```bash
# kubectl exec：进入pod启动的容器
kubectl exec -it podName -n <namespaces> /bin/sh    [[进入容器]]
kubectl exec -it podName -n <namespaces> /bin/bash  [[进入容器]]

# kubectl label：添加label值
kubectl label nodes k8s-node01 zone=north  [[为指定节点添加标签]] 
kubectl label nodes k8s-node01 zone-       [[为指定节点删除标签]]
kubectl label pod podName -n nsName role-name=test    [[为指定pod添加标签]]
kubectl label pod podName -n nsName role-name=dev --overwrite  [[修改lable标签值]]
kubectl label pod podName -n nsName role-name-        [[删除lable标签]]

# kubectl滚动升级； 
通过 kubectl apply -f myapp-deployment-v1.yaml 启动deploy
kubectl apply -f myapp-deployment-v2.yaml     [[通过配置文件滚动升级]]
kubectl set image deploy/myapp-deployment myapp="registry.cn-beijing.aliyuncs.com/google_registry/myapp:v3"   [[通过命令滚动升级]]
kubectl rollout undo deploy/myapp-deployment 或者 kubectl rollout undo deploy myapp-deployment    [[pod回滚到前一个版本]]
kubectl rollout undo deploy/myapp-deployment --to-revision=2  [[回滚到指定历史版本]]

# kubectl scale：动态伸缩
kubectl scale deploy myapp-deployment --replicas=5 
[[动态伸缩【根据资源类型和名称伸缩，其他配置「如：镜像版本不同」不生效】]]
kubectl scale --replicas=8 -f myapp-deployment-v2.yaml  
```

---

# K8s常用命令合集

## 1. 创建资源

一般创建资源会有两种方式：通过文件或者命令创建。

```bash
# 通过文件创建一个Deployment
kubectl create -f /path/to/deployment.yaml
cat /path/to/deployment.yaml | kubectl create -f -
# 不过一般可能更常用下面的命令来创建资源
kubectl apply -f /path/to/deployment.yaml

# 通过kubectl命令直接创建
kubectl run nginx_app --image=nginx:1.9.1 --replicas=3 
```

kubectl还提供了一些更新资源的命令，比如kubectl edit、kubectl patch和kubectl replace等。

```bash
# kubectl edit：相当于先用get去获取资源，然后进行更新，最后对更新后的资源进行apply
kubectl edit deployment/nginx_app

# kubectl patch：使用补丁修改、更新某个资源的字段，比如更新某个node
kubectl patch node/node-0 -p '{"spec":{"unschedulable":true}}'
kubectl patch -f node-0.json -p '{"spec": {"unschedulable": "true"}}'

# kubectl replace：使用配置文件来替换资源
kubectl replace -f /path/to/new_nginx_app.yaml
```

## 2. 查看资源

获取不同种类资源的信息。

```bash
# 一般命令的格式会如下：
kubectl get <resource_type>
# 比如获取K8s集群下pod的信息
kubectl get pod
# 更加详细的信息
kubectl get pod -o wide
# 指定资源的信息，格式：kubectl get <resource_type>/<resource_name>，比如获取deployment nginx_app的信息
kubectl get deployment/nginx_app -o wide
# 也可以对指定的资源进行格式化输出，比如输出格式为json、yaml等
kubectl get deployment/nginx_app -o json
# 还可以对输出结果进行自定义，比如对pod只输出容器名称和镜像名称
kubectl get pod httpd-app-5bc589d9f7-rnhj7 -o custom-columns=CONTAINER:.spec.containers[0].name,IMAGE:.spec.containers[0].image
# 获取某个特定key的值还可以输入如下命令得到，此目录参照go template的用法，且命令结尾'\n'是为了输出结果换行
kubectl get pod httpd-app-5bc589d9f7-rnhj7 -o template --template='{{(index spec.containers 0).name}}{{"\n"}}'
# 还有一些可选项可以对结果进行过滤，这儿就不一一列举了，如有兴趣，可参照kubectl get --help说明
```

## 3. 部署命令集

部署命令包括资源的运行管理命令、扩容和缩容命令和自动扩缩容命令。

### 3.1 rollout命令

管理资源的运行，比如eployment、Daemonet、StatefulSet等资源。

- 查看部署状态：比如更新deployment/nginx_app中容器的镜像后查看其更新的状态。

```bash
kubectl set image deployment/nginx_app nginx=nginx:1.9.1
kubectl rollout status deployment/nginx_app
```

- 资源的暂停及恢复：发出一次或多次更新前暂停一个 Deployment，然后再恢复它，这样就能在Deployment暂停期间进行多次修复工作，而不会发出不必要的 rollout。

```bash
# 暂停
kubectl rollout pause deployment/nginx_app
# 完成所有的更新操作命令后进行恢复
kubectl rollout resume deployment/nginx_app
```

- 回滚：如上对一个Deployment的image做了更新，但是如果遇到更新失败或误更新等情况时可以对其进行回滚。

```bash
# 回滚之前先查看历史版本信息
kubectl rollout history deployment/nginx_app
# 回滚
kubectl rollout undo deployment/nginx_app
# 当然也可以指定版本号回滚至指定版本
kubectl rollout undo deployment/nginx_app --to-revision=<version_index>
```

### 3.2 scale命令

对一个Deployment、RS、StatefulSet进行扩/缩容。

```bash
# 扩容
kubectl scale deployment/nginx_app --replicas=5
# 如果是缩容，把对应的副本数设置的比当前的副本数小即可
# 另外，还可以针对当前的副本数目做条件限制，比如当前副本数是5则进行缩容至副本数目为3
kubectl scale --current-replicas=5 --replicas=3 deployment/nginx_app
```

### 3.3 autoscale命令

通过创建一个autoscaler，可以自动选择和设置在K8s集群中Pod的数量。

```bash
# 基于CPU的使用率创建3-10个pod
kubectl autoscale deployment/nginx_app --min=3 --max=10 --cpu_percent=80
```

## 4. 集群管理命令

### 4.1 cordon & uncordon命令

设置是否能够将pod调度到该节点上。

```bash
# 不可调度
kubectl cordon node-0

# 当某个节点需要维护时，可以驱逐该节点上的所有pods(会删除节点上的pod，并且自动通过上面命令设置
# 该节点不可调度，然后在其他可用节点重新启动pods)
kubectl drain node-0
# 待其维护完成后，可再设置该节点为可调度
kubectl uncordon node-0
```

### 4.2 taint命令

目前仅能作用于节点资源，一般这个命令通常会结合pod的tolerations字段结合使用，对于没有设置对应toleration的pod是不会调度到有该taint的节点上的，这样就可以避免pod被调度到不合适的节点上。一个节点的taint一般会包括key、value和effect(effect只能在NoSchedule, PreferNoSchedule, NoExecute中取值)。

```bash
# 设置taint
kubecl taint nodes node-0 key1=value1:NoSchedule
# 移除taint
kubecl taint nodes node-0 key1:NoSchedule-
```

如果pod想要被调度到上述设置了taint的节点node-0上，则需要在该pod的spec的tolerations字段设置：

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"

# 或者
tolerations:
- key: "key1"
  operator: "Exists"
  effect: "NoSchedule"
```

## 5. 其它

```bash
# 映射端口允许外部访问
kubectl expose deployment/nginx_app --type='NodePort' --port=80
# 然后通过kubectl get services -o wide来查看被随机映射的端口
# 如此就可以通过node的外部IP和端口来访问nginx服务了

# 转发本地端口访问Pod的应用服务程序
kubectl port-forward nginx_app_pod_0 8090:80
# 如此，本地可以访问：curl -i localhost:8090

# 在创建或启动某些资源的时候没有达到预期结果，可以使用如下命令先简单进行故障定位
kubectl describe deployment/nginx_app
kubectl logs nginx_pods
kubectl exec nginx_pod -c nginx-app <command>

# 集群内部调用接口(比如用curl命令)，可以采用代理的方式，根据返回的ip及端口作为baseurl
kubectl proxy &

# 查看K8s支持的完整资源列表
kubectl api-resources

# 查看K8s支持的api版本
kubectl api-versions
```

---

# k8s常用命令(需要可查ctrl+f)



## kubernetes用到的一些命令

**kubectl管理工具以及命令**

![img](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/18885539-d5d52769d1469bfe.png)

image

**基础命令：create，delete，get，run，expose，set，explain，edit**

**create命令**：根据文件或者输入来创建资源



```css
# 创建Deployment和Service资源
kubectl create -f javak8s-deployment.yaml
kubectl create -f javak8s-service.yaml
```

**delete命令：**删除资源



```cpp
# 根据yaml文件删除对应的资源，但是yaml文件并不会被删除，这样更加高效
kubectl delete -f javak8s-deployment.yaml 
kubectl delete -f javak8s-service.yaml
# 也可以通过具体的资源名称来进行删除，使用这个删除资源，需要同时删除pod和service资源才行
kubectl delete 具体的资源名称
```

**get命令：**获得资源信息



```csharp
# 查看所有的资源信息
kubectl get all
# 查看pod列表
kubectl get pod
# 显示pod节点的标签信息
kubectl get pod --show-labels
# 根据指定标签匹配到具体的pod
kubectl get pods -l app=example
# 查看node节点列表
kubectl get node 
# 显示node节点的标签信息
kubectl get node --show-labels
# 查看pod详细信息，也就是可以查看pod具体运行在哪个节点上（ip地址信息）
kubectl get pod -o wide
# 查看服务的详细信息，显示了服务名称，类型，集群ip，端口，时间等信息
kubectl get svc
[root@master ~]# kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
go-service      NodePort    10.10.10.247   <none>        8089:33702/TCP   29m
java-service    NodePort    10.10.10.248   <none>        8082:32823/TCP   5h17m
kubernetes      ClusterIP   10.10.10.1     <none>        443/TCP          5d16h
nginx-service   NodePort    10.10.10.146   <none>        88:34823/TCP     2d19h
# 查看命名空间
kubectl get ns
# 查看所有pod所属的命名空间
kubectl get pod --all-namespaces
# 查看所有pod所属的命名空间并且查看都在哪些节点上运行
kubectl get pod --all-namespaces  -o wide
# 查看目前所有的replica set，显示了所有的pod的副本数，以及他们的可用数量以及状态等信息
kubectl get rs
[root@master ~]# kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
go-deployment-58c76f7d5c      1         1         1       32m
java-deployment-76889f56c5    1         1         1       5h21m
nginx-deployment-58d6d6ccb8   3         3         3       2d19h
# 查看目前所有的deployment
kubectl get deployment
[root@master ~]# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
go-deployment      1/1     1            1           34m
java-deployment    1/1     1            1           5h23m
nginx-deployment   3/3     3            3           2d19h
# 查看已经部署了的所有应用，可以看到容器，以及容器所用的镜像，标签等信息
 kubectl get deploy -o wide
[root@master bin]# kubectl get deploy -o wide     
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES       SELECTOR
nginx   3/3     3            3           16m   nginx        nginx:1.10   app=example
```

**run命令：**在集群中创建并运行一个或多个容器镜像。



```objectivec
# 基本语法
run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [--command] -- [COMMAND] [args...]
# 示例，运行一个名称为nginx，副本数为3，标签为app=example，镜像为nginx:1.10，端口为80的容器实例
kubectl run nginx --replicas=3 --labels="app=example" --image=nginx:1.10 --port=80
其他用法参见：http://docs.kubernetes.org.cn/468.html
```

**expose命令：**创建一个service服务，并且暴露端口让外部可以访问



```bash
# 创建一个nginx服务并且暴露端口让外界可以访问
kubectl expose deployment nginx --port=88 --type=NodePort --target-port=80 --name=nginx-service
关于expose的详细用法参见：http://docs.kubernetes.org.cn/475.html
```

**set命令：** 配置应用的一些特定资源，也可以修改应用已有的资源



```cpp
# 使用kubectl set --help查看，它的子命令，env，image，resources，selector，serviceaccount，subject。
set命令详情参见：http://docs.kubernetes.org.cn/669.html
语法：
resources (-f FILENAME | TYPE NAME) ([--limits=LIMITS & --requests=REQUESTS]
```

**kubectl set resources 命令**

这个命令用于设置资源的一些范围限制。

资源对象中的[Pod](https://links.jianshu.com/go?to=http%3A%2F%2Fdocs.kubernetes.org.cn%2F312.html)可以指定计算资源需求（CPU-单位m、内存-单位Mi），即使用的最小资源请求（Requests），限制（Limits）的最大资源需求，Pod将保证使用在设置的资源数量范围。

对于每个Pod资源，如果指定了Limits（限制）值，并省略了Requests（请求），则Requests默认为Limits的值。

**可用资源对象包括(支持大小写)：replicationcontroller、deployment、daemonset、job、replicaset。**

例如：



```bash
# 将deployment的nginx容器cpu限制为“200m”，将内存设置为“512Mi”
kubectl set resources deployment nginx -c=nginx --limits=cpu=200m,memory=512Mi
# 为nginx中的所有容器设置 Requests和Limits
kubectl set resources deployment nginx --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256Mi
# 删除nginx中容器的计算资源值
kubectl set resources deployment nginx --limits=cpu=0,memory=0 --requests=cpu=0,memory=0
```

**kubectl set selector命令**

设置资源的selector（选择器）。如果在调用"set selector"命令之前已经存在选择器，则新创建的选择器将覆盖原来的选择器。

selector必须以字母或数字开头，最多包含63个字符，可使用：字母、数字、连字符" - " 、点"."和下划线" _ "。如果指定了--resource-version，则更新将使用此资源版本，否则将使用现有的资源版本。

**注意：目前selector命令只能用于Service对象。**



```bash
# 语法
selector (-f FILENAME | TYPE NAME) EXPRESSIONS [--resource-version=version]
```

**kubectl set image命令**

 用于更新现有资源的容器镜像。

可用资源对象包括：pod (po)、replicationcontroller (rc)、deployment (deploy)、daemonset (ds)、job、replicaset (rs)。



```bash
# 语法
image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N
```



```bash
# 将deployment中的nginx容器镜像设置为“nginx：1.9.1”。
kubectl set image deployment/nginx busybox=busybox nginx=nginx:1.9.1
# 所有deployment和rc的nginx容器镜像更新为“nginx：1.9.1”
kubectl set image deployments,rc nginx=nginx:1.9.1 --all
# 将daemonset abc的所有容器镜像更新为“nginx：1.9.1”
kubectl set image daemonset abc *=nginx:1.9.1
# 从本地文件中更新nginx容器镜像
kubectl set image -f path/to/file.yaml nginx=nginx:1.9.1 --local -o yaml
```

------

**explain命令**：用于显示资源文档信息



```undefined
kubectl explain rs
```

**edit命令**:用于编辑资源信息



```bash
# 编辑Deployment nginx的一些信息
kubectl edit deployment nginx
# 编辑service类型的nginx的一些信息
kubectl edit service/nginx
```

------

**设置命令：label，annotate，completion**

**label命令**:用于更新（增加、修改或删除）资源上的 label（标签）

- label 必须以字母或数字开头，可以使用字母、数字、连字符、点和下划线，最长63个字符。
- 如果--overwrite 为 true，则可以覆盖已有的 label，否则尝试覆盖 label 将会报错。
- 如果指定了--resource-version，则更新将使用此资源版本，否则将使用现有的资源版本。



```bash
# 语法
label [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version]
```



```bash
# 给名为foo的Pod添加label unhealthy=true
kubectl label pods foo unhealthy=true
# 给名为foo的Pod修改label 为 'status' / value 'unhealthy'，且覆盖现有的value
kubectl label --overwrite pods foo status=unhealthy
# 给 namespace 中的所有 pod 添加 label
kubectl label  pods --all status=unhealthy
# 仅当resource-version=1时才更新 名为foo的Pod上的label
kubectl label pods foo status=unhealthy --resource-version=1
# 删除名为“bar”的label 。（使用“ - ”减号相连）
kubectl label pods foo bar-
```

**annotate命令**：更新一个或多个资源的Annotations信息。也就是注解信息，可以方便的查看做了哪些操作。

- Annotations由key/value组成。
- Annotations的目的是存储辅助数据，特别是通过工具和系统扩展操作的数据，更多介绍在[这里](https://links.jianshu.com/go?to=http%3A%2F%2Fdocs.kubernetes.org.cn%2F255.html)。
- 如果--overwrite为true，现有的annotations可以被覆盖，否则试图覆盖annotations将会报错。
- 如果设置了--resource-version，则更新将使用此resource version，否则将使用原有的resource version。



```bash
# 语法
annotate [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version]
```



```kotlin
# 更新Pod“foo”，设置annotation “description”的value “my frontend”，如果同一个annotation多次设置，则只使用最后设置的value值
kubectl annotate pods foo description='my frontend'
# 根据“pod.json”中的type和name更新pod的annotation
kubectl annotate -f pod.json description='my frontend'
# 更新Pod"foo"，设置annotation“description”的value“my frontend running nginx”，覆盖现有的值
kubectl annotate --overwrite pods foo description='my frontend running nginx'
# 更新 namespace中的所有pod
kubectl annotate pods --all description='my frontend running nginx'
# 只有当resource-version为1时，才更新pod ' foo '
kubectl annotate pods foo description='my frontend running nginx' --resource-version=1
# 通过删除名为“description”的annotations来更新pod ' foo '。#不需要- overwrite flag。
kubectl annotate pods foo description-
```

**completion命令**：用于设置kubectl命令自动补全



```bash
$ source <(kubectl completion bash) # setup autocomplete in bash, bash-completion package should be installed first.
$ source <(kubectl completion zsh)  # setup autocomplete in zsh
```

------

**kubectl 部署命令：rollout，rolling-update，scale，autoscale**

**rollout命令**:用于对资源进行管理

可用资源包括：deployments，daemonsets。

子命令：

- [*history*](https://links.jianshu.com/go?to=http%3A%2F%2Fdocs.kubernetes.org.cn%2F645.html)（查看历史版本）
- [*pause*](https://links.jianshu.com/go?to=http%3A%2F%2Fdocs.kubernetes.org.cn%2F647.html)（暂停资源）
- [*resume*](https://links.jianshu.com/go?to=http%3A%2F%2Fdocs.kubernetes.org.cn%2F650.html)（恢复暂停资源）
- [*status*](https://links.jianshu.com/go?to=http%3A%2F%2Fdocs.kubernetes.org.cn%2F652.html)（查看资源状态）
- [*undo*](https://links.jianshu.com/go?to=http%3A%2F%2Fdocs.kubernetes.org.cn%2F654.html)（回滚版本）



```bash
# 语法
kubectl rollout SUBCOMMAND
# 回滚到之前的deployment
kubectl rollout undo deployment/abc
# 查看daemonet的状态
kubectl rollout status daemonset/foo
```

**rolling-update命令:**执行指定ReplicationController的滚动更新。

该命令创建了一个新的RC， 然后一次更新一个pod方式逐步使用新的PodTemplate，最终实现Pod滚动更新，new-controller.json需要与之前RC在相同的namespace下。



```bash
# 语法
rolling-update OLD_CONTROLLER_NAME ([NEW_CONTROLLER_NAME] --image=NEW_CONTAINER_IMAGE | -f NEW_CONTROLLER_SPEC)
# 使用frontend-v2.json中的新RC数据更新frontend-v1的pod
kubectl rolling-update frontend-v1 -f frontend-v2.json
# 使用JSON数据更新frontend-v1的pod
cat frontend-v2.json | kubectl rolling-update frontend-v1 -f -
# 其他的一些滚动更新
kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2

kubectl rolling-update frontend --image=image:v2

kubectl rolling-update frontend-v1 frontend-v2 --rollback
```

**scale命令**：扩容或缩容 Deployment、ReplicaSet、Replication Controller或 Job 中Pod数量

scale也可以指定多个前提条件，如：当前副本数量或 --resource-version ，进行伸缩比例设置前，系统会先验证前提条件是否成立。这个就是弹性伸缩策略



```bash
# 语法
kubectl scale [--resource-version=version] [--current-replicas=count] --replicas=COUNT (-f FILENAME | TYPE NAME)
# 将名为foo中的pod副本数设置为3。
kubectl scale --replicas=3 rs/foo
kubectl scale deploy/nginx --replicas=30
# 将由“foo.yaml”配置文件中指定的资源对象和名称标识的Pod资源副本设为3
kubectl scale --replicas=3 -f foo.yaml
# 如果当前副本数为2，则将其扩展至3。
kubectl scale --current-replicas=2 --replicas=3 deployment/mysql
# 设置多个RC中Pod副本数量
kubectl scale --replicas=5 rc/foo rc/bar rc/baz
```

**autoscale命令：** 这个比scale更加强大，也是弹性伸缩策略 ，它是根据流量的多少来自动进行扩展或者缩容

指定Deployment、ReplicaSet或ReplicationController，并创建已经定义好资源的自动伸缩器。使用自动伸缩器可以根据需要自动增加或减少系统中部署的pod数量。



```swift
# 语法
kubectl autoscale (-f FILENAME | TYPE NAME | TYPE/NAME) [--min=MINPODS] --max=MAXPODS [--cpu-percent=CPU] [flags]
# 使用 Deployment “foo”设定，使用默认的自动伸缩策略，指定目标CPU使用率，使其Pod数量在2到10之间
kubectl autoscale deployment foo --min=2 --max=10
# 使用RC“foo”设定，使其Pod的数量介于1和5之间，CPU使用率维持在80％
kubectl autoscale rc foo --max=5 --cpu-percent=80
```

------

**集群管理命令：certificate，cluster-info，top，cordon，uncordon，drain，taint**

**certificate命令**：用于证书资源管理，授权等



```bash
[root@master ~]# kubectl certificate --help
Modify certificate resources.

Available Commands:
  approve     Approve a certificate signing request
  deny        Deny a certificate signing request

Usage:
  kubectl certificate SUBCOMMAND [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).

# 例如，当有node节点要向master请求，那么是需要master节点授权的
kubectl  certificate approve node-csr-81F5uBehyEyLWco5qavBsxc1GzFcZk3aFM3XW5rT3mw node-csr-Ed0kbFhc_q7qx14H3QpqLIUs0uKo036O2SnFpIheM18
```

**cluster-info命令：**显示集群信息



```bash
kubectl cluster-info

[root@master ~]# kubectl cluster-info
Kubernetes master is running at http://localhost:8080
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

**top命令：**用于查看资源的cpu，内存磁盘等资源的使用率



```undefined
kubectl top pod --all-namespaces
它需要heapster运行才行
```

**cordon命令**：用于标记某个节点不可调度

**uncordon命令：**用于标签节点可以调度

**drain命令：** 用于在维护期间排除节点。

**taint命令**：参见：[https://blog.frognew.com/2018/05/taint-and-toleration.html](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.frognew.com%2F2018%2F05%2Ftaint-and-toleration.html)

------

**集群故障排查和调试命令：describe，logs，exec，attach，port-foward，proxy，cp，auth**

**describe命令**：显示特定资源的详细信息



```bash
# 语法
kubectl describe TYPE NAME_PREFIX
（首先检查是否有精确匹配TYPE和NAME_PREFIX的资源，如果没有，将会输出所有名称以NAME_PREFIX开头的资源详细信息）
支持的资源包括但不限于（大小写不限）：pods (po)、services (svc)、 replicationcontrollers (rc)、nodes (no)、events (ev)、componentstatuses (cs)、 limitranges (limits)、persistentvolumes (pv)、persistentvolumeclaims (pvc)、 resourcequotas (quota)和secrets。
[[查看my-nginx]] pod的详细状态
kubectl describe po my-nginx
```

**logs命令：**用于在一个pod中打印一个容器的日志，如果pod中只有一个容器，可以省略容器名



```ruby
# 语法
kubectl logs [-f] [-p] POD [-c CONTAINER]

# 返回仅包含一个容器的pod nginx的日志快照
$ kubectl logs nginx
# 返回pod ruby中已经停止的容器web-1的日志快照
$ kubectl logs -p -c ruby web-1
# 持续输出pod ruby中的容器web-1的日志
$ kubectl logs -f -c ruby web-1
# 仅输出pod nginx中最近的20条日志
$ kubectl logs --tail=20 nginx
# 输出pod nginx中最近一小时内产生的所有日志
$ kubectl logs --since=1h nginx
# 参数选项
  -c, --container="": 容器名。
  -f, --follow[=false]: 指定是否持续输出日志（实时日志）。
      --interactive[=true]: 如果为true，当需要时提示用户进行输入。默认为true。
      --limit-bytes=0: 输出日志的最大字节数。默认无限制。
  -p, --previous[=false]: 如果为true，输出pod中曾经运行过，但目前已终止的容器的日志。
      --since=0: 仅返回相对时间范围，如5s、2m或3h，之内的日志。默认返回所有日志。只能同时使用since和since-time中的一种。
      --since-time="": 仅返回指定时间（RFC3339格式）之后的日志。默认返回所有日志。只能同时使用since和since-time中的一种。
      --tail=-1: 要显示的最新的日志条数。默认为-1，显示所有的日志。
      --timestamps[=false]: 在日志中包含时间戳。
```

**exec命令**：进入容器进行交互，在容器中执行命令



```bash
# 语法
kubectl exec POD [-c CONTAINER] -- COMMAND [args...]
[[命令选项]]
  -c, --container="": 容器名。如果未指定，使用pod中的一个容器。
  -p, --pod="": Pod名。
  -i, --stdin[=false]: 将控制台输入发送到容器。
  -t, --tty[=false]: 将标准输入控制台作为容器的控制台输入。
# 进入nginx容器，执行一些命令操作
kubectl exec -it nginx-deployment-58d6d6ccb8-lc5fp bash
```

**attach命令**：连接到一个正在运行的容器。



```bash
[[语法]]
kubectl attach POD -c CONTAINER
# 参数选项
-c, --container="": 容器名。如果省略，则默认选择第一个 pod
  -i, --stdin[=false]: 将控制台输入发送到容器。
  -t, --tty[=false]: 将标准输入控制台作为容器的控制台输入。

# 获取正在运行中的pod 123456-7890的输出，默认连接到第一个容器
kubectl attach 123456-7890
# 获取pod 123456-7890中ruby-container的输出
kubectl attach 123456-7890 -c ruby-container
# 切换到终端模式，将控制台输入发送到pod 123456-7890的ruby-container的“bash”命令，并将其输出到控制台/
# 错误控制台的信息发送回客户端。
kubectl attach 123456-7890 -c ruby-container -i -t
```

**cp命令**：拷贝文件或者目录到pod容器中

用于pod和外部的文件交换,类似于docker 的cp，就是将容器中的内容和外部的内容进行交换。

------

**其他命令：api-servions，config，help，plugin，version**

**api-servions命令**：打印受支持的api版本信息



```csharp
# kubectl api-versions
[root@master ~]# kubectl api-versions
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1beta1
apps/v1
apps/v1beta1
apps/v1beta2
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
batch/v1
batch/v1beta1
certificates.k8s.io/v1beta1
events.k8s.io/v1beta1
extensions/v1beta1
networking.k8s.io/v1
policy/v1beta1
rbac.authorization.k8s.io/v1
rbac.authorization.k8s.io/v1beta1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
```

**help命令**：用于查看命令帮助



```bash
# 显示全部的命令帮助提示
kubectl --help
# 具体的子命令帮助，例如
kubectl create --help
```

**config**:用于修改kubeconfig配置文件（用于访问api，例如配置认证信息）

**version命令：**打印客户端和服务端版本信息



```css
[root@master ~]# kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.3", GitCommit:"2d3c76f9091b6bec110a5e63777c332469e0cba2", GitTreeState:"clean", BuildDate:"2019-08-19T11:13:54Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.0", GitCommit:"925c127ec6b946659ad0fd596fa959be43f0cc05", GitTreeState:"clean", BuildDate:"2017-12-15T20:55:30Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
```

**plugin命令：**运行一个命令行插件

------

**高级命令：apply，patch，replace，convert**

**apply命令：** 通过文件名或者标准输入对资源应用配置

通过文件名或控制台输入，对资源进行配置。 如果资源不存在，将会新建一个。可以使用 JSON 或者 YAML 格式。



```php
# 语法
kubectl apply -f FILENAME

# 将pod.json中的配置应用到pod
kubectl apply -f ./pod.json
# 将控制台输入的JSON配置应用到Pod
cat pod.json | kubectl apply -f -

选项
-f, --filename=[]: 包含配置信息的文件名，目录名或者URL。
      --include-extended-apis[=true]: If true, include definitions of new APIs via calls to the API server. [default true]
  -o, --output="": 输出模式。"-o name"为快捷输出(资源/name).
      --record[=false]: 在资源注释中记录当前 kubectl 命令。
  -R, --recursive[=false]: Process the directory used in -f, --filename recursively. Useful when you want to manage related manifests organized within the same directory.
      --schema-cache-dir="~/.kube/schema": 非空则将API schema缓存为指定文件，默认缓存到'$HOME/.kube/schema'
      --validate[=true]: 如果为true，在发送到服务端前先使用schema来验证输入。
```

**patch命令：** 使用补丁修改，更新资源的字段，也就是修改资源的部分内容



```rust
# 语法
kubectl patch (-f FILENAME | TYPE NAME) -p PATCH

# Partially update a node using strategic merge patch
kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}'
# Update a container's image; spec.containers[*].name is required because it's a merge key
kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"kubernetes-serve-hostname","image":"new image"}]}}'
```

**replace命令：** 通过文件或者标准输入替换原有资源



```bash
# 语法
kubectl replace -f FILENAME

# Replace a pod using the data in pod.json.
kubectl replace -f ./pod.json
# Replace a pod based on the JSON passed into stdin.
cat pod.json | kubectl replace -f -
# Update a single-container pod's image version (tag) to v4
kubectl get pod mypod -o yaml | sed 's/\(image: myimage\):.*$/\1:v4/' | kubectl replace -f -
# Force replace, delete and then re-create the resource
kubectl replace --force -f ./pod.json
```

**convert命令：** 不同的版本之间转换配置文件



```bash
# 语法
kubectl convert -f FILENAME

# Convert 'pod.yaml' to latest version and print to stdout.
kubectl convert -f pod.yaml
# Convert the live state of the resource specified by 'pod.yaml' to the latest version
# and print to stdout in json format.
kubectl convert -f pod.yaml --local -o json
# Convert all files under current directory to latest version and create them all.
kubectl convert -f . | kubectl create -f -
```

以上就是k8s的一些基本命令。

---

# k8s运维命令大全

发布于2021-12-04 10:23:42阅读 7680

> k8s常用命令
>
> node
>
> 查看服务器节点 kubectl get nodes
>
> 查看服务器节点详情 kubectl get nodes -o wide
>
> 节点打标签 kubectl label nodes <节点名称> labelName=<标签名称>
>
> 查看节点标签 kubectl get node --show-labels
>
> 删除节点标签 kubectl label node <节点名称> labelName-
>
> 
>
> 
>
> pod
>
> 查看pod节点 kubectl get pod
>
> 查看pod节点详情 kubectl get pod -o wide
>
> 查看所有名称空间下的pod kubectl get pod --all-namespaces
>
> 根据yaml文件创建pod kubectl apply -f <文件名称>
>
> 根据yaml文件删除pod kubectl delete -f <文件名称>
>
> 删除pod节点 kubectl delete pod <pod名称> -n <名称空间>
>
> 查看异常的pod节点 kubectl get pods -n <名称空间> | grep -v Running
>
> 查看异常pod节点的日志 kubectl describe pod <pod名称> -n <名称空间>
>
> 
>
> 
>
> svc
>
> 查看服务 kubectl get svc
>
> 查看服务详情 kubectl get svc -o wide
>
> 查看所有名称空间下的服务 kubectl get svc --all-namespaces
>
> \#查看所有namespace的pods运行情况
>
> kubectl get pods --all-namespaces
>
> \#查看具体pods，记得后边跟namespace名字哦
>
> kubectl get pods kubernetes-dashboard-76479d66bb-nj8wr --namespace=kube- system
>
> 查看pods具体信息
>
> kubectl get pods -o wide kubernetes-dashboard-76479d66bb-nj8wr --namespace=kube-system
>
> 查看集群健康状态
>
> kubectl get cs
>
> 获取所有deployment
>
> kubectl get deployment --all-namespaces
>
> 查看kube-system namespace下面的pod/svc/deployment 等等（-o wide 选项可以查看存在哪个对应的节点）
>
> kubectl get pod /svc/deployment -n kube-system
>
> 列出该 namespace 中的所有 pod 包括未初始化的
>
> kubectl get pods --include-uninitialized
>
> 查看deployment()
>
> kubectl get deployment nginx-app
>
> 查看rc和servers
>
> kubectl get rc,services
>
> 查看pods结构信息（重点，通过这个看[日志分析](https://cloud.tencent.com/product/es?from=10680)错误）
>
> 对控制器和服务，node同样有效
>
> kubectl describe pods xxxxpodsname --namespace=xxxnamespace
>
> 其他控制器类似吧，就是kubectl get 控制器 控制器具体名称
>
> 查看pod日志
>
> kubectl logs $POD_NAME
>
> 查看pod变量
>
> kubectl exec my-nginx-5j8ok -- printenv | grep SERVICE
>
> 集群
>
> kubectl get cs # 集群健康情况
>
> kubectl cluster-info # 集群核心组件运行情况
>
> kubectl get namespaces # 表空间名
>
> kubectl version # 版本
>
> kubectl api-versions # API
>
> kubectl get events # 查看事件
>
> kubectl get nodes //获取全部节点
>
> kubectl delete node k8s2 //删除节点
>
> kubectl rollout status deploy nginx-test
>
> kubectl get deployment --all-namespaces
>
> kubectl get svc --all-namespaces
>
> 创建
>
> kubectl create -f ./nginx.yaml # 创建资源
>
> kubectl apply -f xxx.yaml （创建+更新，可以重复使用）
>
> kubectl create -f . # 创建当前目录下的所有yaml资源
>
> kubectl create -f ./nginx1.yaml -f ./mysql2.yaml # 使用多个文件创建资源
>
> kubectl create -f ./dir # 使用目录下的所有清单文件来创建资源
>
> kubectl create -f https://git.io/vPieo # 使用 url 来创建资源
>
> kubectl run -i --tty busybox --image=busybox ----创建带有终端的pod
>
> kubectl run nginx --image=nginx # 启动一个 nginx 实例
>
> kubectl run mybusybox --image=busybox --replicas=5 ----启动多个pod
>
> kubectl explain pods,svc # 获取 pod 和 svc 的文档
>
> 更新
>
> kubectl rolling-update python-v1 -f python-v2.json # 滚动更新 pod frontend-v1
>
> kubectl rolling-update python-v1 python-v2 --image=image:v2 # 更新资源名称并更新镜像
>
> kubectl rolling-update python --image=image:v2 # 更新 frontend pod 中的镜像
>
> kubectl rolling-update python-v1 python-v2 --rollback # 退出已存在的进行中的滚动更新
>
> cat pod.json | kubectl replace -f - # 基于 stdin 输入的 JSON 替换 pod
>
> 为 nginx RC 创建服务，启用本地 80 端口连接到[容器](https://cloud.tencent.com/product/tke?from=10680)上的 8000 端口
>
> kubectl expose rc nginx --port=80 --target-port=8000
>
> 更新单容器 pod 的镜像版本（tag）到 v4
>
> kubectl get pod nginx-pod -o yaml | sed 's/\(image: myimage\):.*$/\1:v4/' | kubectl replace -f -
>
> kubectl label pods nginx-pod new-label=awesome # 添加标签
>
> kubectl annotate pods nginx-pod icon-url=http://goo.gl/XXBTWq # 添加注解
>
> kubectl autoscale deployment foo --min=2 --max=10 # 自动扩展 deployment “foo”
>
> 编辑资源
>
> kubectl edit svc/docker-registry # 编辑名为 docker-registry 的 service
>
> KUBE_EDITOR="nano" kubectl edit svc/docker-registry # 使用其它编辑器
>
> vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf [[修改启动参数]]
>
> 动态伸缩pod
>
> kubectl scale --replicas=3 rs/foo # 将foo副本集变成3个
>
> kubectl scale --replicas=3 -f foo.yaml # 缩放“foo”中指定的资源。
>
> kubectl scale --current-replicas=2 --replicas=3 deployment/mysql # 将deployment/mysql从2个变成3个
>
> kubectl scale --replicas=5 rc/foo rc/bar rc/baz # 变更多个控制器的数量
>
> kubectl rollout status deploy deployment/mysql # 查看变更进度
>
> \#label 操作
>
> kubectl label：添加label值 kubectl label nodes node1 zone=north [[增加节点lable值]] spec.nodeSelector: zone: north [[指定pod在哪个节点]]
>
> kubectl label pod redis-master-1033017107-q47hh role=master [[增加lable值]] key=value
>
> kubectl label pod redis-master-1033017107-q47hh role- [[删除lable值]]
>
> kubectl label pod redis-master-1033017107-q47hh role=backend --overwrite [[修改lable值]]
>
> 滚动升级
>
> kubectl rolling-update：滚动升级 kubectl rolling-update redis-master -f redis- master-controller-v2.yaml [[配置文件滚动升级]]
>
> kubectl rolling-update redis-master --image=redis-master:2.0 [[命令升级]]
>
> kubectl rolling-update redis-master --image=redis-master:1.0 --rollback [[pod版本回滚]]
>
> etcdctl 常用操作
>
> etcdctl cluster-health [[检查网络集群健康状态]]
>
> etcdctl --endpoints=https://192.168.71.221:2379 cluster-health [[带有安全认证检查网络集群健康状态]]
>
> etcdctl member list
>
> etcdctl set /k8s/network/config ‘{ “Network”: “10.1.0.0/16” }’
>
> etcdctl get /k8s/network/config
>
> 删除
>
> kubectl delete pod -l app=flannel -n kube-system # 根据label删除：
>
> kubectl delete -f ./pod.json # 删除 pod.json 文件中定义的类型和名称的 pod
>
> kubectl delete pod,service baz foo # 删除名为“baz”的 pod 和名为“foo”的 service
>
> kubectl delete pods,services -l name=myLabel # 删除具有 name=myLabel 标签的 pod 和 serivce
>
> kubectl delete pods,services -l name=myLabel --include-uninitialized # 删除具有 name=myLabel 标签的 pod 和 service，包括尚未初始化的
>
> kubectl -n my-ns delete po,svc --all # 删除 my-ns namespace下的所有 pod 和 serivce，包括尚未初始化的
>
> kubectl delete pods prometheus-7fcfcb9f89-qkkf7 --grace-period=0 --force 强制删除
>
> kubectl delete deployment kubernetes-dashboard --namespace=kube-system
>
> kubectl delete svc kubernetes-dashboard --namespace=kube-system
>
> kubectl delete -f kubernetes-dashboard.yaml
>
> kubectl replace --force -f ./pod.json # 强制替换，删除后重新创建资源。会导致服务中断。
>
> 交互
>
> kubectl logs nginx-pod # dump 输出 pod 的日志（stdout）
>
> kubectl logs nginx-pod -c my-container # dump 输出 pod 中容器的日志（stdout，pod 中有多个容器的情况下使用）
>
> kubectl logs -f nginx-pod # 流式输出 pod 的日志（stdout）
>
> kubectl logs -f nginx-pod -c my-container # 流式输出 pod 中容器的日志（stdout，pod 中有多个容器的情况下使用）
>
> kubectl run -i --tty busybox --image=busybox -- sh # 交互式 shell 的方式运行 pod
>
> kubectl attach nginx-pod -i # 连接到运行中的容器
>
> kubectl port-forward nginx-pod 5000:6000 # 转发 pod 中的 6000 端口到本地的 5000 端口
>
> kubectl exec nginx-pod -- ls / # 在已存在的容器中执行命令（只有一个容器的情况下）
>
> kubectl exec nginx-pod -c my-container -- ls / # 在已存在的容器中执行命令（pod 中有多个容器的情况下）
>
> kubectl top pod POD_NAME --containers # 显示指定 pod和容器的指标度量
>
> kubectl exec -ti podName /bin/bash # 进入pod
>
> 调度配置
>
> kubectl cordon k8s-node # 标记 my-node 不可调度
>
> kubectl drain k8s-node # 清空 my-node 以待维护
>
> kubectl uncordon k8s-node # 标记 my-node 可调度
>
> kubectl top node k8s-node # 显示 my-node 的指标度量
>
> kubectl cluster-info dump # 将当前集群状态输出到 stdout
>
> kubectl cluster-info dump --output-directory=/path/to/cluster-state # 将当前集群状态输出到 /path/to/cluster-state
>
> \#如果该键和影响的污点（taint）已存在，则使用指定的值替换
>
> kubectl taint nodes foo dedicated=special-user:NoSchedule
>
> \#查看kubelet进程启动参数
>
> ps -ef | grep kubelet
>
> 查看日志:
>
> journalctl -u kubelet -f
>
> 导出配置文件：
>
> 导出proxy
>
> kubectl get ds -n kube-system -l k8s-app=kube-proxy -o yaml>kube-proxy- ds.yaml
>
> 导出kube-dns
>
> kubectl get deployment -n kube-system -l k8s-app=kube-dns -o yaml >kube-dns- dp.yaml
>
> kubectl get services -n kube-system -l k8s-app=kube-dns -o yaml >kube-dns- services.yaml
>
> 导出所有 configmap
>
> kubectl get configmap -n kube-system -o wide -o yaml > configmap.yaml
>
> 复杂操作命令：
>
> 删除kube-system 下Evicted状态的所有pod：
>
> kubectl get pods -n kube-system |grep Evicted| awk ‘{print $1}’|xargs kubectl delete pod -n kube-system
>
> 以下为维护环境相关命令：
>
> 重启kubelet服务
>
> systemctl daemon-reload
>
> systemctl restart kubelet

---

# kubectl命令网站

[Kubernetes(K8s) kubectl logs 常用命令-CJavaPy](https://www.cjavapy.com/article/2425/)

---

# kubernetes 查看pod 的容器日志

1.pod若处于运行状态，则通过kubectl logs 即可

```powershell
# 查看指定pod的日志
kubectl logs <pod_name>
kubectl logs -f <pod_name> [[类似tail]] -f的方式查看(tail -f 实时查看日志文件 tail -f 日志文件log)

# 查看指定pod中指定容器的日志
kubectl logs <pod_name> -c <container_name>

kubectl logs pod_name -c container_name -n namespace (一次性查看)
kubectl logs -f <pod_name> -n namespace (tail -f方式实时查看)
```

2.若pod处于init状态，则需要通过docker ps查看

```perl
[[获取对应的pod]] name
kubectl get pods -n  namespace -o wide (STATUS是init的pod_name)

[[通过docker]] ps 获取该pod的中的CONTAINER ID
docker ps | grep pod_name

[[通过docker]] log获取对应的日志信息
docker logs CONTAINER_ID
```

---

kubectl logs 常用于将容器中的日志导出。命令格式：

```bash
kubectl logs [-f] [-p] POD [-c CONTAINER]
```

命令选项详解：

```bash
-c, --container="": 容器名

-f, --follow[=false]: 指定是否持续输出日志
    --interactive[=true]: 如果为true，当需要时提示用户进行输入。默认为true
    --limit-bytes=0: 输出日志的最大字节数。默认无限制

-p, --previous[=false]: 如果为true，输出pod中曾经运行过，但目前已终止的容器的日志
    --since=0: 仅返回相对时间范围，如5s、2m或3h，之内的日志。默认返回所有日志。只能同时使用since和since-time中的一种
    --since-time="": 仅返回指定时间（RFC3339格式）之后的日志。默认返回所有日志。只能同时使用since和since-time中的一种
    --tail=-1: 要显示的最新的日志条数。默认为-1，显示所有的日志
    --timestamps[=false]: 在日志中包含时间戳
```

如果一个pod中只有一个容器，则不用指定容器名。

---





### 自我总结

---

##### 更新deploy里的定义的镜像版本

![截屏2022-09-20 16.46.06](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/%E6%88%AA%E5%B1%8F2022-09-20%2016.46.06.png)

```bash
kubectl set image deployment/sgcc-rpa-foundation sgcc-rpa-foundation=sgcc-rpa-foundation:1.1

[[或写为]]
kubectl set image deployment/sgcc-rpa-foundation *=sgcc-rpa-foundation:1.1
```



##### 调整pod副本数为4

```bash
kubectl scale deployment sgcc-rpa-foundation --replicas=4
```



##### 创建deployment是记录下每次操作所执行的命令

```bash
kubectl create -f sgcc-rpa-foundation --record
```



##### 实时查看deployment对象的状态变化

```bash
kubectl rollout status deployment/sgcc-rpa-foudation
```



##### 编辑更新资源内容

```bash
[[编辑修改deploy资源yaml]]
kubectl edit deployment/sgcc-rpa-foundation -n kube-rpa

[[编辑修改pod资源yaml]]
kubectl edit po sgcc-rpa-frontend-5f89fb7bd7-fq5h8 -n kube-rpa
```



##### 查看deployment变更的对应版本

```bash
[[查看全部变更版本信息]]
kubectl rollout history

[[查看具体版本变更详细信息]]
kubectl rollout history deployment/sgcc-rpa-foundation --revision=2
```



##### 回滚到指定版本

```bash
[[回滚到上一个版本]]
kubectl rollout undo deployment/sgcc-rpa-foundation

[[回滚到指定版本]]
kubectl rollout undo deployment/sgcc-rpa-foundation --revision=2

kubectl rollout undo deploymensgcc-rpa-foundation --to-revision=2
```



##### 暂停与恢复deployment状态以便进行多次修改只生成一个版本信息

```bash
[[暂停]]
kubectl rollout pause deployment/sgcc-rpa-foundation

[[恢复]]
kubectl rollout resume deployment/sgcc-rpa-roundation
```



##### 查看携带指定标签的pod的创建过程

```bash
kubectl get -w -l app=sgcc-rpa-foundation
```



##### 启动一个一次性pod

```bash
kubectl run -i --tty --image busybox dns-test --restart=Never --rm /bin/sh
```



##### 搜索docker镜像

```bash
docker search 镜像名
```



##### 查看docker容器的详情

```bash
docker inspect 容器名

[[过滤出容器ip地址]]
docker inspect 1f6cf |grep -i ipaddress

[[过滤网络类型]]
docker inspect 1f6cf |grep -i network
```



##### 查看api对象帮助

```bash
[[查看声明帮助]]
kubectl explain pod

[[车查看具体字段介绍]]
kubectl explain pod.spec.containers
```



##### 强制移除资源对象

```bash
kubectl delete pod test --force --grace-period=0

# --force  强制删除
[[--grace-period]]=0 设置删除宽限期为0，加速删除速度 
加参数 --force --grace-period=0，grace-period表示过渡存活期，默认30s，在删除POD之前允许POD慢慢终止其上的容器进程，从而优雅退出，0表示立即终止POD
```



##### 查看指定名称空间下所有资源

```bash
kubectl get all -n kube-rpa

[[可descride]]
[[可加-o]] wide
```



##### 滚动更新并指定升级间隔时间

```bash
[[滚动更新标签为rc名为sgcc-rpa-frontend的pod，指定文件sgcc-rpa-frontend]].yaml 升级间隔时间为30秒 
kubectl rolling-update sgcc-rpa-frontend -f sgcc-rpa-frontend2.yaml --update-period=30
[[此种方式通过指定不同yaml亦可实现回滚操作]]

[[回滚上一个版本]]
kubectl rolling-update sgcc-rpa-frontend sgcc-rpa-frontend2 --rollback
[[这种方式适合升级一半通过ctrl]]+终止更新操作后执行,默认间隔时间为1分钟，建议指定 
```



##### 切换默认工作名称空间

```bash
[[切换默认名称空间]]
kubectl config set-context --current --namespace=kube-rpa

[[切换回默认名称空间，default可不写]]
kubectl config set-context --current --namespace=
```



改变资源内定义的副本数

```bash
[[将deploy的yaml文件内的副本定义改为3]]
kubectl scale deploy sgcc-rpa-frontend --replicas=3
```



##### 修改service端口限制范围

![截屏2022-09-20 15.33.51](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/%E6%88%AA%E5%B1%8F2022-09-20%2015.33.51.png)

![](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/%E6%88%AA%E5%B1%8F2022-09-20%2015.33.42.png)

```bash
[[修改apiserver配置文件]]
vim /etc/kubernetes/apiserver

[[在KUBE_API_ARGS内添加]]
KUBE_API_ARGS="--service-node-port-range=10000-60000"
[[默认范围30000-32767]]
```



##### 手动添加svc并关联deploy

![截屏2022-09-20 16.36.46](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/%E6%88%AA%E5%B1%8F2022-09-20%2016.36.46.png)

```bash
kubectl expose deploymeny sgcc-rpa-frontend --port=80 --type=NodePort
```



##### 给节点定义标签

![截屏2022-09-20 20.14.29](https://typora-images-1306935446.cos.ap-shanghai.myqcloud.com/markdown/%E6%88%AA%E5%B1%8F2022-09-20%2020.14.29.png)

```bash
kubectl label node k8s-master app=master
```



##### 调度到指定node上

```bash
[[查看node标签]]

kubectl get nodes --show-label=true

[[给node节点打标签]]

kubectl label node k8s-slave1 nodetype=worker1

[[删除node上的节点]]

kubectl label node k8s-slavel nodetypr-

[[yaml里指定调度]]

		spec:
		  nodeSelector:
		    nodetype: worker
```

#### K8s中污点(Taint)详解及命令

##### 1.污点 ( Taint ) 的组成

```javascript
使用kubectl taint命令可以给某个Node节点设置污点，Node被设置上污点之后就和Pod之间存在了一种相斥的关系，可以让Node拒绝Pod的调度执行，甚至将Node已经存在的Pod驱逐出去
```

每个污点的组成如下：

```javascript
key=value:effect
```

每个污点有一个 key 和 value 作为污点的标签，其中 value 可以为空，eﬀect 描述污点的作用。

当前 taint eﬀect 支持如下三个选项：

```javascript
NoSchedule ：表示k8s将不会将Pod调度到具有该污点的Node上
PreferNoSchedule ：表示k8s将尽量避免将Pod调度到具有该污点的Node上
NoExecute ：表示k8s将不会将Pod调度到具有该污点的Node上，同时会将Node上已经存在的Pod驱逐出去
```

##### 2.污点的设置、查看和去除

设置污点

```javascript
kubectl taint nodes k8s-node2 check=yuanzhang:NoExecute
```

节点说明中，查找Taints字段

```javascript
kubectl describe nodes k8s-node2
```

###### 去除污点

```javascript
kubectl taint nodes k8s-node2 check:NoExecute-
```





##### 干跑一次（不真正创建资源）

```bash
kubectl create deployment myapp --image=nginx --replicas=2 --dry-run=client
```

##### 利用干跑一次获取yaml模版

```bash
kubectl create deployment myappre--image=nginx --replicas=2 --dry-run=client -o yaml

```

同理，要创建deployment也可以，service也可以！

```bash
kubectl create deployment my-deployment --image=my-image --dry-run=client -o yaml

[[当然要想直接写入文件可以重定向到文件内]]
kubectl create deployment my-deployment --image=my-image --dry-run=client -o yaml > my-deployment.yaml

kubectl create deployment nginx-deployment --image=nginx:1.16 --namespace=kube-te --replicas=2 --dry-run=client -o yaml 

[[service]]
kubectl create service clusterip my-service --tcp=80:8080 --dry-run=client -o yaml > my-service.y

[[secret]]
kubectl create secret docker-registry aliyun \
--docker-username=552408925@qq.co \
--docker-password=123456 \
--docker-server registiy.cn-huhehaote.aliyuncs.com \
--dry-run=client -o yaml
```



##### 驱逐node

```bash
kubectl drain node01 --ignore-daemonsets -- force


[[解除不可调度]]

kubectl uncordon node01
```



##### pod外执行命令作用于pod容器内

```bash
kubectl exec - it pod-nginx-new nginx -s stop
```



##### 查看service的详情

```bash
kubectl describe endpoints pod- readiness
```



##### 查看pod资源使用

```bash
kubectl top pod xxxpod
```





##### 安装完metrics-server后使用kubectl top pod可能存在如下提示

```bash
W0414 20:50:33.056703   24141 top_pod.go:140] Using json format to get metrics. Next release will switch to protocol-buffers, switch early by passing --use-protocol-buffers flag

[[并不影响使用，只是提示这是来自]] Kubernetes 组件 "top_pod.go" 的日志消息。它表明该组件当前正在使用 JSON 格式来检索度量数据，但在下一个版本中，将切换到 Protocol Buffers 格式。通过传递 "--use-protocol-buffers" 标志可以提前进行切换

[[使用时添加--use-protocol-buffers可屏蔽此提示]]

kubectl top pod --use-protocol-buffers
```





##### 附带安装metrics-server时的注意事项博客



```bash
[[安装官方deployment]]
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```



```bash
[[按图中添加下面选项]]

hostNetwork: true
image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6
args:
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP
```



![img](https://markdown-picgo-typora.oss-cn-beijing.aliyuncs.com/markdown202304142215715.png)



```yaml
[[徐亮伟已修改的deployment]]

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
- apiGroups:
  - metrics.k8s.io
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
        image: registry.cn-huhehaote.aliyuncs.com/oldxu3957/metrics-server:v0.6.1
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /livez
            port: https
            scheme: HTTPS
          periodSeconds: 10
        name: metrics-server
        ports:
        - containerPort: 4443
          name: https
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /readyz
            port: https
            scheme: HTTPS
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        volumeMounts:
        - mountPath: /tmp
          name: tmp-dir
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
      - emptyDir: {}
        name: tmp-dir
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100
```





##### 命令行创建hpa

```bash
kubectl autoscale deployment nginx-vpn  --min=1 --max=10 --cpu-percent=50

kubectl autoscale deployment nginx-vpn  --min=1 --max=10 --cpu-percent=50  --dry-run=client -o yaml
```

##### yaml创建hpa

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-vpn
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-vpn
  targetCPUUtilizationPercentage: 50
```





##### 实时监视kubec get xxx（某一条命命令）的状态变化

```bash
watch -n1  kubectl get pod  -l app=nginx-vpn

命令"watch -n1 kubectl get pod -l app=nginx-vpn"用于持续监视带有"label=app=nginx-vpn"标签的Kubernetes pod的状态。

[[以下是该命令的每个部分的含义：]]
"watch"是一个Linux命令，它会重复运行一个命令，并实时在终端上显示其输出。
"-n1"指定命令应该运行的间隔时间。在这种情况下，它被设置为1秒。
"kubectl"是Kubernetes的命令行工具。
"get pod"是kubectl命令的一部分，用于获取pod的信息。
"-l app=nginx-vpn"是一个过滤器，用于只显示带有"label=app=nginx-vpn"标签的pod的信息。


kubectl get pod  -l app=nginx-vpn -w
 
命令"kubectl get pod -l app=nginx-vpn -w"用于监视带有"label=app=nginx-vpn"标签的Kubernetes pods的实时状态。

[[以下是该命令的每个部分的含义：]]

"kubectl"是Kubernetes的命令行工具。
"get pod"是kubectl命令的一部分，用于获取pod的信息。
"-l app=nginx-vpn"是一个过滤器，用于只显示带有"label=app=nginx-vpn"标签的pod的信息。
"-w"告诉kubectl命令保持运行并实时更新输出，直到您手动停止命令（使用CTRL-C）。
因此，当您运行此命令时，您将看到一个实时更新的列表，其中包含带有"label=app=nginx-vpn"标签的pod的当前状态。如果有任何更改（例如pod启动或停止），列表将自动更新以反映这些更改。
```



##### 模拟负载压力

```bash
while sleep 0.01; do curl http://10.96.79.138; done
```



```bash
kubectl run -i --tty temp -image oldxu3957/mqtools:latest

您提供的命令正在使用 ，这是一个用于管理 Kubernetes 集群的命令行工具。该命令似乎正在使用该命令在群集中创建新部署。kubectlrun

以下是命令的每个部分的含义：

kubectl：用于管理 Kubernetes 集群的命令行工具。
run：用于创建新部署或作业的命令。kubectl
-tty：为 Pod 中的容器分配终端。
temp：正在创建的新部署的名称。
-image：指定要用于部署的容器映像。
oldxu3957/mqtools:latest：用于部署的容器映像的名称和版本。在本例中，图像带有标记。oldxu3957/mqtoolslatest
总体而言，此命令会创建一个使用容器映像命名的新部署，并为容器中的容器分配终端。tempoldxu3957/mqtools:latest
```





### 处理K8S集群pod不能访问其他service的问题

当时K8S环境想用来设计部署微服务这块的架构才发现的这个问题

我的K8S集群是使用kubeadm安装的，当时也是跟着网上教程走的，并没有注意网络路由使用的iptables规则

现在出现pod不能ping通service或者ping通CLUSTER-IP的问题，导致如果我再集群里部署注册中心，并不能正常使用的问题

以下为把iptables变更为ipvs模块的操作

修改网络模式ipvs
1、修改kube-proxy
再master上执行：kubectl edit cm kube-proxy -n kube-system  （如果是高可用，在第一个master上执行）
mode “ipvs”  （找到kind: KubeProxyConfiguration这一项。。。他下面的第二行就是这个mode）

```bash
kubectl edit cm kube-proxy -n kube-system
```

2、添加ipvs模块

```bash
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
\#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
```

3、添加权限并生效

```bash
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

4、重启kube-proxy

```bash
kubectl get pod -n kube-system | grep kube-proxy |awk '{system("kubectl delete pod "$1" -n kube-system")}'
```

5、查看kube-proxy启动日志，确认是否为ipvs

```bash
kubectl logs -n kube-system kube-proxy-ff74q
[[这个pod名称使用命令kubectl]] get pod -n kube-system查出来
```

```bash
kubectl logs -n kube-system kube-proxy-pqbwv
```

6、验证是否可以pod访问service 

一切没有问题后，pod上的应用就能正常注册了

---



##### 查看自己ecs公网地址的方法

```bash
curl members.3322.org/dyndns/getip
curl icanhazip.com  
curl ifconfig.me  
curl curlmyip.com  
curl ip.appspot.com  
curl ipinfo.io/ip  
curl ipecho.net/plain  
curl www.trackip.net/i  
```



##### bashboard浏览器打不开解决方法

  在浏览器空白地方输入`thisisunsafe`就可以了



