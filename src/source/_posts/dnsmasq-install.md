---
title: Dnsmasq DNS Server 安装配置
date: 2023-06-01 21:24:33
categories: 
- 脚手架
- DNS
---

本文讲解如何配置一个内网的公共 `DNS Server`，以及如何拦截域名解析

1. 安装 `Dnsmasq`
   这里只给出了 `Ubuntu`
   ``` shell
   apt install -y dnsmasq
   ```
   
2. 配置 `Dnsmasq`
   ``` shell
   vim /etc/dnsmasq.conf
   
   # 修改文件内容
   resolv-file=/etc/resolv.dnsmasq.conf  # 这个参数表示 `Dnsmasq 会从这个指定的文件中寻找上游 DNS 服务器 	
   strict-order  # 取消注释，表示严格按照 resolv-file 文件中的顺序从上到下进行 DNS 解析, 直到第一个成功解析成功为止 	
   listen-address=0.0.0.0  # 0.0.0.0改成服务器公网IP
   addn-hosts=/etc/dnsmasq.hosts  # 表示加载 dnsmasq.hosts 中的配置，此文件作用类似于我们电脑中的 Host 文件
   ```
   
   ``` shell
   vim /etc/resolv.dnsmasq.conf
   
   # 修改文件内容
   nameserver 223.5.5.5
   nameserver 223.6.6.6
   ```
   
   ``` shell
   vim /etc/dnsmasq.hosts
   
   #将广告域名指向到 127.0.0.1 实现广告屏蔽
   127.0.0.1  ad.youku.com
   ```

3. 启动与测试
   ``` shell
   systemctl restart dnsmasq
   ```
   
   > 如果无法启动，请 `Htop` 检查是否存在已有其他服务启动的 `Dnsmasq`，或者是检查并关停 `systemctl stop system-resolved`。
   
   然后将我们电脑的 `DNS` 服务器指向这台安装了 `Dnsmasq` 的服务器 `IP` 地址即可。
   ```
