# Kubernetes in timework

### Kubernetes 集群安装
  - 环境准备
    ```bash
    # 安装 git client
    $ sudo yum install git -y
    # Clone kubeadm-ha 源码
    $ git clone https://github.com/open-hand/kubeadm-ha.git
    # 安装 Ansible
    $ cd kubeadm-ha && sudo ./install-ansible.sh
    ```
  - 配置 ansible inventory 文件
    - 项目 example 文件夹下提供了 6 个 ansible inventory 示例文件，请按需求进行选择并修改。
    - 拷贝项目下的`example/hosts.m-master.ip.ini`文件至项目根目录下，命名为`inventory.ini`。
        ```$ cp example/hosts.m-master.ip.ini inventory.ini```
        修改各服务器的 IP 地址、用户名、密码，并维护好各服务器与角色的关系。
        ```bash
        [all]
        172.16.0.45 ansible_port=22 ansible_user="root" ansible_ssh_pass=""
        172.16.0.3 ansible_port=22 ansible_user="root" ansible_ssh_pass=""        
        [etcd]
        172.16.0.45
        [kube-master]
        172.16.0.3
        [kube-worker]
        172.16.0.45
        172.16.0.3

        container_manager="docker"
        ```
  - 部署集群
    ```bash
    # 在项目根目录下执行
    $ ansible-playbook -i inventory.ini 90-init-cluster.yml
    ```
  - 查看等待 pod 的状态为 runnning
    ```bash
    # 任意master节点下执行
    $ kubectl get po --all-namespaces -w
    ```
  - 如果部署失败，想要重置集群，执行
    ```bash
    # 在项目根目录下执行
    $ ansible-playbook -i inventory.ini 99-reset-cluster.yml
    ```
### 本地伪集群环境的安装
  - 安装Docker Desktop
  - 配置proxy
    ```bash
    {
        "registry-mirrors": [
            "https://6n5qzb8y.mirror.aliyuncs.com"
        ],
        "features": {
            "buildkit": true
        },
        "experimental": false,
        "builder": {
            "gc": {
                "defaultKeepStorage": "20GB",
                "enabled": true
            }
        }
    }
    ```
- 安装 Kubernetes
  ![Install Kubernetes with Mac Docker Desktop](/images/docs/WeChatWorkScreenshot_275776cd-a785-4e1f-926f-dce293eb6b7e.png)

### Kubectl 常用命令
```bash
Usage:
  kubectl [flags] [options]
```

