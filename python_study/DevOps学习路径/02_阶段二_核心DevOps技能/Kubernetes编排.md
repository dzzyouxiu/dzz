# Kubernetes编排

## 一、Kubernetes基础

### 1. Kubernetes简介
- **Kubernetes**：开源的容器编排平台
- **特点**：
  - 自动容器部署和复制
  - 自动容器重启和故障转移
  - 服务发现和负载均衡
  - 自动缩放
  - 滚动更新
  - 配置管理

### 2. Kubernetes核心概念

#### 2.1 集群架构
- **控制平面**：管理集群
  - kube-apiserver：API服务器
  - etcd：分布式键值存储
  - kube-scheduler：调度器
  - kube-controller-manager：控制器管理器
  - cloud-controller-manager：云提供商集成
- **节点**：运行容器的机器
  - kubelet：节点代理
  - kube-proxy：网络代理
  - 容器运行时：Docker、containerd、CRI-O

#### 2.2 核心对象
- **Pod**：最小部署单元，包含一个或多个容器
- **Service**：稳定的网络端点，用于访问Pod
- **Deployment**：管理Pod的声明式更新
- **ReplicaSet**：确保指定数量的Pod运行
- **ConfigMap**：存储配置数据
- **Secret**：存储敏感数据
- **Namespace**：逻辑隔离的集群
- **PersistentVolume**：持久化存储
- **PersistentVolumeClaim**：请求存储资源
- **StatefulSet**：管理有状态应用
- **DaemonSet**：在每个节点上运行一个Pod
- **Job**：执行一次性任务
- **CronJob**：定期执行任务

### 3. Kubernetes网络模型
- **Pod网络**：每个Pod有唯一的IP地址
- **服务发现**：通过Service名称访问
- **网络策略**：控制Pod间通信
- **CNI**：容器网络接口，如Flannel、Calico、Cilium

## 二、Kubernetes安装

### 1. 本地开发环境

#### 1.1 Minikube
- **特点**：单节点Kubernetes集群
- **安装**：
  ```bash
  # Linux
  curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
  sudo install minikube-linux-amd64 /usr/local/bin/minikube
  
  # macOS
  brew install minikube
  
  # Windows
  choco install minikube
  ```
- **启动**：
  ```bash
  minikube start
  ```
- **访问**：
  ```bash
  minikube dashboard
  ```

#### 1.2 Kind
- **特点**：使用Docker容器运行Kubernetes集群
- **安装**：
  ```bash
  # Linux/macOS
  GO111MODULE="on" go get sigs.k8s.io/kind@v0.17.0
  
  # Windows
  choco install kind
  ```
- **创建集群**：
  ```bash
  kind create cluster
  ```

### 2. 生产环境

#### 2.1 kubeadm
- **特点**：官方推荐的安装工具
- **安装步骤**：
  1. 安装Docker或containerd
  2. 安装kubeadm、kubelet、kubectl
  3. 初始化控制平面
  4. 加入工作节点

- **基本命令**：
  ```bash
  # 初始化控制平面
  kubeadm init --pod-network-cidr=192.168.0.0/16
  
  # 配置kubectl
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  
  # 安装网络插件
  kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
  
  # 加入工作节点
  kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash <hash>
  ```

#### 2.2 托管Kubernetes服务
- **AWS EKS**：Amazon Elastic Kubernetes Service
- **Azure AKS**：Azure Kubernetes Service
- **Google GKE**：Google Kubernetes Engine
- **阿里云ACK**：Alibaba Cloud Container Service for Kubernetes
- **腾讯云TKE**：Tencent Kubernetes Engine

### 3. kubectl配置

#### 3.1 安装kubectl
- **Linux**：
  ```bash
  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
  ```

- **macOS**：
  ```bash
  brew install kubectl
  ```

- **Windows**：
  ```bash
  choco install kubernetes-cli
  ```

#### 3.2 配置kubectl
- **查看集群**：
  ```bash
  kubectl cluster-info
  ```

