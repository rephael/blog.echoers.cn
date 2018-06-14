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
### 