```bash
Basic Commands (Beginner):
  create        Create a resource from a file or from stdin
  expose        Take a replication controller, service, deployment or pod and expose it as a new Kubernetes service
  run           Run a particular image on the cluster
  set           Set specific features on objects

Basic Commands (Intermediate):
  explain       Get documentation for a resource
  get           Display one or many resources
  edit          Edit a resource on the server
  delete        Delete resources by file names, stdin, resources and names, or by resources and label selector

Deploy Commands:
  rollout       Manage the rollout of a resource
  scale         Set a new size for a deployment, replica set, or replication controller
  autoscale     Auto-scale a deployment, replica set, stateful set, or replication controller

Cluster Management Commands:
  certificate   Modify certificate resources.
  cluster-info  Display cluster information
  top           Display resource (CPU/memory) usage
  cordon        Mark node as unschedulable
  uncordon      Mark node as schedulable
  drain         Drain node in preparation for maintenance
  taint         Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe      Show details of a specific resource or group of resources
  logs          Print the logs for a container in a pod
  attach        Attach to a running container
  exec          Execute a command in a container
  port-forward  Forward one or more local ports to a pod
  proxy         Run a proxy to the Kubernetes API server
  cp            Copy files and directories to and from containers
  auth          Inspect authorization
  debug         Create debugging sessions for troubleshooting workloads and nodes

Advanced Commands:
  diff          Diff the live version against a would-be applied version
  apply         Apply a configuration to a resource by file name or stdin
  patch         Update fields of a resource
  replace       Replace a resource by file name or stdin
  wait          Experimental: Wait for a specific condition on one or many resources
  kustomize     Build a kustomization target from a directory or URL.

Settings Commands:
  label         Update the labels on a resource
  annotate      Update the annotations on a resource
  completion    Output shell completion code for the specified shell (bash or zsh)

Other Commands:
  api-resources Print the supported API resources on the server
  api-versions  Print the supported API versions on the server, in the form of "group/version"
  config        Modify kubeconfig files
  plugin        Provides utilities for interacting with plugins
  version       Print the client and server version information
```
  * 查看集群中的节点`node`
    ```bash
    $ kubectl get node -o wide
    NAME          STATUS   ROLES                         AGE   VERSION    INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
    172.16.0.3    Ready    control-plane,master,worker   34d   v1.20.11   172.16.0.3    <none>        CentOS Linux 7 (Core)   3.10.0-1160.53.1.el7.x86_64   docker://20.10.7
    172.16.0.45   Ready    etcd,worker                   34d   v1.20.11   172.16.0.45   <none>        CentOS Linux 7 (Core)   3.10.0-1160.53.1.el7.x86_64   docker://20.10.7
    ```
  * 查看集群中的`namespace/ns`
    ```bash
    $ kubectl get namespace
    NAME                   STATUS   AGE
    choerodon              Active   34d
    choerodon-test         Active   34d
    default                Active   34d
    ingress-controller     Active   34d
    kube-node-lease        Active   34d
    kube-public            Active   34d
    kube-system            Active   34d
    kubernetes-dashboard   Active   34d
    prod-tmis              Active   34d
    ```
  * 查看命名空间中的`ingress/ing`
    ```bash
    $ kubectl get ingress -n prod-tmis -o wide
    NAME                                    CLASS    HOSTS                     ADDRESS         PORTS     AGE
    minio-c8d97                             <none>   minio-tmis.timework.cn    10.244.77.122   80, 443   34d
    scrm-gateway-8c82c                      <none>   api-tmis.timework.cn      10.244.77.122   80, 443   33d
    tmis-frontend-55cdf                     <none>   tmis-subapp.timework.cn   10.244.77.122   80, 443   34d
    tmis-management-mobile-d9033            <none>   tmis.timework.cn          10.244.77.122   80, 443   34d
    tmis-qiankun-main-front-104c0           <none>   tmis.timework.cn          10.244.77.122   80, 443   34d
    tmis-saas-front-d37f7                   <none>   tmis-subapp.timework.cn   10.244.77.122   80, 443   34d
    websocket-service-node-0c75b-http       <none>   api-tmis.timework.cn      10.244.77.122   80, 443   34d
    websocket-service-node-0c75b-socketio   <none>   api-tmis.timework.cn      10.244.77.122   80, 443   34d
    ```
  * 查看命名空间中的`service/svc`
    ```bash
    $ kubectl get service -n prod-tmis -o wide
    NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                       AGE   SELECTOR
    gateway                       ClusterIP   10.244.94.128   <none>        8080/TCP                      34d   choerodon.io/release=scrm-gateway-8c82c
    minio                         ClusterIP   10.244.110.78   <none>        9000/TCP                      34d   app=minio,choerodon.io/release=minio-c8d97,release=minio-c8d97
    mysql                         NodePort    10.244.115.18   <none>        3306:30306/TCP                34d   choerodon.io/infra=mysql,choerodon.io/release=mysql-1fddd
    rabbitmq-ha-9074a             ClusterIP   10.244.125.77   <none>        15672/TCP,5672/TCP,4369/TCP   34d   app=rabbitmq-ha,release=rabbitmq-ha-9074a
    rabbitmq-ha-9074a-discovery   ClusterIP   None            <none>        15672/TCP,5672/TCP,4369/TCP   34d   app=rabbitmq-ha,release=rabbitmq-ha-9074a
    redis                         ClusterIP   10.244.117.18   <none>        6379/TCP                      34d   choerodon.io/infra=redis,choerodon.io/release=redis-ddc49
    register-server               ClusterIP   10.244.69.111   <none>        8000/TCP                      34d   choerodon.io/release=scrm-register-02d61
    tmis-front                    ClusterIP   10.244.103.96   <none>        80/TCP                        34d   choerodon.io/release=tmis-frontend-55cdf
    tmis-management-mobile        ClusterIP   10.244.72.15    <none>        80/TCP                        34d   choerodon.io/release=tmis-management-mobile-d9033
    tmis-qiankun-main-front       ClusterIP   10.244.81.135   <none>        80/TCP                        34d   choerodon.io/release=tmis-qiankun-main-front-104c0
    tmis-saas-front               ClusterIP   10.244.121.2    <none>        80/TCP                        34d   choerodon.io/release=tmis-saas-front-d37f7
    websocket-service-node        ClusterIP   10.244.119.93   <none>        3000/TCP                      34d   choerodon.io/release=websocket-service-node-0c75b
    ```
  * 查看命名空间中的`deployment/deploy`
    ```bash
    $ kubectl get deployment -n prod-tmis -o wide
    NAME                            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                      IMAGES                                                                                               SELECTOR
    minio-c8d97                     1/1     1            1           34d   minio                           minio/minio:RELEASE.2020-03-25T07-03-04Z                                                             app=minio,choerodon.io/release=minio-c8d97,release=minio-c8d97
    mysql-1fddd                     1/1     1            1           34d   mysql-1fddd                     mysql:5.7.28                                                                                         choerodon.io/infra=mysql,choerodon.io/release=mysql-1fddd
    redis-ddc49                     1/1     1            1           34d   redis-ddc49                     harbor-devops.timework.cn/operation-baseframework/redis:6                                            choerodon.io/infra=redis,choerodon.io/release=redis-ddc49
    scrm-admin-87755                1/1     1            1           34d   scrm-admin-87755                harbor-devops.timework.cn/operation-tw-crm/scrm-admin:1.4.0                                          choerodon.io/release=scrm-admin-87755
    scrm-gateway-8c82c              1/1     1            1           34d   scrm-gateway-8c82c              harbor-devops.timework.cn/operation-tw-crm/scrm-gateway:2022.2.23-150749-master                      choerodon.io/release=scrm-gateway-8c82c
    scrm-import-7e9d8               1/1     1            1           34d   scrm-import-7e9d8               harbor-devops.timework.cn/operation-tw-crm/scrm-import:1.4.0                                         choerodon.io/release=scrm-import-7e9d8
    scrm-platform-f1b01             1/1     1            1           34d   scrm-platform-f1b01             harbor-devops.timework.cn/operation-tw-crm/scrm-platform:2022.2.9-114738-feature-device              choerodon.io/release=scrm-platform-f1b01
    scrm-register-02d61             1/1     1            1           34d   scrm-register-02d61             harbor-devops.timework.cn/operation-tw-crm/scrm-register:1.4.0                                       choerodon.io/release=scrm-register-02d61
    scrm-scheduler-d40a8            1/1     1            1           34d   scrm-scheduler-d40a8            harbor-devops.timework.cn/operation-tw-crm/scrm-scheduler:2021.12.1-163101-master                    choerodon.io/release=scrm-scheduler-d40a8
    tmis-file-51f54                 1/1     1            1           34d   tmis-file-51f54                 harbor-devops.timework.cn/operation-tw-sygg2/tmis-file:2022.1.21-172342-master                       choerodon.io/release=tmis-file-51f54
    tmis-frontend-55cdf             1/1     1            1           34d   tmis-frontend-55cdf             harbor-devops.timework.cn/operation-tw-sygg2/tmis-frontend:2022.3.24-101839-master-18146             choerodon.io/release=tmis-frontend-55cdf
    tmis-iam-3f278                  1/1     1            1           34d   tmis-iam-3f278                  harbor-devops.timework.cn/operation-tw-sygg2/tmis-iam:2022.1.24-215715-master                        choerodon.io/release=tmis-iam-3f278
    tmis-management-mobile-d9033    1/1     1            1           34d   tmis-management-mobile-d9033    harbor-devops.timework.cn/operation-tw-sygg2/tmis-management-mobile:2022.3.22-100026-master-17918    choerodon.io/release=tmis-management-mobile-d9033
    tmis-message-ca8e1              1/1     1            1           34d   tmis-message-ca8e1              harbor-devops.timework.cn/operation-tw-sygg2/tmis-message:2022.1.26-233310-master                    choerodon.io/release=tmis-message-ca8e1
    tmis-oauth-e8380                1/1     1            1           34d   tmis-oauth-e8380                harbor-devops.timework.cn/operation-tw-sygg2/tmis-oauth:2022.2.17-172405-master                      choerodon.io/release=tmis-oauth-e8380
    tmis-qiankun-main-front-104c0   1/1     1            1           34d   tmis-qiankun-main-front-104c0   harbor-devops.timework.cn/operation-tw-sygg2/tmis-qiankun-main-front:2022.3.25-180708-master-18324   choerodon.io/release=tmis-qiankun-main-front-104c0
    tmis-saas-front-d37f7           1/1     1            1           34d   tmis-saas-front-d37f7           harbor-devops.timework.cn/operation-tw-sygg2/tmis-saas-front:2021.12.27-172153-master                choerodon.io/release=tmis-saas-front-d37f7
    tmis-service-3558f              1/1     1            1           34d   tmis-service-3558f              harbor-devops.timework.cn/operation-tw-sygg2/tmis-service:1.2.0                                      choerodon.io/release=tmis-service-3558f
    websocket-service-node-0c75b    1/1     1            1           34d   websocket-service-node-0c75b    harbor-devops.timework.cn/operation-tw-sygg2/websocket-service-node:1.2.0                            choerodon.io/release=websocket-service-node-0c75b
    ```
  * 查看命名空间中的`pod/po`
    ```bash
    $ kubectl get pod -n prod-tmis -o wide
    NAME                                             READY   STATUS    RESTARTS   AGE     IP              NODE          NOMINATED NODE   READINESS GATES
    minio-c8d97-c5d5b7854-p79b7                      1/1     Running   0          34d     10.244.63.71    172.16.0.3    <none>           <none>
    mysql-1fddd-7f6896874f-lj8gk                     1/1     Running   0          34d     10.244.55.77    172.16.0.45   <none>           <none>
    rabbitmq-ha-9074a-0                              1/1     Running   0          34d     10.244.55.80    172.16.0.45   <none>           <none>
    rabbitmq-ha-9074a-1                              1/1     Running   0          34d     10.244.63.72    172.16.0.3    <none>           <none>
    rabbitmq-ha-9074a-2                              1/1     Running   0          34d     10.244.63.73    172.16.0.3    <none>           <none>
    redis-ddc49-7cf7c9c576-5wpvk                     1/1     Running   0          34d     10.244.55.79    172.16.0.45   <none>           <none>
    scrm-admin-87755-9f594b775-94ljm                 1/1     Running   0          34d     10.244.55.93    172.16.0.45   <none>           <none>
    scrm-gateway-8c82c-5bb7699794-qbv92              1/1     Running   0          33d     10.244.55.103   172.16.0.45   <none>           <none>
    scrm-import-7e9d8-7fc5b5d99c-6npt8               1/1     Running   0          34d     10.244.55.92    172.16.0.45   <none>           <none>
    scrm-platform-f1b01-5fff9b779-jgd8v              1/1     Running   0          34d     10.244.55.90    172.16.0.45   <none>           <none>
    scrm-register-02d61-f6c866bfd-c6nt9              1/1     Running   0          34d     10.244.55.85    172.16.0.45   <none>           <none>
    scrm-scheduler-d40a8-7d5d7dc45c-s8tgv            1/1     Running   0          34d     10.244.63.75    172.16.0.3    <none>           <none>
    tmis-file-51f54-75665fd896-kqs85                 1/1     Running   0          34d     10.244.63.83    172.16.0.3    <none>           <none>
    tmis-frontend-55cdf-8546db6869-jrbbf             1/1     Running   0          4d14h   10.244.55.120   172.16.0.45   <none>           <none>
    tmis-iam-3f278-f46667b59-m2k9w                   1/1     Running   0          34d     10.244.55.94    172.16.0.45   <none>           <none>
    tmis-management-mobile-d9033-7f87667ff4-dzhc9    1/1     Running   0          5d13h   10.244.55.118   172.16.0.45   <none>           <none>
    tmis-message-ca8e1-886f965bd-qlkcw               1/1     Running   0          34d     10.244.63.76    172.16.0.3    <none>           <none>
    tmis-oauth-e8380-66cb6b485c-78xcv                1/1     Running   0          34d     10.244.55.96    172.16.0.45   <none>           <none>
    tmis-qiankun-main-front-104c0-76757f5b88-gcp76   1/1     Running   0          3d6h    10.244.55.121   172.16.0.45   <none>           <none>
    tmis-saas-front-d37f7-6588b74c75-rx4q4           1/1     Running   0          34d     10.244.63.82    172.16.0.3    <none>           <none>
    tmis-service-3558f-74c6c95d98-8fvj8              1/1     Running   0          4d6h    10.244.63.108   172.16.0.3    <none>           <none>
    websocket-service-node-0c75b-6767887999-vgfxh    1/1     Running   0          6d9h    10.244.63.103   172.16.0.3    <none>           <none>
    ```
