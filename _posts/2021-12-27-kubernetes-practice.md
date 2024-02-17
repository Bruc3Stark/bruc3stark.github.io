---
layout: mypost
title: 每天5分钟玩转Kubernetes实战
categories: [Kubernetes]
---

### Chapter 1：先把Kubernetes跑起来

#### 1.3 部署应用

```shell
kubectl create deployment kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1
```



---

#### 1.4 访问应用

```shell
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
```



---

#### 1.5 Scale 应用

```shell
kubectl scale deployments/kubernetes-bootcamp --replicas=3
```



---

#### 1.6 滚动更新

```shell
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
```



### Chapter 4：Kubernetes 架构

#### 4.4 用例子把它们串起来

```shell
kubectl create deployment httpd-app --image=httpd

kubectl scale deployment/httpd-app --replicas=2
```



### Chapter 5：运行应用

#### 5.1.1 运行 Deployment

```shell
kubectl create deployment nginx-deployment --image=nginx:1.7.9

kubectl scale deployment/nginx-deployment --replicas=2
```



---

#### 5.1.3 Deployment 配置文件简介

```yaml
apiVersion: apps/v1                 # Api 接口版本
kind: Deployment                    # 定义控制器
metadata:
  name: nginx-deployment            # deployment 名称
spec:
  replicas: 2                       # 副本数
  selector:                         # 选择标签
    matchLabels:                    # 定义标签
      app: nginx-deployment         # 标签名 
  template:                         # pod 容器
    metadata:
      labels:                       # 定义标签
        app: nginx-deployment       # 标签名
    spec:
      containers:
      - name: nginx                 # 容器名
        image: nginx:1.7.9          # 容器镜像
```



---

#### 5.3.1 Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob
spec:
  template:
    metadata:
      name: myjob
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["echo", "hello k8s job! "]
      restartPolicy: Never
```

#### 5.3.2 Job 的并行性

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob
spec:
  completions: 6
  parallelism: 2
  template:
    metadata:
      name: myjob
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["echo", "hello k8s job! "]
      restartPolicy: OnFailure
```

#### 5.3.3 定时 Job

```yaml
apiVersion: batch/v2alpha1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: '*/1 * * * *'
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command: ["echo", "hello k8s job! "]
          restartPolicy: OnFailure
```

报错：

```shell
error: error validating "cronjob.yml": error validating data: [ValidationError(CronJob.spec.jobTemplate.spec): unknown field "containers" in io.k8s.api.batch.v1.JobSpec, ValidationError(CronJob.spec.jobTemplate.spec): unknown field "restartPolicy" in io.k8s.api.batch.v1.JobSpec, ValidationError(CronJob.spec.jobTemplate.spec): missing required field "template" in io.k8s.api.batch.v1.JobSpec]; if you choose to ignore these errors, turn validation off with --validate=false
```



* 问题分析：

Kubernetes 默认没有 enable CronJob 功能，需要在 kube-apiserver 中加入这个功能

```shell
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```



修改内容如下：

![5](5.png)

```shell
# 重启 kubelet 服务
systemctl restart kubelet

# 验证是否成功
kubectl api-versions | grep batch/v2
```



### Chapter 6：通过 Service 访问 Pod

#### 6.1 创建 Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    run: httpd
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
```



---

#### 6.3 DNS 访问 Service

```shell
kubectl get deployment -n kube-system

# 在临时的 busybox pod 中验证 DNS 有效性
kubectl run busybox --rm -ti --image=busybox /bin/sh

/ # wget httpd-svc.default:8080
/ # nslookup httpd-svc  # 用 nslookup 查看 httpd-svc 的 DNS 信息

/ # wget httpd2-svc.kube-public:8080
```



---

#### 6.4 外网如何访问 Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  type: NodePort
  selector:
    run: httpd
  ports:
  - protocol: TCP
    nodePort: 30000
    port: 8080
    targetPort: 80
```



```shell
kubectl get service httpd-svc

curl ${IP}:30000  # 30000端口已打开
```



### Chapter 7：Rolling Update

#### 7.1 实践

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 3
  selector:
    matchLabels:
      run: httpd
  template:
    metadata:
      labels:
        run: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:2.2.31
        ports:
        - containerPort: 80
```



```shell
kubectl get deployment httpd -owide

kubectl get replicaset -owide

kubectl describe deployment httpd
```



---

#### 7.2 回滚

```shell
kubectl apply -f httpd.v1.yml --record  # --record 作用是将命令记录到 revision 记录中

kubectl apply -f httpd.v2.yml --record

kubectl apply -f httpd.v3.yml --record

kubectl rollout history deployment httpd

kubectl rollout undo deployment httpd --to-revision=1
```



### Chapter 8：Health Check

#### 8.1 默认的健康检查

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: healthcheck
  name: healthcheck
spec:
  restartPolicy: OnFailure
  containers:
  - name: healthcheck
    image: busybox
    args:
    - /bin/sh
    - -c
    - sleep 10; exit 1
```



```shell
kubectl get pod healthcheck
```



---

#### 8.2 Liveness 探测

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness
spec:
  restartPolicy: OnFailure
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 10
      periodSeconds: 5
```



```shell
kubectl describe pod liveness

kubectl get pod liveness
```



---

#### Readiness 探测

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness
spec:
  restartPolicy: OnFailure
  containers:
  - name: readiness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 10
      periodSeconds: 5
```



```shell
kubectl get pod readiness
```



---

#### 8.4 Health Check 在 Scale Up 中的应用

<font color=red>TODO：未能理解如何创建 MySQL 测试用例</font>



---

#### 8.5 Health Check 在滚动更新中的应用

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 10
  selector:
    matchLabels:
      run: app
  template:
    metadata:
      labels:
        run: app
    spec:
      containers:
      - name: app
        image: busybox
        args:
        - /bin/sh
        - -c
        - sleep 10; touch /tmp/healthy; sleep 30000
        readinessProbe:
          exec:
            command:
            - cat
            - /tmp/healthy
          initialDelaySeconds: 10
          periodSeconds: 5
```



```shell
kubectl apply -f app.v1.yml --record

kubectl get deployment app -owide
```



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 10
  selector:
    matchLabels:
      run: app
  template:
    metadata:
      labels:
        run: app
    spec:
      containers:
      - name: app
        image: busybox
        args:
        - /bin/sh
        - -c
        - sleep 3000
        readinessProbe:
          exec:
            command:
            - cat
            - /tmp/healthy
          initialDelaySeconds: 10
          periodSeconds: 5
```



```shell
kubectl apply -f app.v2.yml --record

kubectl get deployment app -owide

kubectl describe deployment app

kubectl rollout history deployment app

kubectl rollout undo deployment app --to-revision=1  # 回滚
```



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  strategy:
    rollingUpdate:
      maxSurge: 35%
      maxUnavailable: 35%
  replicas: 10
  selector:
    matchLabels:
      run: app
  template:
    metadata:
      labels:
        run: app
    spec:
      containers:
      - name: app
        image: busybox
        args:
        - /bin/sh
        - -c
        - sleep 3000
        readinessProbe:
          exec:
            command:
            - cat
            - /tmp/healthy
          initialDelaySeconds: 10
          periodSeconds: 5
```




