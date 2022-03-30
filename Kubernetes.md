# Kubernetes

Kubernetes 是用于自动部署，扩展和管理容器化应用程序的开源系统。
它将组成应用程序的容器组合成逻辑单元，以便于管理和服务发现。Kubernetes 源自Google 15 年生产环境的运维经验，同时凝聚了社区的最佳创意和实践。

### Kubernetes 特性

* 自动化上线和回滚  
  Kubernetes 会分步骤地将针对应用或其配置的更改上线，同时监视应用程序运行状况以确保你不会同时终止所有实例。如果出现问题，Kubernetes 会为你回滚所作更改。你应该充分利用不断成长的部署方案生态系统。
  <br>
* 服务发现与负载均衡
  无需修改你的应用程序即可使用陌生的服务发现机制。Kubernetes 为容器提供了自己的 IP 地址和一个 DNS 名称，并且可以在它们之间实现负载均衡。
  <br> 
* 存储编排
  自动挂载所选存储系统，包括本地存储、诸如 GCP 或 AWS 之类公有云提供商所提供的存储或者诸如 NFS、iSCSI、Gluster、Ceph、Cinder 或 Flocker 这类网络存储系统。
  <br>
* Secret 和配置管理
  部署和更新 Secrets 和应用程序的配置而不必重新构建容器镜像，且 不必将软件堆栈配置中的秘密信息暴露出来。
  <br>
* 自动装箱
  根据资源需求和其他约束自动放置容器，同时避免影响可用性。 将关键性的和尽力而为性质的工作负载进行混合放置，以提高资源利用率并节省更多资源。
  <br>
* 批量执行
  除了服务之外，Kubernetes 还可以管理你的批处理和 CI 工作负载，在期望时替换掉失效的容器。
  <br>
* IPv4/IPv6 双协议栈
  为 Pod 和 Service 分配 IPv4 和 IPv6 地址
  <br>
* 水平扩缩
  使用一个简单的命令、一个 UI 或基于 CPU 使用情况自动对应用程序进行扩缩。
  <br>
* 自我修复
  重新启动失败的容器，在节点死亡时替换并重新调度容器，杀死不响应用户定义的健康检查的容器，并且在它们准备好服务之前不会将它们公布给客户端。
  <br>
* 为扩展性设计
  无需更改上游源码即可扩展你的 Kubernetes 集群。

### Kubernetes Pods

`Pod` 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。 

`Pod`是一组（一个或多个） 容器； 这些容器共享存储、网络、以及怎样运行这些容器的声明。 通常不需要直接创建 Pod，会使用`Deployment`或者`Job`这类工作负载资源 来创建`Pod`。如果`Pod`需要跟踪状态，可以考虑`StatefulSet`资源。

Kubernetes 集群中的 Pod 主要有两种用法：
* 运行单个容器的`Pod`。
  "每个 Pod 一个容器"模型是最常见的 Kubernetes 用例； 在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。
  <br>
* 运行多个协同工作的容器的`Pod`。 
  `Pod`可能封装由多个紧密耦合且需要共享资源的共处容器组成的应用程序。这些位于同一位置的容器可能形成单个内聚的服务单元。例如：一个容器将文件从共享卷提供给公众， 而另一个单独的“边车”（sidecar）容器则刷新或更新这些文件。 Pod 将这些容器和存储资源打包为一个可管理的实体。
  <br>
  <img src="/images/docs/pod.svg" width="450px;">

<br>
每个`Pod`运行给定应用程序的单个实例。如果希望横向扩展应用程序（例如，运行多个实例 以提供更多的资源），则应该使用多个 Pod，每个实例使用一个 Pod。 在 Kubernetes 中，这通常被称为 副本（Replication）。 通常使用一种工作负载资源及其控制器 来创建和管理一组 Pod 副本。

##### 使用 Pod
你很少在 Kubernetes 中直接创建一个个的 Pod，甚至是单实例（Singleton）的 Pod。 这是因为 Pod 被设计成了相对临时性的、用后即抛的一次性实体。 当 Pod 由你或者间接地由 控制器 创建时，它被调度在集群中的节点上运行。 Pod 会保持在该节点上运行，直到 Pod 结束执行、Pod 对象被删除、Pod 因资源不足而被 驱逐 或者节点失效为止。

##### Pod 和控制器
你可以使用工作负载资源来创建和管理多个 Pod。 资源的控制器能够处理副本的管理、上线，并在 Pod 失效时提供自愈能力。 例如，如果一个节点失败，控制器注意到该节点上的 Pod 已经停止工作， 就可以创建替换性的 Pod。调度器会将替身 Pod 调度到一个健康的节点执行。