### Kubernetes入门

![K8s Ingress service deployment pod](/images/docs/937bdf484119d3dea073f55410b1a290.png)

参考文档如下:
 * ingress https://kubernetes.io/zh/docs/concepts/services-networking/ingress/
 * service https://kubernetes.io/zh/docs/concepts/services-networking/service/
 * deployment https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/
 * pod https://kubernetes.io/zh/docs/concepts/workloads/pods/
 * pv & pvc https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/

##### Ingress

Ingress 是对集群中服务的外部访问进行管理的 API 对象，典型的访问方式是 HTTP。

Ingress 可以提供负载均衡、SSL 终结和基于名称的虚拟托管。

Ingress 公开了从集群外部到集群内服务的 HTTP 和 HTTPS 路由。 流量路由由 Ingress 资源上定义的规则控制。

Ingress 可为 Service 提供外部可访问的 URL、负载均衡流量、终止 SSL/TLS，以及基于名称的虚拟托管。 Ingress 控制器 通常负责通过负载均衡器来实现 Ingress，尽管它也可以配置边缘路由器或其他前端来帮助处理流量。

Ingress 不会公开任意端口或协议。 将 HTTP 和 HTTPS 以外的服务公开到 Internet 时，通常使用 Service.Type=NodePort 或 Service.Type=LoadBalancer 类型的 Service。

