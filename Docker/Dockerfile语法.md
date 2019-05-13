---
title: Dockerfile语法
categories: Docker
tags: [Docker]
---

```dockerfile
FROM scratch # 制作baseimage
FROM centos # 使用baseimage
LABEL version="1.0" # 定义metadata
RUN set -ex; # 执行命令并创建新的imagelayer
WORKDIR demo # 创建目录并进入，默认根目录/test
ADD hello / # 将本地文件添加到里面
ADD test.tar.gz / # 添加到根目录并解压
COPY docker-entrypoint.sh /usr/local/bin/ # 添加文件，但不能解压
ENV GOSU_VERSION 1.7 # 设置常量  使用$GOSU_VERSION引用常量
VOLUME /var/lib/mysql # 存储
EXPOSE 3306 33060 # 网络
CMD ["mysqld"] # 设置容器启动后默认执行的命令和参数
ENTRYPOINT ["docker-entrypoint.sh"] # 设置容器启动时运行的命令
```

[docker-library](https://github.com/docker-library/)里有官方收录的Dockerfile，可以作为参考

```
docker build -t imagename path # 构建image
docker run imagename # 运行
```

<!--more-->