- **查看节点**：
  ```bash
  kubectl get nodes
  ```

- **配置上下文**：
  ```bash
  # 查看上下文
  kubectl config get-contexts
  
  # 切换上下文
  kubectl config use-context <context-name>
  ```

## 三、Kubernetes核心操作

### 1. Pod操作

#### 1.1 创建Pod
- **命令行创建**：
  ```bash
  kubectl run nginx --image=nginx
  ```

- **YAML创建**：
  ```bash
  cat > pod.yaml << 'EOF'
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
  spec:
    containers:
    - name: nginx
      image: nginx
      ports:
      - containerPort: 80
  EOF
  
  kubectl apply -f pod.yaml
  ```

#### 1.2 查看Pod
```bash
# 查看所有Pod
kubectl get pods

# 查看Pod详情
kubectl describe pod nginx

# 查看Pod日志
kubectl logs nginx

# 实时查看日志
kubectl logs -f nginx

# 进入Pod
kubectl exec -it nginx -- /bin/bash
```

#### 1.3 删除Pod
```bash
kubectl delete pod nginx

# 删除所有Pod
kubectl delete pods --all
```

### 2. Deployment操作

#### 2.1 创建Deployment
- **命令行创建**：
  ```bash
  kubectl create deployment nginx --image=nginx --replicas=3
  ```

- **YAML创建**：
  ```bash
  cat > deployment.yaml << 'EOF'
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.19.0
          ports:
          - containerPort: 80
  EOF
  
  kubectl apply -f deployment.yaml
  ```

#### 2.2 查看Deployment
```bash
# 查看所有Deployment
kubectl get deployments

# 查看Deployment详情
kubectl describe deployment nginx

# 查看ReplicaSet
kubectl get replicasets
```

#### 2.3 更新Deployment
```bash
# 更新镜像
kubectl set image deployment/nginx nginx=nginx:1.19.1

# 滚动更新策略
kubectl patch deployment nginx -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'

# 查看滚动更新状态
kubectl rollout status deployment nginx

# 回滚更新
kubectl rollout undo deployment nginx

# 查看历史版本
kubectl rollout history deployment nginx

# 回滚到特定版本
kubectl rollout undo deployment nginx --to-revision=2
```

#### 2.4 扩缩容
```bash
# 手动扩缩容
kubectl scale deployment nginx --replicas=5

# 自动扩缩容（HPA）
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80
```

### 3. Service操作

#### 3.1 创建Service
- **命令行创建**：
  ```bash
  kubectl expose deployment nginx --port=80 --type=NodePort
  ```

- **YAML创建**：
  ```bash
  cat > service.yaml << 'EOF'
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx
  spec:
    selector:
      app: nginx
    ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
    type: NodePort
  EOF
  
  kubectl apply -f service.yaml
  ```

#### 3.2 查看Service
```bash
# 查看所有Service
kubectl get services

# 查看Service详情
kubectl describe service nginx

# 查看Service端点
kubectl get endpoints nginx
```

#### 3.3 Service类型
- **ClusterIP**：默认类型，集群内部访问
- **NodePort**：暴露端口到节点
- **LoadBalancer**：使用云提供商的负载均衡器
- **ExternalName**：通过CNAME记录转发

### 4. 配置管理

#### 4.1 ConfigMap
- **创建ConfigMap**：
  ```bash
  # 从文件创建
  kubectl create configmap app-config --from-file=config.yaml
  
  # 从字面值创建
  kubectl create configmap app-config --from-literal=DB_HOST=localhost --from-literal=DB_PORT=5432
  
  # YAML创建
  cat > configmap.yaml << 'EOF'
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  data:
    DB_HOST: localhost
    DB_PORT: "5432"
    app.properties: |
      server.port=8080
      spring.profiles.active=prod
  EOF
  
  kubectl apply -f configmap.yaml
  ```

