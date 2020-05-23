---
title: 使用docker快速部署greenplum集群
#description: 
date: 2020-05-14 02:30:00
categories: 
 - Itools
tags:
 - greenplum
---

## 摘要
使用docker快速部署greenplum集群
![](/resources/images/20200514/dbeaver_greenplum.png)
<!-- more -->
# 使用docker安装greenplum
## 1、制作docker镜像
### 1.1 从官网下载对应平台的Greenplum安装包后，准备如下Dockerfile文件：
```dockerfile
FROM lyasper/gphost
COPY greenplum-db-5.*-rhel7-x86_64.rpm /home/gpadmin/greenplum-db.rpm
RUN rpm -i /home/gpadmin/greenplum-db.rpm
RUN chown -R gpadmin /usr/local/greenplum-db*
RUN rm -f /home/gpadmin/greenplum-db.rpm
```
### 1.2 运行如下命令创建镜像
```shell
docker build . -t mygreenplum
```
## 2、docker-compose配置集群
### 2.1 初始化集群
准备如下配置文件docker-compose.yaml
```yaml
version: '3'
services:
  mdw:
    hostname: mdw
    image: "mygreenplum"
    ports:
     - "5222:22"
     - "5432:5432"
  sdw1:
    hostname: sdw1
    image: "mygreenplum"
  sdw2:
    hostname: sdw2
    image: "mygreenplum"
  etl:
    hostname: etl
    image: "mygreenplum"
    ports:
     - "8000:8000"
```

### 2.2 启动容器
安装docker-compose
```shell
curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
启动docker容器
```shell
docker-compose up -d
```
## 3、初始化集群
### 3.1 登陆到mdw容器中
```shell
ssh -p 5222 gpadmin@127.0.0.1
```
初始密码为:changeme
### 3.2 准备集群配置文件
```shell
source /usr/local/greenplum-db/greenplum_path.sh
./artifact/prepare.sh -s 2 -n 2
```
- -s表示 segment机器（容器）的数量
- -n表示每个容器里primary segment的个数

### 3.3 初始化集群
```shell
source env.sh 
gpinitsystem -a -c gpinitsystem_config
```
使用docker快速部署greenplum集群
![](/resources/images/20200514/Databasesuccessfullystarted.png)
