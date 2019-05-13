---
title: Docker及常用软件安装
categories: Docker
tags: [Docker]
---

```
yum remove docker \
           docker-client \
           docker-client-latest \
           docker-common \
           docker-latest \
           docker-latest-logrotate \
           docker-logrotate \
           docker-selinux \
           docker-engine-selinux \
           docker-engine
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce
systemctl start docker
docker version
```

## RabbitMQ

```
docker search rabbitmq:management
docker pull rabbitmq:management
docker run -d --name rabbitmq --publish 5671:5671  --publish 5672:5672 --publish 4369:4369 --publish 25672:25672 --publish 15671:15671 --publish 15672:15672 rabbitmq:management
```

```
http://192.168.10.51:15672/
guest guest
```

## Redis

```
docker pull redis
docker run -di --name=redis -p 6379:6379 redis
```

## zipkin

```
docker pull openzipkin/zipkin
docker run -d -p 9411:9411 openzipkin/zipkin
```

## zookeeper

```
docker pull zookeeper
docker run --privileged=true -d --name zookeeper --publish 2181:2181  -d zookeeper:latest
```