##### Service

将运行在一组 Pods 上的应用程序公开为网络服务的抽象方法。
使用 Kubernetes，你无需修改应用程序即可使用不熟悉的服务发现机制。 Kubernetes 为 Pods 提供自己的 IP 地址，并为一组 Pod 提供相同的 DNS 名， 并且可以在它们之间进行负载均衡。

Kubernetes Service 定义了这样一种抽象：逻辑上的一组 Pod，一种可以访问它们的策略 —— 通常称为微服务。 Service 所针对的 Pods 集合通常是通过选择算符来确定的。 要了解定义服务端点的其他方法，请参阅不带选择算符的服务。

举个例子，考虑一个图片处理后端，它运行了 3 个副本。这些副本是可互换的 —— 前端不需要关心它们调用了哪个后端副本。 然而组成这一组后端程序的 Pod 实际上可能会发生变化， 前端客户端不应该也没必要知道，而且也不需要跟踪这一组后端的状态。

Service 定义的抽象能够解耦这种关联。

##### Deployment

一个 Deployment 为 Pods 和 ReplicaSets 提供声明式的更新能力。

你负责描述 Deployment 中的 目标状态，而 Deployment 控制器（Controller） 以受控速率更改实际状态， 使其变为期望状态。你可以定义 Deployment 以创建新的 ReplicaSet，或删除现有 Deployment， 并通过新的 Deployment 收养其资源。

