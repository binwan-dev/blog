---
title: K8S 手动续期证书  
categories: 
- Kubenetes
---

使用 kubeadm 安装 kubernetes 集群非常方便，但是也有一个比较烦人的问题就是默认的证书有效期只有一年时间，所以需要考虑证书升级的问题，本文的演示集群版本为 v1.18.18 版本，不保证下面的操作对其他版本也适用，在操作之前一定要先对证书目录进行备份，防止操作错误进行回滚。本文主要介绍两种方式来更新集群证书。

## 手动更新证书

由 kubeadm 生成的客户端证书默认只有一年有效期，我们可以通过 ```check-expiration``` 命令来检查证书是否过期：  
``` bash
$ kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Apr 24, 2023 02:51 UTC   364d                                    no
apiserver                  Apr 24, 2023 02:53 UTC   364d            ca                      no
apiserver-etcd-client      Apr 24, 2023 02:53 UTC   364d            etcd-ca                 no
apiserver-kubelet-client   Apr 24, 2023 02:53 UTC   364d            ca                      no
controller-manager.conf    Apr 24, 2023 02:51 UTC   364d                                    no
etcd-healthcheck-client    Apr 24, 2023 02:45 UTC   364d            etcd-ca                 no
etcd-peer                  Apr 24, 2023 02:45 UTC   364d            etcd-ca                 no
etcd-server                Apr 24, 2023 02:45 UTC   364d            etcd-ca                 no
front-proxy-client         Apr 24, 2023 02:53 UTC   364d            front-proxy-ca          no
scheduler.conf             Apr 24, 2023 02:51 UTC   364d                                    no

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Apr 20, 2031 10:49 UTC   8y              no
etcd-ca                 Apr 20, 2031 10:49 UTC   8y              no
front-proxy-ca          Apr 20, 2031 10:49 UTC   8y              no
```
该命令显示 /etc/kubernetes/pki 文件夹中的客户端证书以及 kubeadm 使用的 KUBECONFIG 文件中嵌入的客户端证书的到期时间/剩余时间。
> **NOTE**
> kubeadm 不能管理由外部 CA 签名的证书，如果是外部得证书，需要自己手动去管理证书的更新。

另外需要说明的是上面的列表中没有包含 kubelet.conf，因为 kubeadm 将 kubelet 配置为自动更新证书。

另外 kubeadm 会在控制面板升级的时候自动更新所有证书，所以使用 kubeadm 搭建得集群最佳的做法是经常升级集群，这样可以确保你的集群保持最新状态并保持合理的安全性。但是对于实际的生产环境我们可能并不会去频繁得升级集群，所以这个时候我们就需要去手动更新证书。

要手动更新证书也非常方便，我们只需要通过 kubeadm alpha certs renew 命令即可更新你的证书，这个命令用 CA（或者 front-proxy-CA ）证书和存储在 /etc/kubernetes/pki 中的密钥执行更新。
> **NOTE**
> 如果是高可用集群，需要在所有Master节点上都要执行。

### 更新证书
```bash
# 默认地址为 /etc/kubernetes，使用时请注意替换为自己的kubenetes地址。
# 备份证书
$ mkdir /etc/kubernetes.bak 
$ cp -r /etc/kubernetes/pki/ /etc/kubernetes.bak
$ cp /etc/kubernetes/*.conf /etc/kubernetes.bak

# 备份 etcd 数据目录
$ cp -r /var/lib/etcd /var/lib/etcd.bak

# 执行更新证书的命令
$ kubeadm alpha certs renew all --config={kubeadm.yaml} # kubeadm.yaml 记得替换

# 更新下 kubeconfig 文件
$ kubeadm init phase kubeconfig all --config {kubeadm.yaml} # kubeadm.yaml 记得替换

# 替换 admin 配置文件
$ mv $HOME/.kube/config $HOME/.kube/config.old
$ cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ chown $(id -u):$(id -g) $HOME/.kube/config

# 重启 kube-apiserver、kube-controller、kube-scheduler、etcd 这4个容器. 
$ docker restart {apiserver_container}
$ docker restart {controller_container}
$ docker restart {scheduler_container}
$ docker restart {etcd_container}
```
> **NOTE**
> 以上命令需要在所有 Master 节点上运行。

以上命令即可完成手动更新，使用一下命令来验证证书更新。
``` bash
$ echo | openssl s_client -showcerts -connect 127.0.0.1:6443 -servername api 2>/dev/null | openssl x509 -noout -enddate
notAfter=Apr 24 02:53:20 2023 GMT
```
可以看到现在的有效期是一年过后的，证明已经更新成功了。