下面是一些管理一个或者多个 Pod 的工作负载资源的示例：

* Deployment
* StatefulSet
* DaemonSet

##### Pod 模版
负载资源的控制器通常使用 Pod 模板（Pod Template） 来替你创建 Pod 并管理它们。
Pod 模板是包含在工作负载对象中的规范，用来创建`Pod`。这类负载资源包括`Deployment`、 `Job` 和 `DaemonSets`等。
工作负载的控制器会使用负载对象中的 PodTemplate 来生成实际的 Pod。 PodTemplate 是你用来运行应用时指定的负载资源的目标状态的一部分。
```yaml
apiVersion: app/v1
kind: Deployment
metadata:
  name: hello
spec:
  template:
    # 这里是 Pod 模版
    spec:
      replicas: 1
      containers:
      - name: hello
        image: busybox
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # 以上为 Pod 模版
```
修改 Pod 模版或者切换到新的 Pod 模版都不会对已经存在的 Pod 起作用。 Pod 不会直接收到模版的更新。相反， 新的 Pod 会被创建出来，与更改后的 Pod 模版匹配。

例如，Deployment 控制器针对每个 Deployment 对象确保运行中的 Pod 与当前的 Pod 模版匹配。如果模版被更新，则 Deployment 必须删除现有的 Pod，基于更新后的模版 创建新的 Pod。每个工作负载资源都实现了自己的规则，用来处理对 Pod 模版的更新。

##### 容器探针
由 kubelet 对容器执行的定期诊断。要执行诊断，kubelet 可以执行三种动作：
1. `ExecAction`（借助容器运行时执行）
2. `TCPSocketAction`（由 kubelet 直接检测）
3. `HTTPGetAction`（由 kubelet 直接检测）


### Kubernetes Storage
1. Volumes / 卷
2. Persistent Volumes / 持久卷
3. Projected Volumes / 投射卷
4. Ephemeral Volumes / 临时卷
5. Storage Classes / 存储类
6. Dynamic Volume Provisioning / 动态卷供应
7. Volume Snapshots / 卷快照
8. Volume Snapshot Classes / 卷快照类
9. CSI Volume Cloning / CSI 卷克隆
10. Storage Capacity / 存储容量 
11. Node-specific Volume Limits / 特定于节点的卷数限制
12. Volume Health Monitoring / 卷健康监测

##### Volumes / 卷
Container 中的文件在磁盘上是临时存放的，这给 Container 中运行的较重要的应用程序带来一些问题。 问题之一是当容器崩溃时文件丢失。 kubelet 会重新启动容器，但容器会以干净的状态重启。 第二个问题会在同一 Pod 中运行多个容器并共享文件时出现。 Kubernetes 卷（Volume） 这一抽象概念能够解决这两个问题。

Docker 也有 卷（Volume） 的概念，但对它只有少量且松散的管理。 Docker 卷是磁盘上或者另外一个容器内的一个目录。 Docker 提供卷驱动程序，但是其功能非常有限。

Kubernetes 支持很多类型的卷。 Pod 可以同时使用任意数目的卷类型。 临时卷类型的生命周期与 Pod 相同，但持久卷可以比 Pod 的存活期长。 当 Pod 不再存在时，Kubernetes 也会销毁临时卷；不过 Kubernetes 不会销毁持久卷。 对于给定 Pod 中任何类型的卷，在容器重启期间数据都不会丢失。

卷的核心是一个目录，其中可能存有数据，Pod 中的容器可以访问该目录中的数据。 所采用的特定的卷类型将决定该目录如何形成的、使用何种介质保存数据以及目录中存放的内容。