- **使用ConfigMap**：
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  spec:
    template:
      spec:
        containers:
        - name: app
          image: app
          envFrom:
          - configMapRef:
              name: app-config
          volumeMounts:
          - name: config-volume
            mountPath: /app/config
        volumes:
        - name: config-volume
          configMap:
            name: app-config
  ```

#### 4.2 Secret
- **创建Secret**：
  ```bash
  # 从字面值创建
  kubectl create secret generic app-secret --from-literal=DB_PASSWORD=secret
  
  # 从文件创建
  echo -n 'admin' > username.txt
  echo -n 'secret' > password.txt
  kubectl create secret generic app-secret --from-file=username.txt --from-file=password.txt
  
  # YAML创建（base64编码）
  cat > secret.yaml << 'EOF'
  apiVersion: v1
  kind: Secret
  metadata:
    name: app-secret
  type: Opaque
  data:
    username: YWRtaW4=
    password: c2VjcmV0
  EOF
  
  kubectl apply -f secret.yaml
  ```

- **使用Secret**：
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  spec:
    template:
      spec:
        containers:
        - name: app
          image: app
          envFrom:
          - secretRef:
              name: app-secret
          volumeMounts:
          - name: secret-volume
            mountPath: /app/secret
            readOnly: true
        volumes:
        - name: secret-volume
          secret:
            secretName: app-secret
  ```

### 5. 存储操作

#### 5.1 PersistentVolume
- **创建PV**：
  ```bash
  cat > pv.yaml << 'EOF'
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv0001
  spec:
    capacity:
      storage: 1Gi
    accessModes:
    - ReadWriteOnce
    persistentVolumeReclaimPolicy:
      Retain
    hostPath:
      path: /mnt/data
  EOF
  
  kubectl apply -f pv.yaml
  ```

#### 5.2 PersistentVolumeClaim
- **创建PVC**：
  ```bash
  cat > pvc.yaml << 'EOF'
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: my-pvc
  spec:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 500Mi
  EOF
  
  kubectl apply -f pvc.yaml
  ```

- **使用PVC**：
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  spec:
    template:
      spec:
        containers:
        - name: app
          image: app
          volumeMounts:
          - name: data-volume
            mountPath: /app/data
        volumes:
        - name: data-volume
          persistentVolumeClaim:
            claimName: my-pvc
  ```

## 四、Helm包管理

### 1. Helm简介
- **Helm**：Kubernetes的包管理工具
- **Chart**：预配置的Kubernetes资源包
- **Repository**：存储Chart的地方
- **Release**：Chart的运行实例

### 2. Helm安装

#### 2.1 安装Helm
- **Linux/macOS**：
  ```bash
  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
  chmod 700 get_helm.sh
  ./get_helm.sh
  ```

- **Windows**：
  ```bash
  choco install kubernetes-helm
  ```

#### 2.2 配置Helm仓库
```bash
# 添加官方仓库
helm repo add stable https://charts.helm.sh/stable

# 添加其他仓库
helm repo add bitnami https://charts.bitnami.com/bitnami

# 更新仓库
helm repo update

# 查看仓库
helm repo list
```

### 3. Helm基本操作

#### 3.1 搜索Chart
```bash
# 搜索Chart
helm search repo nginx

# 搜索所有仓库
helm search hub nginx
```

#### 3.2 安装Chart
```bash
# 安装Chart
helm install my-nginx bitnami/nginx

# 指定版本安装
helm install my-nginx bitnami/nginx --version 9.3.0

# 使用values文件
helm install my-nginx bitnami/nginx -f values.yaml

# 查看Release
helm list
```

#### 3.3 升级和回滚
```bash
# 升级Release
helm upgrade my-nginx bitnami/nginx --set service.type=NodePort

# 查看Release历史
helm history my-nginx

# 回滚Release
helm rollback my-nginx 1
```

#### 3.4 删除Release
```bash
helm uninstall my-nginx
```

### 4. 自定义Chart

#### 4.1 创建Chart
```bash
# 创建Chart
helm create my-chart

