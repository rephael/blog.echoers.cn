---
title: "基于Gogs与Drone的CICD平台搭建"
date: 2018-05-22T11:56:50+08:00
Categories:[blog]
Description:"比较现代化的替代方案"
Tags:[MySQL,Docker]
draft: true
---

CI/CD是DevOps中不可或缺的流程之一，在项目组建初期，根据以往的经验，选择了gitlab+jenkins的组合。这两个工具有着比较悠久的历史，功能非常完备，是非常值得推荐的工具组合。
<!--more-->

* gitlab
> gitlab基于ROR框架开发，目前已经迭代到了10个大版本，是市面上比较成熟的git工具之一。由于历史悠久，功能已经是相当完善，不仅仅局限于代码管理，现在已经发展成为了一个完整的项目周期管理平台，集成了git、ci、scrume等非常前沿的工具。

* jenkins
> java语言开发，插件系统非常完善，支持几乎所有的平台与语言，功能强大，是比较主流的CI/CD平台。
#### 为什么选择gogs+drone
* gitlab是ruby写的，jenkins是java写的，而且项目都比较庞大，所以对硬件的要求比较高，gitlab要求系统至少4G的内存，jenkins至少2G，所以要想在一台服务器上装这两个，准备至少8G的系统内存吧。
* 最近对golang比较上心，语言在性能与开销之间比较平衡，比较适合我这种对硬件比较抠门的人。
* 逼格高，可以用来炫技。
### 安装
* 通过docker-compose安装，对应的`docker-compose.yml`文件内容如下：

```Dockerfile
version: '2'

services:
  gogs:
    image: gogs/gogs:latest
    ports:
      - "10022:22"
      - 3000:3000
    volumes:
      - /vagrant/gogs-data:/data
    restart: always
  mysql:
    image: mysql:latest
    ports:
      - 3306:3306
    volumes:
      - /vagrant/mysql-data:/var/lib/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=z94264f3264t826
      - MYSQL_DATABASE=gogs
  drone-server:
    image: drone/drone:latest
    ports:
      - 8000:8000
      - 9000:9000
    volumes:
      - /var/lib/drone:/var/lib/drone
    restart: always
    environment:
      # 开启注册，此配置允许任何人自注册和登录系统
      - DRONE_OPEN=true
      #直接配置172.17.32.212:9000 会报错
      - DRONE_HOST=http://172.17.32.212:9000
      # 设置管理员用户
      - DRONE_ADMIN=admin
      # 开启Gogs驱动
      - DRONE_GOGS=true
      # Gogs服务器地址
      - DRONE_GOGS_URL=http://172.17.32.212:3000
      # 此SECRET为任意值
      - DRONE_SECRET=rotkube
  drone-agent:
    image: drone/agent:latest
    command: agent
    restart: always
    depends_on:
      - drone-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      # Drone Server 地址，无需加http://
      - DRONE_SERVER=172.17.32.212:9000
      # 与Drone Server一致即可
      - DRONE_SECRET=rotkube
```
* 部署docker-compose服务
```bash
$ docker-compose up -d
```
这样就完成了工具的部署

