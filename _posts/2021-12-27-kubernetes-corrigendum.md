---
layout: mypost
title: 每天5分钟玩转Kubernetes勘误
categories: [Kubernetes]
---

### Chapter 1：先把 Kubernetes 跑起来

#### 1.3 部署应用

```shell
kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port 8080
```

* 问题：<font color=red>发现只创建 pod，并未创建部署</font>



修改为：

```shell
kubectl create deployment kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1
```



### Chapter 4：Kubernetes 架构

#### 4.4 用例子把它们串起来

```shell
kubectl run httpd-app --image=httpd --replicas=2
```

* 问题：<font color=red>在 Kubernetes v1.18.0以后，-–replicas 已弃用 ,推荐用 deployment 创建 pods</font>

  ```shell
  Flag --replicas has been deprecated, has no effect and will be removed in the future.
  ```



修改为：

```shell
kubectl create deployment httpd-app --image=httpd

kubectl scale deployment/httpd-app --replicas=2
```



### Chapter 5：运行应用

#### 5.1.1 运行 Deployment

```shell
kubectl run nginx-deployment --image=nginx:1.7.9 --replicas=2
```

* 问题：<font color=red>发现只创建 pod，并未创建部署</font>



修改为：

```shell
kubectl create deployment nginx-deployment --image=nginx:1.7.9

kubectl scale deployment/nginx-deployment --replicas=2
```



---

#### 5.1.3 Deployment 配置文件简介

通过 yaml 文件部署应用（nginx-deployment.yaml）：

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: web_server
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
```

* 问题：<font color=red>版本升级导致 yaml 语法变化</font>

  ```shell
  error: unable to recognize "nginx-deployment.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"
  ```



修改为：

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

* 注意：spec 中必须添加副本标签与 Deployment 控制器进行匹配<font color=red>（添加 selector 标签参数，与 pod 和 deployment 相匹配）</font>



---

#### 5.1.5 Failover（没有解决）

在关闭 node2 后节点状态变为 NotReady ，与书上一致

![1](1.png)



<font color=red>但 node2 上跑的 pod 状态并没有变成 Unknown ，也并没有在 node1 上重新创建新的 pod</font>，与书上不一致

![2](2.png)



并且在使用 kubectl 删除原部署并重新启动部署后，<font color=red>node2 的 pod 无法正常删除</font>

![3](3.png)



直到重启 node2，才成功删除原 pod

![4](4.png)



---

#### 5.2.3 运行自己的 DaemonSet

通过 yml 文件部署 DaemonSet（node-exporter.yml）：

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: node-exporter-daemonset
spec:
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      hostNetwork: true
      containers:
      - name: node-exporter
        image: prom/node-exporter
        imagePullPolicy: IfNotPresent
        command:
        - /bin/node_exporter
        - --path.procfs
        - /host/proc
        - --path.sysfs
        - /host/sys
        - --collector.filesystem.ignored-mount-points
        - ^/(sys|proc|dev|host|etc)($|/)
        volumeMounts:
        - name: proc
          mountPath: /host/proc
        - name: sys
          mountPath: /host/sys
        - name: root
          mountPath: /rootfs
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      - name: root
        hostPath:
          path: /
```

* 问题：<font color=red>版本升级导致 yaml 语法变化</font>

  ```shell
  error: unable to recognize "node_exporter.yml": no matches for kind "DaemonSet" in version "extensions/v1beta1"
  ```



修改 yaml 文件为：

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter-daemonset
spec:
  selector:
    matchLabels:
      app: node-exporter-daemonset
  template:
    metadata:
      labels:
        app: node-exporter-daemonset
    spec:
      hostNetwork: true
      containers:
      - name: node-exporter
        image: prom/node-exporter
        imagePullPolicy: IfNotPresent
        command:
        - /bin/node_exporter
        - --path.procfs
        - /host/proc
        - --path.sysfs
        - /host/sys
        - --collector.filesystem.ignored-mount-points
        - ^/(sys|proc|dev|host|etc)($|/)
        volumeMounts:
        - name: proc
          mountPath: /host/proc
        - name: sys
          mountPath: /host/sys
        - name: root
          mountPath: /rootfs
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      - name: root
        hostPath:
          path: /
```
---
#### 5.3 Job

```shell
kubectl get pod --show-all
```
* 问题：<font color=red>版本升级导致 kubectl 参数变化，`--show-all`参数被移除</font>

  ```shell
  Error: unknown flag: --show-all
  ```



使用`-owide`参数代替

```shell
kubectl get pod -owide
```



### Chapter 6：通过 Service 访问 Pod

#### 6.1 创建 Service

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 3
  template:
    metadata:
      labels:
        run: httpd
    spec:
      containers:
      - name: httpd
        image: httpd
        ports:
        - containerPort: 80
```

* 问题：<font color=red>版本升级导致 yaml 语法变化</font>（同 5.1.3 Deployment 配置文件简介）

  ```shell
  error: unable to recognize "httpd.yml": no matches for kind "Deployment" in version "apps/v1beta1"
  
  error: error validating "httpd.yml": error validating data: ValidationError(Deployment.spec): missing required field "selector" in io.k8s.api.apps.v1.DeploymentSpec; if you choose to ignore these errors, turn validation off with --validate=false
  ```



修改 yaml 文件为：

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
        image: httpd
        ports:
        - containerPort: 80
```



---

#### 6.3 DNS 访问 Service

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: httpd2
  namespace: kube-public
spec:
  replicas: 3
  template:
    metadata:
      labels:
        run: httpd2
    spec:
      containers:
      - name: httpd2
        image: httpd
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: httpd2-svc
  namespace: kube-public
spec:
  selector:
    run: httpd2
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
```

* 问题：<font color=red>版本升级导致 yaml 语法变化</font>（同上）

```shell
error: unable to recognize "httpd2.yml": no matches for kind "Deployment" in version "apps/v1beta1"
```



修改 yaml 文件为：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd2
  namespace: kube-public
spec:
  replicas: 3
  selector:
    matchLabels:
      run: httpd2
  template:
    metadata:
      labels:
        run: httpd2
    spec:
      containers:
      - name: httpd2
        image: httpd
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: httpd2-svc
  namespace: kube-public
spec:
  selector:
    run: httpd2
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
```



### Chapter 8：Health Check

#### 8.5 Health Check 在滚动更新中的应用

```yaml
apiVersion: app/v1beta1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 10
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

* 问题：<font color=red>版本升级导致 yaml 语法变化</font>（同 5.1.3 Deployment 配置文件简介）

```shell
error: unable to recognize "app.v1.yml": no matches for kind "Deployment" in version "app/v1beta1"
```



修改 yaml 文件为：

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



