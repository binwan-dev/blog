---
title: K8S docker迁移到containerd  
date: 2022-07-20 22:28:33
categories: 
- Kubenetes
---

由于 kubernetes 从1.24.0版本开始默认禁止使用docker，所以想要升级到1.24.0需要从docker迁移到containerd。

整个升级步骤总共分为：
1. 安装并配置containerd；
2. 排空节点服务，停止并移除docker服务（不是必须），停止kubelet服务；
3. 配置kubelet使用containerd作为容器运行时；
4. 重启kubelet服务并验证节点；

#### 1. 安装并配置containerd
由于docker也已切换到containerd上，可以直接参考官方docker安装教程。
此处参考docker安装即可，查看 https://docs.docker.com/engine/install/centos

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
```

#### 2. 排空节点服务停止服务
``` bash
kubectl drain <node-name> --ignore-daemonsets
systemctl stop kubelet
systemctl stop docker.service
systemctl disbale docker.service(不是必须)
```

#### 3. 配置kubelet使用containerd作为容器运行时
``` bash
kubectl edit no <node-name>
# 这一命令会打开一个文本编辑器，供你在其中编辑 Node 对象。 要选择不同的文本编辑器，你可以设置 KUBE_EDITOR 环境变量。
# 更改 kubeadm.alpha.kubernetes.io/cri-socket 值，将其从 /var/run/dockershim.sock 改为你所选择的 CRI 套接字路径 （例如：unix:///run/containerd/containerd.sock）。
# 注意新的 CRI 套接字路径必须带有 unix:// 前缀。
# 保存文本编辑器中所作的修改，这会更新 Node 对象。

# 编辑文件 /var/lib/kubelet/kubeadm-flags.env，将 containerd 运行时添加到标志中： --container-runtime=remote 和 --container-runtime-endpoint=unix:///run/containerd/containerd.sock"。

systemctl restart kubelet
```
#### 4. 验证节点
输出如下面所示。CONTAINER-RUNTIME 列给出容器运行时及其版本。
``` bash
kubectl get nodes -o wide
```