###### 25种卷类型
  - local
    `local`卷所代表的是某个被挂载的本地存储设备，例如磁盘、分区或者目录。  
    `local`卷只能用作静态创建的持久卷。尚不支持动态配置。
    与`hostPath`卷相比，`local`卷能够以持久和可移植的方式使用，而无需手动将`Pod`调度到节点。系统通过查看 `PersistentVolume`的节点亲和性配置，就能了解卷的节点约束。
    然而，local 卷仍然取决于底层节点的可用性，并不适合所有应用程序。 如果节点变得不健康，那么 local 卷也将变得不可被 Pod 访问。使用它的 Pod 将不能运行。 使用 local 卷的应用程序必须能够容忍这种可用性的降低，以及因底层磁盘的耐用性特征而带来的潜在的数据丢失风险。
    下面是一个使用`local`卷和`nodeAffinity`的持久卷示例：
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: example-pv
    spec:
      capacity:
        storage: 100Gi
      volumeMode: Filesystem
      accessModes:
      - ReadWriteOnce
      persistentVolumeReclaimPolicy: Delete
      storageClassName: local-storage
      local:
        path: /mnt/disks/ssd1
      nodeAffinity:
        required:
          nodeSelectorTerms:
          - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
              - example-node
    ```
  - hostPath
    >警告：
    HostPath 卷存在许多安全风险，最佳做法是尽可能避免使用 HostPath。 当必须使用 HostPath 卷时，它的范围应仅限于所需的文件或目录，并以只读方式挂载。
    如果通过 AdmissionPolicy 限制 HostPath 对特定目录的访问，则必须要求 volumeMounts 使用 readOnly 挂载以使策略生效。

    `hostPath`卷能将主机节点文件系统上的文件或目录挂载到你的`Pod`中。虽然这不是大多数`Pod`需要的，但是它为一些应用程序提供了强大的逃生舱。
    例如，`hostPath`的一些用法有：
    - 运行一个需要访问 Docker 内部机制的容器；可使用 hostPath 挂载 /var/lib/docker 路径。
    - 在容器中运行`cAdvisor`时，以`hostPath`方式挂载 /sys。
    - 允许`Pod`指定给定的`hostPath`在运行 Pod 之前是否应该存在，是否应该创建以及应该以什么方式存在。
  
    支持的 type 值如下：
    |  取值   | 行为  |
    |  ----  | ----  |
    |   | 空字符串（默认）用于向后兼容，这意味着在安装 hostPath 卷之前不会执行任何检查。 |
    | DirectoryOrCreate  | 如果在给定路径上什么都不存在，那么将根据需要创建空目录，权限设置为 0755，具有与 kubelet 相同的组和属主信息。 |
    | Directory | 在给定路径上必须存在的目录。 |
    | FileOrCreate | 如果在给定路径上什么都不存在，那么将在那里根据需要创建空文件，权限设置为 0644，具有与 kubelet 相同的组和所有权。|
    | File | 在给定路径上必须存在的文件。|
    | Socket | 在给定路径上必须存在的 UNIX 套接字。|
    | CharDevice | 在给定路径上必须存在的字符设备。|
    | BlockDevice | 在给定路径上必须存在的块设备。|

    当使用这种类型的卷时要小心，因为：
    - HostPath 卷可能会暴露特权系统凭据（例如 Kubelet）或特权API(例如容器运行时套接字)，可用于容器逃逸或攻击集群的其他部分。
    - 具有相同配置（例如基于同一 PodTemplate 创建）的多个 Pod 会由于节点上文件的不同而在不同节点上有不同的行为。
    - 下层主机上创建的文件或目录只能由 root 用户写入。你需要在 特权容器 中以 root 身份运行进程，或者修改主机上的文件权限以便容器能够写入 hostPath 卷。

    `hostPath` 配置示例：
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: test-pd
    spec:
      containers:
      - image: k8s.gcr.io/test-webserver
        name: test-container
        volumeMounts:
        - mountPath: /test-pd
          name: test-volume
      volumes:
      - name: test-volume
        hostPath:
          # 宿主上目录位置
          path: /data
          # 此字段为可选
          type: Directory
    ```

  - nfs
    nfs 卷能将 NFS (网络文件系统) 挂载到你的 Pod 中。 不像 emptyDir 那样会在删除 Pod 的同时也会被删除，nfs 卷的内容在删除 Pod 时会被保存，卷只是被卸载。 这意味着 nfs 卷可以被预先填充数据，并且这些数据可以在 Pod 之间共享。
    ```bash
    # 安装NFS和RPC
    $ yum -y install nfs-utils rpcbind
    # 配置nfs目录
    # 新建data目录
    $ mkdir /data
    # 在文件/etc/exports添加如下内容
    $ echo '/data *(rw,no_root_squash,sync)' >> /etc/exports
    # 执行命令使配置生效
    $ exportfs -r
    # 配置完成后，启动 NFS 服务器
    $ systemctl enable nfs-server && systemctl start nfs-server
    # 验证
    $ showmount -e
    Export list for instance-cesz58w0:
    /data *
    ```
  - secret
    secret 卷用来给 Pod 传递敏感信息，例如密码, SSL证书，RSA秘钥对等

  - configMap
    configMap 卷提供了向 Pod 注入配置数据的方法。 ConfigMap 对象中存储的数据可以被 configMap 类型的卷引用，然后被 Pod 中运行的容器化应用使用。
    引用 configMap 对象时，你可以在 volume 中通过它的名称来引用。 你可以自定义 ConfigMap 中特定条目所要使用的路径。 下面的配置显示了如何将名为 log-config 的 ConfigMap 挂载到名为 configmap-pod 的 Pod 中：
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: configmap-pod
    spec:
      containers:
      - name: test
        image: busybox
        volumeMounts:
          - name: config-vol
            mountPath: /etc/config
      volumes:
        - name: config-vol
          configMap:
            name: log-config
            items:
              - key: log_level
                path: log_level
    ```

  - persistentVolumeClaim
    persistentVolumeClaim 卷用来将持久卷（PersistentVolume）挂载到 Pod 中。 持久卷申领（PersistentVolumeClaim）是用户在不知道特定云环境细节的情况下“申领”持久存储（例如 GCE PersistentDisk 或者 iSCSI 卷）的一种方法。

  - awsElasticBlockStore
  - azureDisk
  - azureFile
  - cephfs
  - cinder
  - downwardAPI
  - emptyDir
  - fc (光纤通道)
  - flocker （已弃用）
  - gcePersistentDisk
  - gitRepo (已弃用) 
  - glusterfs
  - iscsi
  - portworxVolume
  - projected （投射）
  - quobyte (已弃用)
  - rbd
  - storageOS (已弃用)
  - vsphereVolume

##### Persistent Volumes / 持久卷
持久卷（PersistentVolume，PV）是集群中的一块存储，可以由管理员事先供应，或者 使用存储类（Storage Class）来动态供应。 持久卷是集群资源，就像节点也是集群资源一样。PV 持久卷和普通的 Volume 一样，也是使用 卷插件来实现的，只是它们拥有独立于任何使用 PV 的 Pod 的生命周期。 此 API 对象中记述了存储的实现细节，无论其背后是 NFS、iSCSI 还是特定于云平台的存储系统。

持久卷申领（PersistentVolumeClaim，PVC）表达的是用户对存储的请求。概念上与 Pod 类似。 Pod 会耗用节点资源，而 PVC 申领会耗用 PV 资源。Pod 可以请求特定数量的资源（CPU 和内存）；同样 PVC 申领也可以请求特定的大小和访问模式 （例如，可以要求 PV 卷能够以 ReadWriteOnce、ReadOnlyMany 或 ReadWriteMany 模式之一来挂载，参见访问模式）。

尽管 PersistentVolumeClaim 允许用户消耗抽象的存储资源，常见的情况是针对不同的 问题用户需要的是具有不同属性（如，性能）的 PersistentVolume 卷。 集群管理员需要能够提供不同性质的 PersistentVolume，并且这些 PV 卷之间的差别不 仅限于卷大小和访问模式，同时又不能将卷是如何实现的这些细节暴露给用户。 为了满足这类需求，就有了 存储类（StorageClass） 资源。

###### 卷和申领的生命周期
- 供应 (PV 卷的供应有两种方式：静态供应或动态供应)
  > 静态供应:
  集群管理员创建若干 PV 卷。这些卷对象带有真实存储的细节信息，并且对集群 用户可用（可见）。PV 卷对象存在于 Kubernetes API 中，可供用户消费（使用）。
  动态供应:
  如果管理员所创建的所有静态 PV 卷都无法与用户的 PersistentVolumeClaim 匹配， 集群可以尝试为该 PVC 申领动态供应一个存储卷。 这一供应操作是基于 StorageClass 来实现的：PVC 申领必须请求某个 存储类，同时集群管理员必须 已经创建并配置了该类，这样动态供应卷的动作才会发生。 如果 PVC 申领指定存储类为 ""，则相当于为自身禁止使用动态供应的卷。
  为了基于存储类完成动态的存储供应，集群管理员需要在 API 服务器上启用 DefaultStorageClass 准入控制器。 举例而言，可以通过保证 DefaultStorageClass 出现在 API 服务器组件的 --enable-admission-plugins 标志值中实现这点；该标志的值可以是逗号 分隔的有序列表。关于 API 服务器标志的更多信息，可以参考 kube-apiserver 文档。
- 绑定
  用户创建一个带有特定存储容量和特定访问模式需求的 PersistentVolumeClaim 对象； 在动态供应场景下，这个 PVC 对象可能已经创建完毕。 主控节点中的控制回路监测新的 PVC 对象，寻找与之匹配的 PV 卷（如果可能的话）， 并将二者绑定到一起。 如果为了新的 PVC 申领动态供应了 PV 卷，则控制回路总是将该 PV 卷绑定到这一 PVC 申领。 否则，用户总是能够获得他们所请求的资源，只是所获得的 PV 卷可能会超出所请求的配置。 一旦绑定关系建立，则 PersistentVolumeClaim 绑定就是排他性的，无论该 PVC 申领是 如何与 PV 卷建立的绑定关系。 PVC 申领与 PV 卷之间的绑定是一种一对一的映射，实现上使用 ClaimRef 来记述 PV 卷 与 PVC 申领间的双向绑定关系。

  如果找不到匹配的 PV 卷，PVC 申领会无限期地处于未绑定状态。 当与之匹配的 PV 卷可用时，PVC 申领会被绑定。 例如，即使某集群上供应了很多 50 Gi 大小的 PV 卷，也无法与请求 100 Gi 大小的存储的 PVC 匹配。当新的 100 Gi PV 卷被加入到集群时，该 PVC 才有可能被绑定。
- 使用
  Pod 将 PVC 申领当做存储卷来使用。集群会检视 PVC 申领，找到所绑定的卷，并 为 Pod 挂载该卷。对于支持多种访问模式的卷，用户要在 Pod 中以卷的形式使用申领 时指定期望的访问模式。

  一旦用户有了申领对象并且该申领已经被绑定，则所绑定的 PV 卷在用户仍然需要它期间 一直属于该用户。用户通过在 Pod 的 volumes 块中包含 persistentVolumeClaim 节区来调度 Pod，访问所申领的 PV 卷。 相关细节可参阅使用申领作为卷。

- 保护使用中的存储对象
  保护使用中的存储对象（Storage Object in Use Protection）这一功能特性的目的 是确保仍被 Pod 使用的 PersistentVolumeClaim（PVC）对象及其所绑定的 PersistentVolume（PV）对象在系统中不会被删除，因为这样做可能会引起数据丢失。<br>
  如果用户删除被某 Pod 使用的 PVC 对象，该 PVC 申领不会被立即移除。 PVC 对象的移除会被推迟，直至其不再被任何 Pod 使用。 此外，如果管理员删除已绑定到某 PVC 申领的 PV 卷，该 PV 卷也不会被立即移除。 PV 对象的移除也要推迟到该 PV 不再绑定到 PVC。
  ```bash
  $ kubectl describe pvc hostpath
  Name:          hostpath
  Namespace:     default
  StorageClass:  example-hostpath
  Status:        Terminating
  Volume:
  Labels:        <none>
  Annotations:   volume.beta.kubernetes.io/storage-class=example-hostpath
                 volume.beta.kubernetes.io/storage-provisioner=example.com/hostpath
  Finalizers:    [kubernetes.io/pvc-protection]
  ...

  $ kubectl describe pv task-pv-volume
  Name:            task-pv-volume
  Labels:          type=local
  Annotations:     <none>
  Finalizers:      [kubernetes.io/pv-protection]
  StorageClass:    standard
  Status:          Terminating
  Claim:
  Reclaim Policy:  Delete
  Access Modes:    RWO
  Capacity:        1Gi
  Message:
  Source:
    Type:          HostPath (bare host directory volume)
    Path:          /tmp/data
    HostPathType:
  Events:          <none>
  ```
- 回收
  当用户不再使用其存储卷时，他们可以从 API 中将 PVC 对象删除，从而允许 该资源被回收再利用。PersistentVolume 对象的回收策略告诉集群，当其被 从申领中释放时如何处理该数据卷。 目前，数据卷可以被 Retained（保留）、Recycled（回收）或 Deleted（删除）。
- 保留（Retain）
  回收策略 Retain 使得用户可以手动回收资源。当 PersistentVolumeClaim 对象 被删除时，PersistentVolume 卷仍然存在，对应的数据卷被视为"已释放（released）"。 由于卷上仍然存在这前一申领人的数据，该卷还不能用于其他申领。 管理员可以通过下面的步骤来手动回收该卷：
    1. 删除 PersistentVolume 对象。与之相关的、位于外部 2. 基础设施中的存储资产 （例如 AWS EBS、GCE PD、Azure Disk 或 Cinder 卷）在 PV 删除之后仍然存在。
    2. 根据情况，手动清除所关联的存储资产上的数据。
    3. 手动删除所关联的存储资产。
   
  如果你希望重用该存储资产，可以基于存储资产的定义创建新的 PersistentVolume 卷对象。

  ### Kubernetes 中的代理
  用户在使用 Kubernetes 的过程中可能遇到几种不同的代理（proxy）：
  1. kubectl proxy：
       - 运行在用户的桌面或 pod 中
       - 从本机地址到 Kubernetes apiserver 的代理




   
   
   
   

