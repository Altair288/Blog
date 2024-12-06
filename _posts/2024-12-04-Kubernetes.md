---
layout: post
title: Ubuntu22.04 搭建Kubernetes 1.28版本集群
date: 2024-12-06
Author: 
categories: 
tags: [Kubernets]
comments: false
toc: true
---



# 依赖安装

准备工作需要在所有节点上进行。

- 安装 ssh 服务

    1.安装 `openssh-server`
    ```shell
    sudo apt-get install openssh-server
    ```

    2.修改配置文件
    ```shell
    vim /etc/ssh/sshd_config
    ```

    找到配置项
    ```shell
    LoginGraceTime 120PermitRootLogin prohibit-passwordStrictModes yes
    ```

    把 `prohibit-password` 改为 `yes`，如下：
    ```shell
    LoginGraceTime 120PermitRootLogin yesStrictModes yes
    ```

- 重启机器，并设置 root 密码
    ```shell
    rebootsudo passwd root
    ```
    
    设置主机名,保证每个节点名称都不相同
    ```shell
    hostnamectl set-hostname xxx
    ```

- 同步节点时间

    1.配置时间与时区
    ```shell
    timedatectl set-timezone Asia/Shanghai
    timedatectl set-ntp no
    apt install ntp -y
    systemctl enable ntp
    ```

    2.安装 ntpdate
    ```shell
    sudo apt-get -y install ntpdate
    ```

    3.配置 crontab，添加定时任务
    ```shell
    crontab -e

    0 */1 * * * ntpdate time1.aliyun.com
    ```

- 关闭 Swap
    关闭 Linux 的 swap 分区，提升 Kubernetes 的性能。
    ```shell
    # 确认 swap 是否启用
    sudo swapon --show

    # 暂时关闭 swap
    sudo swapoff -a

    # 永久关闭 swap
    sed -i '/swap/d' /etc/fstab
    ```

> 为什么要关闭 swap 交换分区？
> Swap 交换分区，如果机器内存不够，会使用 swap 分区，但是 swap 分区的性能较低，k8s 设计的时候为了能提升性能，默认是不允许使用交换分区的。Kubeadm 初始化的时候会检测 swap 是否关闭，如果没关闭，那就初始化失败。如果不想要关闭交换分区，安装k8s 的时候可以指定 --ignore-preflight-errors=Swap 来解决。

# 开始搭建

## 集群规划

每台上都安装 **docker-ce**、**docker-ce-cli**、**containerd.io**，使用 **Containerd** 作为容器运行时，和 **kubelet** 交互。

所有节点都安装 **kubelet、kubeadm、kubectl** 软件包，都启动 **kubelet**.**service** 服务。

## 配置 docker 和 k8s 的 APT源

k8s APT源新版配置方法

（比如需要安装 1.29 版本，则需要将如下配置中的 v1.28 替换成 v1.29）

```shell
apt-get update && apt-get install -y apt-transport-https
curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.28/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.28/deb/ /" |
    tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
```

旧版 kubernetes 源只更新到 1.28 部分版本，不过 docker 源部分可以用

```shell
sudo curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

sudo curl -fsSL https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
sudo add-apt-repository "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main"

apt update
```

在 Ubuntu 系统中，可以通过以下两个位置查看源列表：

``/etc/apt/sources.list`` 文件：这是主要的源列表文件。您可以使用文本编辑器（如 ``vi`` 或 ``nano``）以管理员权限打开该文件，查看其中列出的软件源。

``/etc/apt/sources.list.d/`` 目录：该目录包含额外的源列表文件。这些文件通常以 ``.list`` 扩展名结尾，并包含单独的软件源配置。

```shell
apt-cache madison kubelet # 命令来列出可用的 kubelet 软件包版本。检查是否存在版本号为 '1.28.8-00' 的软件包。

/etc/apt/sources.list # apt软件系统源
```

## 安装docker、containerd相关

```shell
apt install docker-ce docker-ce-cli containerd.io
```

使用 `apt` 可以查看安装的 `docker` 三个软件及关联软件。

```shell
apt list --installed | grep -i -E 'docker|containerd'
```