##### Pod
Pod （就像在鲸鱼荚或者豌豆荚中）是一组（一个或多个） 容器； 这些容器共享存储、网络、以及怎样运行这些容器的声明。 Pod 中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。 Pod 所建模的是特定于应用的“逻辑主机”，其中包含一个或多个应用容器， 这些容器是相对紧密的耦合在一起的。 在非云环境中，在相同的物理机或虚拟机上运行的应用类似于 在同一逻辑主机上运行的云应用。

除了应用容器，Pod 还可以包含在 Pod 启动期间运行的 Init 容器。 你也可以在集群中支持临时性容器 的情况下，为调试的目的注入临时性容器。


你很少在 Kubernetes 中直接创建一个个的 Pod，甚至是单实例（Singleton）的 Pod。 这是因为 Pod 被设计成了相对临时性的、用后即抛的一次性实体。 当 Pod 由你或者间接地由 控制器 创建时，它被调度在集群中的节点上运行。 Pod 会保持在该节点上运行，直到 Pod 结束执行、Pod 对象被删除、Pod 因资源不足而被 驱逐 或者节点失效为止。

### 实战部分
* 在本地部署MySQL数据库
  - 创建`pv`  mysql_pv.yml 
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: mysql-data-pv-volume
      labels:
        type: local
    spec:
      storageClassName: manual
      capacity:
        storage: 10Gi
      accessModes:
        - ReadWriteOnce
      hostPath:
        path: "/Users/tomaer/.k8s_volume/mysql/data"
    ```
    ```bash
    # 创建MySQL data目录并且创建pv
    $ mkdir -p /Users/tomaer/.k8s_volume/mysql/data && kubectl apply -f mysql_pv.yml
    ```
  - 创建`pvc`，并且与pv绑定 mysql_pvc.yml
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-data-pv-claim
      namespace: test
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
    ```
    ```bash
    # 创建pvc 并且查看是否绑定成功
    $ kubectl apply -f mysql_pvc.yml && kubectl get pv
    NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS   REASON   AGE
    mysql-data-pv-volume   10Gi       RWO            Retain           Bound    test/mysql-data-pv-claim   manual                  32m
    ```
  - 创建`deployment` mysql_deployment.yml
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mysql
      namespace: test
    spec:
      selector:
        matchLabels:
          app: mysql
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: mysql
        spec:
          containers:
          - image: mysql:5.7.18
            name: mysql
            args: ["--character-set-server=utf8mb4", "--collation-server=utf8mb4_general_ci", "--explicit_defaults_for_timestamp=1","--max_connections=1000","--skip-name-resolve"]
            env:
            - name: MYSQL_ROOT_PASSWORD
              value: password
            - name: TZ
              value: Asia/Shanghai
            ports:
            - containerPort: 3306
              name: mysql
            volumeMounts:
            - name: mysql-data-persistent-storage
              mountPath: /var/lib/mysql
          volumes:
          - name: mysql-data-persistent-storage
            persistentVolumeClaim:
              claimName: mysql-data-pv-claim
    ```
    ```bash
    $ kubectl apply -f mysql_deployment.yml
    ```
  - 创建`service` mysql_service.yml
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: mysql
      namespace: test
    spec:
      type: NodePort
      ports:
      - port: 3306
        targetPort: 3306
        nodePort: 30036
      selector:
        app: mysql
    ```
    ```bash
    $ kubectl apply -f mysql_service.yml
    ```
