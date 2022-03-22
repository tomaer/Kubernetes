# 在 Linux 系统中安装并设置 kubectl

## 准备开始
kubectl 版本和集群版本之间的差异必须在一个小版本号内。 例如：v1.23 版本的客户端能与 v1.22、 v1.23 和 v1.24 版本的控制面通信。 用最新兼容版的 kubectl 有助于避免不可预见的问题。

## 在 Linux 系统中安装 kubectl

在 Linux 系统中安装 kubectl 有如下几种方法：
* 用 curl 在 Linux 系统中安装 kubectl
* 用原生包管理工具安装
* 用其他包管理工具安装

### 用 curl 在 Linux 系统中安装 kubectl
1. 用以下命令下载最新发行版：
    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    ```

    如需下载某个指定的版本，请用指定版本号替换该命令的这一部分：
   ```bash 
   $(curl -L -s https://dl.k8s.io/release/stable.txt)
   ```
   例如，要在 Linux 中下载v1.23.0版本，请输入：

   ```bash
   curl -LO https://dl.k8s.io/release/v1.23.0/bin/linux/amd64/kubectl
   ```
2. 验证该可执行文件（可选步骤）
    下载 kubectl 校验和文件：
    ```bash
    curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
    ```
    基于校验和文件，验证 kubectl 的可执行文件：
    ```bash
    echo "$(<kubectl.sha256) kubectl" | sha256sum --check
    ```
    验证通过时，输出为：
    ```bash 
    kubectl: OK
    ```
    验证失败时，`sha256`将以非零值退出，并打印如下输出：
    ```bash
    kubectl: FAILED
    sha256sum: WARNING: 1 computed checksum did NOT match
    ```
3. 安装 kubectl
   ```bash
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   ```
   即使你没有目标系统的 root 权限，仍然可以将 kubectl 安装到目录 `~/.local/bin` 中：

   ```bash
   chmod +x kubectl
   mkdir -p ~/.local/bin/kubectl
   mv ./kubectl ~/.local/bin/kubectl
   # 之后将 ~/.local/bin/kubectl 添加到 $PATH
   ```
4. 执行测试，以保障你安装的版本是最新的：

   ```bash
   kubectl version --client
   ```

### 用原生包管理工具安装

##### Ubuntu、Debian 或 HypriotOS  

   1. 更新 `apt` 包索引，并安装使用 Kubernetes `apt` 仓库所需要的包：  
        ```bash
        sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl
        ```
   2. 下载 Google Cloud 公开签名秘钥：
        ```bash
        sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
        ```
   3. 添加 Kubernetes apt 仓库：
        ```bash
        echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
        ```
   4. 更新`apt`包索引，使之包含新的仓库并安装 kubectl：
        ```bash
        sudo apt-get update && sudo apt-get install -y kubectl
        ```
##### 基于 Red Hat 的发行版
```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
sudo yum install -y kubectl
```

### 用其他包管理工具安装

##### Snap
如果你使用的 Ubuntu 或其他 Linux 发行版，内建支持 snap 包管理工具， 则可用 snap 命令安装 kubectl。
```bash
snap install kubectl --classic
kubectl version --client
```

##### Homebrew
如果你使用 Linux 系统，并且装了 Homebrew 包管理工具， 则可以使用这种方式安装 kubectl。
```bash
brew install kubectl
kubectl version --client
```

## 验证 kubectl 配置
为了让 kubectl 能发现并访问 Kubernetes 集群，你需要一个 kubeconfig 文件， 该文件在 kube-up.sh 创建集群时，或成功部署一个 Miniube 集群时，均会自动生成。 通常，kubectl 的配置信息存放于文件 ~/.kube/config 中。

通过获取集群状态的方法，检查是否已恰当的配置了 kubectl：
```bash
kubectl cluster-info
```
如果返回一个 URL，则意味着 kubectl 成功的访问到了你的集群。

如果你看到如下所示的消息，则代表 kubectl 配置出了问题，或无法连接到 Kubernetes 集群。

```bash
The connection to the server <server-name:port> was refused - did you specify the right host or port?
（访问 <server-name:port> 被拒绝 - 你指定的主机和端口是否有误？）
```

例如，如果你想在自己的笔记本上（本地）运行 Kubernetes 集群，你需要先安装一个 Minikube 这样的工具，然后再重新运行上面的命令。

如果命令`kubectl cluster-info`返回了 url，但你还不能访问集群，那可以用以下命令来检查配置是否妥当：
```bash
kubectl cluster-info dump
```

## 安装 kubectl convert 插件 

一个 Kubernetes 命令行工具 kubectl 的插件，允许你将清单在不同 API 版本间转换。 在将清单迁移到具有较新 Kubernetes 版本的未弃用 API 版本时，这个插件特别有用。 更多信息请访问 迁移到非弃用 API
    1. 用以下命令下载最新发行版:  
   ```bash
   curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert
   ```
   2. 验证该可执行文件（可选步骤）
   下载 kubectl-convert 校验和文件：
   ```bash
   curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert.sha256"
   ```
   基于校验和，验证 kubectl-convert 的可执行文件：
   ```bash
   echo "$(<kubectl-convert.sha256) kubectl-convert" | sha256sum --check
   ```
   验证通过时，输出为：
   ```bash
   kubectl-convert: OK
   ```
   验证失败时，sha256 将以非零值退出，并打印输出类似于：
   ```bash
   kubectl-convert: FAILED 
   sha256sum: WARNING: 1 computed checksum did NOT match
   ```
   3. 安装 kubectl-convert
   ```bash
   sudo install -o root -g root -m 0755 kubectl-convert /usr/local/bin/kubectl-convert
   ```
   4. 验证插件是否安装成功
   ```bash
   kubectl convert --help
   ```
   如果你没有看到任何错误就代表插件安装成功了。