## 启动 docker 相关服务

ubuntu上的 dub 安装后，如果有服务，会被自动设置为开机自启动，且装完就会拉起，这里给出验证。

```shell
systemctl list-unit-files | grep -E 'docker|containerd'
###三个服务都应是running状态

systemctl status containerd.service
systemctl status docker.service
systemctl status docker.socket
```

## 配置containerd

Cgroup 管理器，k8s默认是 systemd，需要将 Containerd 的 Cgroup 管理器也修改为 systemd（默认是 cgroupfs）。

配置 Containerd ，如果 /etc/containerd 目录不存在，就先创建它：

```shell
mkdir /etc/containerd
```

生成默认配置：

```shell
containerd config default > /etc/containerd/config.toml
```

配置 containerd 改使用 systemd

```shell
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

```shell
vim /etc/containerd/config.toml
约125行，[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]段落
默认：
SystemdCgroup = false
改为：
SystemdCgroup = true

约61行，[plugins."io.containerd.grpc.v1.cri"]段落
默认：
sandbox_image = "registry.k8s.io/pause:3.6"
改为：
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
```

配置后，重启 containerd 服务，并保证 containerd 状态正确

```shell
systemctl restart containerd.service
systemctl status containerd.service
```

# 安装Kubernetes

安装 Kubernetes 需要在所有节点上进行。

```shell
/etc/apt/sources.list # apt软件系统源
apt-cache madison kubelet # 命令来列出可用的 kubelet 软件包版本。检查是否存在版本号为 '1.28.8-00' 的软件包。

ip route show # 查找以 "default via" 开头的行

sudo apt-get purge kubelet kubeadm kubectl # 卸妆
apt install docker-ce docker-ce-cli containerd.io
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
```

在 Ubuntu 系统中，可以通过以下两个位置查看源列表：

```shell
/etc/apt/sources.list 文件：这是主要的源列表文件。您可以使用文本编辑器（如 vi 或 nano）以管理员权限打开该文件，查看其中列出的软件源。

/etc/apt/sources.list.d/ 目录：该目录包含额外的源列表文件。这些文件通常以 .list 扩展名结尾，并包含单独的软件源配置。

清空kubectl describe node命令输出的事件日志
kubectl delete events --all --field-selector involvedObject.kind=Node,involvedObject.name=<节点名称>
可以省略--all参数，只删除特定类型的事件
```

```shell
apt install kubelet kubeadm kubectl
###如果需要指定安装1.28.2这个版本，则可以这样：
apt install kubelet=1.28.2-00 kubeadm=1.28.2-00 kubectl=1.28.2-00
```

确认 kubelet 服务状态
同 docker 一样，kubelet 安装后，服务会自动配置为开机启动，且服务已经启动

```shell
systemctl enable kubelet.service # 配置kubelet为开机自启动
systemctl status kubelet
```

这里没有启动成功是正常的，因为 kubelet 服务成功启动的先决条件，需要 kubelet 的配置文件，所在目录 /var/lib/kubelet 还没有建立。

可以用下面命令看日志，追踪到该临时问题。

```shell
journalctl -xeu kubelet
```

kubelet的正常启动，要等到下一步，master 节点做 kubeadm 的初始化后，才会正常。

## 版本锁定

锁定这三个软件的版本，避免意外升级导致版本错误。

```shell
sudo apt-mark hold kubeadm kubelet kubectl
```

## 下载 Kubernetes 组件镜像

可以通过下面的命令看到 kubeadm 默认配置的 kubernetes 镜像，是外网的镜像

```shell
kubeadm config images list
```

使用阿里的 kubernetes 镜像源，下载镜像

```shell
kubeadm config images pull --image-repository=registry.aliyuncs.com/google_containers --kubernetes-version=v1.28.2 --cri-socket /run/containerd/containerd.sock
```

# Kubernetes初始化

```shell
kubeadm init \
  --apiserver-advertise-address=<自己本机的公网IP> \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.28.2 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --ignore-preflight-errors=all \
  --cri-socket /run/containerd/containerd.sock
