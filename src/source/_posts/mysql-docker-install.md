---
title: 使用 Docker 安装 MySQL 
date: 2023-06-06 18:50:41
categories: 
- 脚手架
- MySQL
---

本文主要介绍如何使用 `Docker` 安装 `MySQL`

访问 MySQL 镜像库地址：https://hub.docker.com/_/mysql?tab=tags 。

可以通过 Sort by 查看其他版本的 MySQL，默认是最新版本 mysql:latest 。

安装
   ``` shell
   # 安装测试版本用来拷贝配置文件
   docker run -d -e MYSQL_ROOT_PASSWORD=123456 --name mysql_test mysql:<tag>    
   docker exec -it mysql_test mysql --help | grep my.cnf # 查找Docker内，MySQL配置文件my.cnf的位置
   mkdir -p /<your mysql storage path>/mysql/conf # 创建本地 mysql 配置文件夹
   mkdir -p /<your mysql storage path>/mysql/data # 创建本地 mysql 数据文件夹
   docker cp mysql_test:/<docker_path>/my.cnf /<your mysql storage path>/mysql/conf
   docker stop mysql_test && docker rm mysql_test
   
   # 安装 mysql
   docker run -d -p <your mysql port>:3306 -e MYSQL_ROOT_PASSWORD=<your mysql root password> -v /<your mysql storage path>/mysql/conf:/etc/mysql -v /<your mysql storage path>/mysql/data:/var/lib/mysql --name <mysql-name> mysql:<tag>
   ```
