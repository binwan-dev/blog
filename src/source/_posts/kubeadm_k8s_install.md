---
title: 使用 kubeadm 安装 kubernetes 集群
categories: 
- Kubenetes
---

本文主要介绍如何通过kubeadm安装一个高可用kubernetes集群。

### 1. 检查并设置主机环境
``` bash
# 关闭 swap （这一步必须要执行）
sudo swapoff -a
# 查看 swap 分区情况
sudo free -m
# 永久关闭 swap
sudo vim /etc/fstab
# 在swap分区这行前加 # 禁用掉，保存退出

# 修改机器名称来唯一标识机器地址
sudo hostnamectl set-hostname <node-name>

# 检查 br_netfilter
lsmod | grep br_netfilter # 检查是否输出有 br_netfilter
# 如果没有输出执行以下命令
sudo modprobe br_netfilter

# 允许 iptables 检查流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

### 2. 安装容器运行时
容器运行时有 Docker、Contrainerd 等多种，本文只描述 Docker、Contrainerd 安装。
 PS：Kubernetes 从 1.24.0 版本开始不在接受默认docker运行时，需要安装docker-shim，故推荐切换到Contrainerd运行时。另外，Docker从1.20.0版本开始底层已经依赖于Contrainerd运行时。

a. 安装 Docker 参考链接 https://docs.docker.com/engine/install/centos/

b. 安装 Containerd 安装方法参考 https://github.com/containerd/containerd
安装完成后执行下述命令配置：
``` bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# 配置SystemCgroup以及 pause 为中国源
vim /etc/containerd/config.toml
# 查找 [plugins."io.containerd.grpc.v1.cri".cni]
# 更改 SystemdCgroup = false 为 true
# 查找 sandbox_image 并修改 sandbox_image = "registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6" （此处可以修改为私有源）
systemctl restart containerd
systemctl enable containerd

#如果机器上还有docker 
systemctl disable docker.service
```

### 3. 安装kubeadm、kubectl、kubelet
``` bash
######## yum 安装 ########
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
         https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
keepcache = 0
EOF
sudo yum list --showduplicates kubeadm --disableexcludes=kubernetes # 查看需要安装的版本（推荐安装比最新版本低两个次版本）
sudo yum install -y kubeadm-<version> kubectl-<version> kubelet-<version>

# 添加环境变量 kubectl
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bashrc
source ~/.bashrc
```

### 4. 安装第一个控制节点
生成 kubeadm-config
``` bash 
cat <<EOF | sudo tee ./kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
nodeRegistration:
  criSocket: <container-sock>
  imagePullPolicy: IfNotPresent
---
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: <kubernetes-version>
controlPlaneEndpoint: "<loadbalancer-address>:<port>"
networking:
  dnsDomain: cluster.local
  podSubnet: ""
  serviceSubnet: 10.96.0.0/12
scheduler: {}
EOF
```
a. 需要修改 <container-sock> docker运行时位置：/var/run/docker.sock  containerd运行时位置：/var/run/containerd/containerd.sock
b. 需要修改 <kubernetes-version> 也就是刚才安装的kubeadm 版本。
c. 需要修改 <loadbalancer-address>:<port> kubectl链接控制kubernetes集群的地址。（推荐使用域名，解析到所有master节点上）

执行kuberadm init 初始化集群
``` bash
kubeadm init --config ./kubeadm-config.yaml --upload-certs # upload-certs 自动上传集群证书到各个节点

# 执行完之后会输出
> kubeadm join <loadbalancer-address>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash> \
>	--control-plane --certificate-key <key>

> Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
> As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
> "kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

> Then you can join any number of worker nodes by running the following on each as root:

> kubeadm join <loadbalancer-address>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>

```
后续work和master节点加入按照输出的token和证书key相应加入。

### 5. 安装第二个或多个控制节点

依次执行 步骤 1-3 所有命令。
执行kubeadm join
``` bash
kubeadm join <loadbalancer-address>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash> --control-plane --certificate-key <key>
```
> **WARNING**
> 注意 Token 默认只有24小时有效
> 若失效点击此处查看[生成token、cert-hash、certificate-key](/2022/06/02/kubeadm-used/)

### 6. 安装work节点
依次执行 步骤 1-3 所有命令。
执行kubeadm join
``` bash
kubeadm join <loadbalancer-address>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
> **WARNING**
> 注意 Token 默认只有24小时有效
> 若失效点击此处查看[生成token、cert-hash、certificate-key](/2022/06/02/kubeadm-used/)

至此一个高可用的kubernetes集群已经安装完成，尽情享受（折腾）吧！