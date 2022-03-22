# 在 Windows 上安装 kubectl

## 准备开始
kubectl 版本和集群版本之间的差异必须在一个小版本号内。 例如：v1.23 版本的客户端能与 v1.22、 v1.23 和 v1.24 版本的控制面通信。 用最新兼容版的 kubectl 有助于避免不可预见的问题。

## 在 Windows 上安装 kubectl
在 Windows 系统中安装 kubectl 有如下几种方法：
- 用 curl 在 Windows 上安装 kubectl
- 在 Windows 上用 Chocolatey 或 Scoop 安装

### 用 curl 在 Windows 上安装 kubectl
1. 下载 [最新发行版 v1.23.0](https://dl.k8s.io/release/v1.23.0/bin/windows/amd64/kubectl.exe)。
   如果你已安装了 curl,也可以使用此命令：
   ```powershell
   curl -LO "https://dl.k8s.io/release/v1.23.0/bin/windows/amd64/kubectl.exe"
   ```
   说明： 要想找到最新稳定的版本（例如：为了编写脚本），可以看看这里 https://dl.k8s.io/release/stable.txt。
2. 验证该可执行文件（可选步骤）
   下载 kubectl 校验和文件：
   ```bash
   curl -LO "https://dl.k8s.io/v1.23.0/bin/windows/amd64/kubectl.exe.sha256"
   ```
   基于校验和文件，验证 kubectl 的可执行文件：
   - 在命令行环境中，手工对比`CertUtil`命令的输出与校验和文件：
        ```powershell
        CertUtil -hashfile kubectl.exe SHA256
        type kubectl.exe.sha256
        ```
   - 用`PowerShell`自动验证，用运算符 -eq 来直接取得 True 或 False 的结果：
        ```powershell
        $($(CertUtil -hashfile .\kubectl.exe SHA256)[1] -replace " ", "") -eq $(type .\kubectl.exe.sha256)
        ```
3. 将 kubectl 二进制文件夹附加或添加到你的`PATH`环境变量中。
4. 测试一下，确保此`kubectl`的版本和期望版本一致：
    ```powershell
    kubectl version --client
    ```
    或者使用下面命令来查看版本的详细信息：
    ```powershell
    kubectl version --client --output=yaml     
    ```
    说明： `Windows`版的`Docker Desktop`将其自带版本的`kubectl`添加到 PATH。 如果你之前安装过 Docker Desktop，可能需要把此 PATH 条目置于 Docker Desktop 安装的条目之前， 或者直接删掉 Docker Desktop 的 kubectl。

### 在 Windows 上用 Chocolatey 或 Scoop 安装
1. 要在 Windows 上安装 kubectl，你可以使用包管理器`Chocolatey`或是命令行安装器`Scoop`。
   - Chocolatey
        ```powershell 
        choco install kubernetes-cli
        ```
   - Scoop
        ```powershell 
        scoop install kubectl
        ```
2. 测试一下，确保安装的是最新版本：
    ```powershell
    kubectl version --client
    ```
3. 导航到你的 home 目录：
    ```powershell
    # 当你用 cmd.exe 时，则运行： cd %USERPROFILE%
    cd ~
    ```
4. 创建目录 `.kube`：
    ```powershell
    mkdir .kube
    ```
5. 切换到新创建的目录 `.kube`
    ```powershell
    cd .kube
    ```
6. 配置 kubectl，以接入远程的 Kubernetes 集群：
   ```powershell
   New-Item config -type file
   ```
   说明： 编辑配置文件，你需要先选择一个文本编辑器，比如 Notepad。

