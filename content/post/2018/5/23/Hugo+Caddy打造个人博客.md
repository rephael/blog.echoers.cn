---
title: "Hugo+Caddy打造个人博客"
date: 2018-05-23T16:43:41+08:00
Categories: [Blog]
Description: "搭建私有博客"
Tags: [hugo,caddy]
draft: false
---

Hugo+Caddy打造个人博客，自动化构建博客系统，提交博客到Github即自动部署。
<!--more-->

### 前言
很久之前就想要搭建个人博客，在各种模板引擎之间无限徘徊，WordPress、Hexo、Jekyll都多多少少接触过，但是由于各种原因吧，一直都没有落地。很多人都推荐通过github部署博客，但是由于伟大的长城访问速度实在堪忧，因此便有了在私有服务器上部署的念头。因个人原因，现在偏向于golang，就顺水推舟选择了[Hugo](http://gohugo.io)，也有[中文文档](http://www.gohugo.org)，使用起来还是挺方便的。同时因为做过DevOps的缘故，非常喜欢自动化部署，经过长时间的摸索，最终确定了[Caddy](https://caddyserver.com)+Hugo的模式。
### Caddy的安装与配置
#### 二进制安装
> 在Caddy[官方下载](https://caddyserver.com/download)页面下载可执行文件，记的勾选git与Hugo两个插件，会自编译生成压缩包，里面包含二进制包、安装指导还有自启动脚本等。
* 将二进制包拷贝到系统路径，并付给相应的权限
```bash
sudo cp /path/to/caddy /usr/local/bin
sudo chown root:root /usr/local/bin/caddy
sudo chmod 755 /usr/local/bin/caddy
```
* 设置caddy允许绑定80，443等端口
```bash
sudo setcap 'cap_net_bind_service=+ep' /usr/local/bin/caddy
```
* 为Caddy设置专有用户、用户组与文件夹
```bash
sudo groupadd -g 33 www-data
sudo useradd \
  -g www-data --no-user-group \
  --home-dir /var/www --no-create-home \
  --shell /usr/sbin/nologin \
  --system --uid 33 www-data

sudo mkdir /etc/caddy
sudo chown -R root:www-data /etc/caddy
sudo mkdir /etc/ssl/caddy
sudo chown -R root:www-data /etc/ssl/caddy
sudo chmod 0770 /etc/ssl/caddy
```
* 将自己的Caddy配置文件放到/etc/caddy，并修改权限
```bash
sudo cp /path/to/Caddyfile /etc/caddy/
sudo chown www-data:www-data /etc/caddy/Caddyfile
sudo chmod 444 /etc/caddy/Caddyfile
```
* 创建服务器祝目录
```bash
sudo mkdir /var/www
sudo chown www-data:www-data /var/www
sudo chmod 555 /var/www
```
* 添加Caddy服务（Centos7版）
```bash
wget https://raw.githubusercontent.com/mholt/caddy/master/dist/init/linux-systemd/caddy.service
sudo cp caddy.service /etc/systemd/system/
sudo chown root:root /etc/systemd/system/caddy.service
sudo chmod 644 /etc/systemd/system/caddy.service
sudo systemctl daemon-reload
sudo systemctl start caddy.service
```
* 设置Caddy自启动
```bash
sudo systenctl enable caddy.service
```
> 至此，Caddy的安装以及配置完成，只需要配置自己的Caddyfile即可。
* 我的配置文件，以此为例修改为自己的即可
```Caddyfile
${your_domain} {
    log /var/log/caddy/access.log
    gzip
    root /var/www/${your_site}/public
    git github.com/${your_github_name}/${your_site}.git {
        path /var/www/${your_site}
        then hugo --destination=/var/www/${your_site}/public
        hook /webhook 87e1129469efab1ea287afcc418e0679
        hook_type github
        clone_args --recursive
        pull_args --recurse-submodules
    }
    hugo
}
```
