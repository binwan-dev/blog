---
title: GitLab 安装
date: 2023-06-01 21:24:33
categories: 
- 脚手架
- GitLab
---

本文讲述如何使用 `Docker` 安装 `GitLab`

1. 执行运行脚本

   ``` shell
   docker run -d  -p 6443:443 -p 680:80 -p 6222:22 --name gitlab --restart always -v /gitlab/config:/etc/gitlab -v /gitlab/logs:/var/log/gitlab -v /gitlab/data:/var/opt/gitlab --name gitlab gitlab/gitlab-ce
   # -p 6222 为 Git Clone 时候ssh的端口
   # -p 443 一般无需设置，除非直接对外提供https服务，一般我们用 Nginx 之内的反向代理
   # -d：后台运行
   # -p：将容器内部端口向外映射
   # --name：命名容器名称
   # -v：将容器内数据文件夹或者日志、配置等文件夹挂载到宿主机指定目录
   ```

2. 配置
   按上面的方式，`GitLab` 容器运行没问题，但在 `GitLab` 上创建项目的时候，生成项目的 `URL` 访问地址是按容器的 `HostName` 来生成的，也就是容器的 `ID`。作为 `GitLab` 服务器，我们需要一个固定的 `URL` 访问地址，于是需要配置 `GitLab.rb`（宿主机路径：`/gitlab/config/gitlab.rb`）。

   ```shell
   # gitlab.rb文件内容默认全是注释
   vim /home/gitlab/config/gitlab.rb
   ```

   ``` shell
   # 配置http协议所使用的访问地址,不加端口号默认为80
   external_url 'http://gitlab.xxx.com'

   # 配置ssh协议所使用的访问地址和端口
   gitlab_rails['gitlab_ssh_host'] = 'gitlab.xxx.com'
   gitlab_rails['gitlab_shell_ssh_port'] = 6222 # 此端口是 run 时 22 端口映射的 6222 端口
   :wq #保存配置文件并退出
   ```

   ``` shell
   # 重启 gitlab 容器
   docker restart gitlab
   ```

   ``` shell
   # 配置 Nginx 反向代理
   vim /etc/nginx/conf.d/gitlab.xxx.com.conf

   # 内容
   server {
       listen       80;
       server_name gitlab.xxx.com;
       rewrite ^(.*)$ https://gitlab.xxx.com$1 permanent;
   }

   server {
       listen 443 ssl;	
       server_name gitlab.xxx.com;

       ssl_certificate "fullchain.pem";
       ssl_certificate_key "privkey.pem";
       ssl_session_cache shared:SSL:1m;
       ssl_session_timeout  10m;
       ssl_ciphers HIGH:!aNULL:!MD5;
       ssl_prefer_server_ciphers on;

       location / {
 	       ## check for goget AND /namespace/project
      	   if ($args ~* "^go-get=1") {
		       set $condition goget;
		   }
           if ($uri ~ ^/([a-zA-Z0-9_-]+)/([a-zA-Z0-9_-]+)$) {
		       set $condition "${condition}path";
           }
           if ($condition = gogetpath) {
		       return 200 "<!DOCTYPE html><html><head><meta content='gitlab.xxx.com$uri git ssh://git@gitlab.xxx.com:6222$uri.git' name='go-import'></head></html>";
           }

           proxy_pass http://127.0.0.1:680;
          	proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection keep-alive;
           proxy_set_header Host $host;
           proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
           proxy_cache_bypass $http_upgrade;
           client_max_body_size 1024m;
       }
   }

   # Nginx reload
   nginx -s reload
   ```

3. 其他
   第一次访问时候一般会设置 `Root` 的密码，之后就不需要了。另外建议使用 `Nginx` 反向代理 `GitLab` 的 `HTTP` 请求。
