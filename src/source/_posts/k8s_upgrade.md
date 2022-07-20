---
title: K8S 集群更新  
categories: 
- Kubenetes
---

本文主要介绍使用 kubeadm 来对 kubernetes 进行升级更新。
大致步骤如下：
### 1. 查找并下载新版本
控制节点和工作节点都需要执行
> **NOTE**
> 注意：需要更新的版本和当前版本不能跳级，必须迭代升级。例如当前版本 1.17.x，只可以下载更新 1.18.x，不可以直接更新到 1.19.x 以及之后版本。

####  ubuntu debian 系
``` bash
apt update
apt-cache madison kubeadm
# 在列表中查找最新的 1.24 版本
# 它看起来应该是 1.24.x-00，其中 x 是最新的补丁版本

# 用新的补丁版本替换 1.24.x-00 中的 x
apt-mark unhold kubelet kubectl kubeadm && \
apt-get update && apt-get install -y kubelet=1.24.x-00 kubectl=1.24.x-00  kubeadm=1.24.x-00 && \
apt-mark hold kubelet kubectl kubeadm
```

#### centos RHEL Fedora
``` bash
yum list --showduplicates kubeadm --disableexcludes=kubernetes	
# 在列表中查找最新的 1.24 版本
# 它看起来应该是 1.24.x-00，其中 x 是最新的补丁版本

# 用新的补丁版本号替换 1.24.x-00 中的 x
yum install -y kubelet-1.24.x-0 kubectl-1.24.x-0 kubeadm-1.24.x-0 --disableexcludes=kubernetes
```

### 2. 升级 kubernetes

有需要可以排空节点，此步骤不是必须。
```bash
kubectl drain <node-to-drain> --ignore-daemonsets
```

> **NOTE**
> 升级分为控制节点和其他控制节点和工作节点升级，二者升级步骤不同，请注意！
#### 具有 /etc/kubernetes/admin.conf 的控制节点执行
``` bash
# 查看升级几乎
kubeadm upgrade plan
# 如有提示需要手动更新的，则需要进行手动更新。
kubeadm upgrade apply v1.24.x
# 执行后看到如下输出则视为升级成功
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.24.x". Enjoy!

# 执行重启kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### 其他控制节点以及工作节点执行
其他节点直接执行 kubeadm upgrade node 即可，无需查看升级计划以及Apply
``` bash
kubeadm upgrade node
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

升级完成，可以运行 kubectl get node 查看容器版本。
