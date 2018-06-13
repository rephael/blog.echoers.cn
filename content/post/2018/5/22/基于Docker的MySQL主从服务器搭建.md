---
title: "基于Docker的MySQL主从服务器搭建"
date: 2018-05-22T12:50:09+08:00
Categories: [blog]
Description: "数据库备份用"
Tags: [MySQL,Docker]
draft: false
---

基于Docker的MySQL主从数据库搭建
<!--more-->

### master主数据库
#### 新增mysql配置文件mysql-master.cnf
```conf
[mysqld]
log-bin=mysql-bin
server-id=161
```
> 注：server-id一般为ip最后一段

#### 编写Dockerfile
```Dockerfile
FROM mysql:latest
MAINTAINER Raphael Zhang
COPY mysql-master.cnf /etc/mysql/conf.d/
EXPOSE 3306
CMD ["mysqld"]
```
> docker.io的mysql镜像配置文件结构与nginx类似，通过以上脚本导入配置即可。

#### 构建docker镜像
```bash
docker build -t mysql-master:latest .
```
#### 创建容器
```bash
docker run --name mysql-master -p 3306:3306 -v $HOME/mysql/conf.d:/etc/mysql/conf.d -v $HOME/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD={your passwd} -d mysql-master:latest
```
#### 登陆到主数据库容器，查看master状态
```bash
docker exec -it mysql-master bash
```
![image](http://note.youdao.com/yws/api/personal/file/WEB4533efb44916ce43e172553c1f0a077b?method=download&shareKey=2c4bc6a4f9d6559bfc46a7ef7d556e62)
#### 主数据库创建用户
```sql
SET sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
GRANT REPLICATION SLAVE ON *.* to 'backup'@'%' identified by 'mysql_master';
```
### slave从数据库
> 创建方式与master方式类似

#### 新增mysql配置文件mysql-slave.cnf
```conf
[mysqld]
log-bin=mysql-bin
server-id=163
```
> 注：server-id一般为ip最后一段

#### 编写Dockerfile
```Dockerfile
FROM mysql:latest
MAINTAINER Raphael Zhang
COPY mysql-slave.cnf /etc/mysql/conf.d/
EXPOSE 3306
CMD ["mysqld"]
```
> 镜像构建过程类似，不再赘述

#### 登陆从数据库，设置相关参数
```sql
SET sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
change master to master_host='172.31.196.161',master_user='backup',master_password='mysql_master',master_log_file='mysql-bin.000002',master_log_pos=439;
```
> master_host为docker的地址不能写127.0.0.1  
master_user是在主库创建的用户   
master_log_pos是主库show master status;查询出的Position

#### 启动服务
```bash
start slave;
```
#### 查看服务状态
```bash
show slave status;
```
显示
```bash
Waiting for master to send event
```
即是成功了。