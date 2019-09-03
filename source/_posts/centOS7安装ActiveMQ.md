---
title: centOS7安装ActiveMQ
comments: false
date: 2019-08-31 10:35:22
categories: Active
tags:
 - Active
 - centOS7
---

### 简介
ActiveMQ版本：5
虚拟机：virtualBox
Linux版本：centOS7
备注：因为ActiveMQ是java语言编写的，所以在安装ActiveMQ之前，必须在linux上安装好 jdk，
注意：不同环境，不同版本，在安装使用上可能会有不同的遭遇，仅以此篇文章作为参考，如有错误，欢迎指正。

### 1. 官网下载
>下载地址：[https://activemq.apache.org/components/classic/download/#]
>![image.png](https://upload-images.jianshu.io/upload_images/18660770-c2d8f9db87e6185d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2. 安装
> 1. 将下载下来的压缩包 apache-activemq-5.15.9-bin.tar.gz 复制到 Linux 虚拟机上 /opt 目录下。
> 2. 解压 ：tar -xzvf apache-activemq-5.15.9-bin.tar.gz 

### 3.启动、关闭
> 进入解压目录 /opt/apache-activemq-5.15.9/bin
> 启动/关闭：./activemq start/stop

### 4.查看后台进程，检验是否启动成功
三种方式：
> 1. 命令：ps -ef|grep activemq|grep -v grep
> ![image.png](https://upload-images.jianshu.io/upload_images/18660770-240ab9f81399246b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 2. 命令：lsof -i:61616
> ![image.png](https://upload-images.jianshu.io/upload_images/18660770-e3c9d20b069743d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 3. 命令：netstat -anp|grep 61616
> ![image.png](https://upload-images.jianshu.io/upload_images/18660770-0b7a10867399a51c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 5. 访问控制台
> 1. 首先需要切换虚拟机的连接网卡为“桥接网卡”
> 2. 关闭 linux 防火墙：service iptables stop
> 3. 在浏览器访问：http://虚拟机IP地址:8161
> 4. 默认用户名/密码：admin/admin
> ![image.png](https://upload-images.jianshu.io/upload_images/18660770-34bab5e67e68e52e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

