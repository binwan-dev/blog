---
title: Kubeadm 基本使用
date: 2022-05-12 22:28:33
categories: 
- Kubenetes
---

# Token
1. 查看有效Token列表
``` bash
kubeadm token list
```
2. 生成 Join 集群 Token
```bash
kubadm token create --print-join-command  # 默认有效期24小时,若想久一些可以结合–ttl参数,设为0则用不过期
```
# Cert
1. 生成 cert-hash
```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```
2. 查看 certificate-key 
```bash
kubeadm certs certificate-key
```
3. 刷新 certificate-key 
```bash
 kubeadm init phase upload-certs --upload-certs
```