# 查看Chart结构
ls -la my-chart/
```

#### 4.2 构建和安装
```bash
# 构建Chart
helm package my-chart

# 安装本地Chart
helm install my-app ./my-chart-0.1.0.tgz
```

## 五、Kubernetes最佳实践

### 1. 应用部署
- **使用Deployment**：管理无状态应用
- **使用StatefulSet**：管理有状态应用
- **使用DaemonSet**：部署节点级服务
- **使用Job/CronJob**：处理批处理任务

### 2. 配置管理
- **使用ConfigMap**：存储配置数据
- **使用Secret**：存储敏感信息
- **使用环境变量**：注入配置到容器
- **使用卷挂载**：挂载配置文件

### 3. 存储管理
- **使用PVC**：抽象存储细节
- **使用StorageClass**：动态配置存储
- **使用PV**：管理持久化存储
- **考虑备份**：定期备份数据

### 4. 网络和服务
- **使用Service**：稳定的访问端点
- **使用Ingress**：管理外部访问
- **使用NetworkPolicy**：增强安全性
- **考虑服务网格**：如Istio、Linkerd

### 5. 安全最佳实践
- **使用RBAC**：基于角色的访问控制
- **使用PodSecurityPolicy**：限制Pod权限
- **使用Namespace**：隔离不同应用
- **限制容器权限**：使用非root用户
- **扫描镜像**：检测安全漏洞
- **使用Secrets**：安全存储敏感信息

### 6. 监控和日志
- **使用Prometheus**：监控集群和应用
- **使用Grafana**：可视化监控数据
- **使用ELK**：收集和分析日志
- **使用Liveness/Readiness探针**：监控应用健康状态

### 7. 高可用性
- **多副本**：确保应用可用性
- **反亲和性**：避免Pod集中在一个节点
- **健康检查**：及时发现和恢复故障
- **自动扩缩容**：根据负载调整副本数

## 六、推荐学习资源

### 书籍
- 《Kubernetes权威指南》
- 《Kubernetes实战》
- 《Kubernetes in Action》
- 《Helm权威指南》

### 在线资源
- [Kubernetes官方文档](https://kubernetes.io/docs/)
- [Helm官方文档](https://helm.sh/docs/)
- [Kubernetes博客](https://kubernetes.io/blog/)
- [Kubernetes GitHub](https://github.com/kubernetes/kubernetes)

### 视频教程
- [Kubernetes入门教程](https://kubernetes.io/docs/tutorials/)
- [Kubernetes Mastery](https://www.udemy.com/course/kubernetes-mastery/)
- [Helm教程](https://helm.sh/docs/intro/quickstart/)

### 实践项目
- [Kubernetes示例](https://github.com/kubernetes/examples)
- [Helm Charts](https://github.com/helm/charts)
- [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)

## 七、实践项目

### 项目一：部署Web应用
- 创建Deployment
- 创建Service
- 配置Ingress
- 部署到Kubernetes

### 项目二：微服务架构
- 部署多个微服务
- 配置服务发现
- 实现服务间通信
- 配置负载均衡

### 项目三：CI/CD流水线
- 集成Jenkins/GitLab CI
- 实现自动化构建和部署
- 配置滚动更新
- 实现监控和告警

## 八、总结

通过本章节的学习，你应该已经掌握了以下Kubernetes技能：

1. **Kubernetes基础**：
   - 核心概念和架构
   - 集群安装和配置
   - kubectl命令行工具

2. **核心操作**：
   - Pod、Deployment、Service管理
   - 配置管理（ConfigMap、Secret）
   - 存储管理（PV、PVC）

3. **Helm包管理**：
   - Chart安装和管理
   - 自定义Chart
   - Release管理

4. **最佳实践**：
   - 应用部署策略
   - 安全配置
   - 监控和高可用

Kubernetes已经成为容器编排的事实标准，掌握Kubernetes将使你能够更好地管理和扩展容器化应用。