* 在本地部署Redis
  - 创建`pv` redis_pv.yml
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: redis-data-pv-volume
      labels:
        type: local
    spec:
      storageClassName: manual
      capacity:
        storage: 3Gi
      accessModes:
        - ReadWriteOnce
      hostPath:
        path: "/Users/tomaer/.k8s_volume/redis/data"
    ```
    ```bash
    # 创建Redis data目录并且创建pv
    $ mkdir -p /Users/tomaer/.k8s_volume/redis/data && kubectl apply -f redis_pv.yml
    ```
  - 创建`pvc`,并且与pv绑定 redis_pvc.yml
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: redis-data-pv-claim
      namespace: test
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 3Gi
    ```
     ```bash
    # 创建pvc 并且查看是否绑定成功
    $ kubectl apply -f redis_pvc.yml && kubectl get pv
    NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS   REASON   AGE
    mysql-data-pv-volume   10Gi       RWO            Retain           Bound    test/mysql-data-pv-claim   manual                  81s
    redis-data-pv-volume   3Gi        RWO            Retain           Bound    test/redis-data-pv-claim   manual                  15s
    ```
  - 创建`deployment` redis_deployment.yml
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: redis
      namespace: test
    spec:
      selector:
        matchLabels:
          app: redis
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: redis
        spec:
          containers:
            - image: redis
              name: redis
              args: ["--protected-mode", "no", "--appendonly", "yes", "--dir", "/data"]
              env:
              - name: TZ
                value: Asia/Shanghai
              ports:
              - containerPort: 6379
                name: redis
              volumeMounts:
              - name: redis-data-persistent-storage
                mountPath: /data
          volumes:
            - name: redis-data-persistent-storage
              persistentVolumeClaim:
                claimName: redis-data-pv-claim
    ```
    ```bash
    $ kubectl apply -f redis_deployment.yml
    ```
  - 创建`Service`, redis_service.yml
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: redis
      namespace: test
    spec:
      type: NodePort
      ports:
      - port: 6379
        targetPort: 6379
        nodePort: 30079
      selector:
        app: redis
    ```
    ```bash
    $ kubectl apply -f redis_service.yml
    ```