```

配置环境变量

```shell
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf
```

# 使用 Calico 网络插件

上述 master 节点初始化后，可以使用 kubectl get node 来检查 kubernetes 集群节点状态，当前 master 节点的状态为 NotReady，这是由于缺少网络插件，集群的内部网络还没有正常运作。

可以在 Calico 的网站（https://www.tigera.io/project-calico/）上找到它的安装方式，需要注意 Calico 版本支持适配的 kubernets 版本。

| Kubernetes 版本       | Calico 版本                                                                                  | Calico YAML文件                                                             |
|---------------------|--------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| 1.18、1.19、1.20 3.18 | https://projectcalico.docs.tigera.io/archive/v3.18/getting-started/kubernetes/requirements | https://projectcalico.docs.tigera.io/archive/v3.18/manifests/calico.yaml  |
| 1.19、1.20、1.21 3.19 | https://projectcalico.docs.tigera.io/archive/v3.19/getting-started/kubernetes/requirements | https://projectcalico.docs.tigera.io/archive/v3.19/manifests/calico.yaml  |
| 1.19、1.20、1.21 3.20 | https://projectcalico.docs.tigera.io/archive/v3.20/getting-started/kubernetes/requirements | https://projectcalico.docs.tigera.io/archive/v3.20/manifests/calico.yaml  |
| 1.20、1.21、1.22 3.21 | https://projectcalico.docs.tigera.io/archive/v3.21/getting-started/kubernetes/requirements | https://projectcalico.docs.tigera.io/archive/v3.21/manifests/calico.yaml  |
| 1.21、1.22、1.23 3.22 | https://projectcalico.docs.tigera.io/archive/v3.22/getting-started/kubernetes/requirements | https://projectcalico.docs.tigera.io/archive/v3.22/manifests/calico.yaml  |
| 1.21、1.22、1.23 3.23 | https://projectcalico.docs.tigera.io/archive/v3.23/getting-started/kubernetes/requirements | https://projectcalico.docs.tigera.io/archive/v3.23/manifests/calico.yaml  |
| 1.22、1.23、1.24 3.24 | https://projectcalico.docs.tigera.io/archive/v3.24/getting-started/kubernetes/requirements | https://projectcalico.docs.tigera.io/archive/v3.24/manifests/calico.yaml  |

```shell
curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/calico.yaml -O
```

Calico 使用的镜像较大，如果安装超时，可以考虑在每个节点上预先使用 docker pull 拉取镜像：

```shell
# 从calico.yaml文件中，找到需要下载的镜像源
docker pull docker.io/calico/kube-controllers:v3.27.3
docker pull docker.io/calico/node:v3.27.3
docker pull docker.io/calico/pod2daemon-flexvol:v3.27.3
docker pull docker.io/calico/cni:v3.27.3
```

Calico 安装使用 kubectl apply即可：

```shell
kubectl apply -f calico.yaml
```

# 其他节点加入集群

查看节点列表，这时还只有主节点

```shell
kubectl get node
```

主节点在初始化结束后，已经创建了临时 token，但该临时 token 只有24小时有效期。

因此这里需要重新在节点创建永久有效的 token

```shell
kubeadm token create --print-join-command 
```

worker节点加入

```shell
kubeadm join <master节点>:6443 --token oyl72q.dth6p8kwi7fopsd6 \
	--discovery-token-ca-cert-hash sha256:b31bb54c63a550d287c89ddd0094e27ca680a6c3386a8630a75445de3c4d6e43 \
  --cri-socket /run/containerd/containerd.sock
```

如果遇到拉取镜像的问题，同样使用以上方式下载到本地即可。

## 配置 Console 节点

Console 节点的部署工作更加简单，它只需要安装一个 kubectl，然后复制“config”文件就行，你可以直接在 Master 节点上用“scp”远程拷贝，例如：

```shell
scp `which kubectl` niuben@192.168.56.2:~/
scp ~/.kube/config niuben@192.168.56.2:~/.kube
```


## 卸载Kubernetes、docker相关

```shell
sudo apt-get purge kubelet kubeadm kubectl # 卸载
apt remove docker-ce docker-ce-cli containerd.io
```