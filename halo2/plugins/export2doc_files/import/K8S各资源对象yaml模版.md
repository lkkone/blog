- ServiceAccount（服务账号）：
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
```
- Role（角色）：
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: my-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```
- RoleBinding（角色绑定）：
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-role-binding
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: my-namespace
roleRef:
  kind: Role
  name: my-role
  apiGroup: rbac.authorization.k8s.io
```
- Namespace（命名空间）：
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```
- NetworkPolicy（网络策略）：
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: database
    ports:
    - protocol: TCP
      port: 3306
```
- ServiceMonitor（服务监控）：
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-service-monitor
  labels:
    release: prometheus-operator
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
```
- Deployment：
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:latest
        ports:
        - containerPort: 80
```
- StatefulSet（有状态副本集）：
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-statefulset
spec:
  serviceName: my-service
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:latest
        ports:
        - containerPort: 80
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
- DaemonSet（守护进程集）：
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-daemonset
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:latest
        ports:
        - containerPort: 80
```
- HorizontalPodAutoscaler（水平自动伸缩器）：
```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: my-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```
- StorageClass（存储类）：
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-storage-class
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  zone: us-west-2a
```
- PersistentVolumeClaim（持久卷声明）：
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```
- PersistentVolume：
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /path/to/my/data
```
- Pod（Pod）：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80
```
- Secret（密钥/密码）：
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: dXNlcm5hbWU=
  password: cGFzc3dvcmQ=
```
- CronJob（定时作业）：
```yanl
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: my-container
            image: busybox
            command: ["echo", "Hello, Kubernetes!"]
          restartPolicy: OnFailure
```
- Job（作业）：
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  template:
    spec:
      containers:
      - name: my-container
        image: busybox
        command: ["echo", "Hello, Kubernetes!"]
      restartPolicy: Never
```
- Ingress（入口）：
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```
- Service（服务发现）：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```
- PodSecurityPolicy（Pod 安全策略）：
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: my-psp
spec:
  privileged: false
  hostNetwork: false
  allowPrivilegeEscalation: false
  # 其他规则和限制...
```
- LimitRange（资源限制范围）：
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: my-limit-range
spec:
  limits:
  - type: Container
    defaultRequest:
      cpu: 100m
      memory: 256Mi
    defaultLimit:
      cpu: 1
      memory: 1Gi
    # 其他资源限制...
```
- ValidatingWebhookConfiguration（验证 Webhook 配置）：
```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: my-webhook-config
webhooks:
- name: my-webhook
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: ["apps", ""]
    apiVersions: ["v1"]
    resources: ["deployments"]
  clientConfig:
    service:
      name: my-webhook-service
      namespace: my-namespace
    caBundle: <base64-encoded-ca-cert>
```
- ResourceQuota（资源配额）：
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 4Gi
    limits.cpu: "8"
    limits.memory: 8Gi
```
- Node Affinity（节点亲和性）：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: key1
            operator: In
            values:
            - value1
```
- Pod Affinity（Pod 亲和性）：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: key1
            operator: In
            values:
            - value1
        topologyKey: kubernetes.io/hostname
```
- Inter-Pod Affinity（跨 Pod 亲和性）：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - my-app
        topologyKey: kubernetes.io/hostname
```
- Node Anti-Affinity（节点反亲和性）：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    nodeAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: key1
            operator: In
            values:
            - value1
```
- Pod Anti-Affinity（Pod 反亲和性）：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: key1
            operator: In
            values:
            - value1
        topologyKey: kubernetes.io/hostname
```
- Inter-Pod Anti-Affinity（跨 Pod 反亲和性）：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - my-app
        topologyKey: kubernetes.io/hostname
```
- 定义一个带有污点的节点：
```yaml
apiVersion: v1
kind: Node
metadata:
  name: my-node
spec:
  taints:
  - key: key1
    value: value1
    effect: NoSchedule
```
- 在 Pod 中设置容忍度以容忍污点：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  tolerations:
  - key: key1
    operator: Equal
    value: value1
    effect: NoSchedule
```
