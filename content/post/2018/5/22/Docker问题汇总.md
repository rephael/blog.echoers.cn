---
title: "Docker问题汇总"
date: 2018-05-22T16:18:00+08:00
Categories: [blog]
Description: "题主在安装配置docker时遇到的问题总结"
Tags: [Docker]
draft: false
---
Docker安装配置以及遇到的一些问题及解决方案记录
<!--more-->
### docker 安装
* centos7安装dokcer ce版
> 注：通过yum直接安装的版本过老，推荐使用以下方式安装
* 如果有旧版本安装，先删除老版本
```bash
$ sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
```
* 安装docker ce
    * 安装依赖组件
    ```bash
    $ sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
    ```
    * 设置稳定安装源
    ```bash
    $ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    ```
    * 安装源可选项
    ```bash
    sudo yum-config-manager --enable/--disable docker-ce-edge (edge源)
    sudo yum-config-manager --enable/--disable docker-ce-test (test源)
    ```
    > 一般选择stable源
    
    * 安装docker ce
    ```bash
    sudo yum install docker-ce
    ```
* 安装之后服务配置
  + 允许开机自启动服务
    ```bash
    sudo systemctl enable docker
    ```
  + 启动docker服务
    ```bash
    sudo systemctl start docker
    ```
### 免root使用docker命令
* 添加docker组
```bash
sudo groupadd docker
```
* 将用户加入该 group 内，然后退出并重新登录就生效（这一步貌似非必须）
```bash
sudo gpasswd -a ${USER} docker
```
* 重启 docker 服务
```bash
sudo systemctl restart docker
```
* 切换当前会话到新 group 或者重启 X 会话
```bash
newgrp - docker
```
> 注意:最后一步是必须的，否则因为 groups 命令获取到的是缓存的组信息，刚添加的组信息未能生效，所以 docker images 执行时同样有错。

### docker本地镜像源配置
### docker pull https问题(老版本)
* 修改配置文件
```bash
sudo vim /etc/sysconfig/docker
```
添加如下内容：
```conf
ADD_REGISTRY='--add-registry 192.168.1.247:5000'
OPTIONS='--insecure-registry 192.168.1.247:5000'
```

> 注意：这个问题一般是在拉取本地镜像源镜像的时候出现，其中链接为本地镜像源地址

### docker ce17 pull https问题
* 添加文件/etc/docker/daemon.json，并添加以下内容重启docker即可
```conf
{"insecure-registries":["192.168.1.247:5000"]}
```
### docker registry v2使用
* 查看所有的镜像
```bash
curl http://192.168.1.247:5000/v2/_catalog
```
* 查看某一镜像的TAG Lsit
```bash
curl http://192.168.1.247:5000/v2/{image_name}/tags/list
```
### docker 允许远程api调用
* 查看配置文件位于哪里
```bash
systemctl show --property=FragmentPath docker 
```
* 编辑配置文件内容，接收所有IP请求
```bash
sudo vim /usr/lib/systemd/system/docker.service  
修改内容：
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2376
```
* 重新加载配置文件，重启docker daemon
```bash
sudo systemctl daemon-reload     
sudo systemctl restart docker 
```
* 清理所有停止的容器
```bash
docker container prune 
```
* 清理所有不用数据(停止的容器,不使用的volume,不使用的networks,悬挂的镜像)
```bash
docker system prune -a
```
* docker时间同步
```bash
ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai > /etc/timezone
```
* 清理多余的网桥
```bash
docker network disconnect --force bridge XXX
```
* 清理被删除tag的镜像文件

> 通过harbor删除tag之后，镜像并不会被删除，需要额外执行清除命令

```bash
docker exec registry registry garbage-collect /etc/registry/config.yml
```
### docker 端口映射错误解决方法
* 某次手贱升级系统之后，启动docker出现该问题，记录解决方案，问题如下：
```bash
COMMAND_FAILED: '/sbin/iptables -t nat -A DOCKER -p tcp -d 0/0 --dport 8111 -j DNAT --to-destination 172.17.0.6:8111 ! -i docker0' failed: iptables: No chain/target/match by that name.
```
* 解决方案
  - pkill docker
  - iptables -t nat -F
  - ifconfig docker0 down
  - brctl delbr docker0
  - 重启docker后解决