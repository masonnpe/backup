---
title: centos常用软件安装
categories: Linux
tags: [Linux]

---

## 配置IP

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

```
BOOTPROTO="static"
IPADDR="192.168.10.110"
NETMASK="255.255.255.0"
GATEWAY="192.168.10.1"
DNS1="8.8.8.8"
```

```
service network restart
vi /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=hadoop1
vi /etc/hosts
192.168.10.110 hadoop1
192.168.10.111 hadoop2
192.168.10.112 hadoop3
192.168.10.113 hadoop4
192.168.10.114 hadoop5
192.168.10.115 hadoop6
```

<!--more-->

## 安装JDK

```
tar -zxvf jdk-8u181-linux-x64.tar.gz
mv jdk1.8.0_181 jdk
vi /etc/profile
```

```shell
export JAVA_HOME=/usr/local/jdk
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tool.jar
```

```
source /etc/profile
```

## 安装Perl

```
yum install -y gcc
tar -zxvf perl-5.16.1.tar.gz
mv perl-5.16.1 perl
./Configure -des -Dprefix=/usr/local/perl
make && make install
```

## 配置ssh免密码通信

**各自主机**

```
ssh-keygen -t rsa
cd /root/.ssh
cp id_rsa.pub authorized_keys
```

**IP:192.168.10.51的主机** 

```
ssh-copy-id -i 192.168.10.52
```

**IP:192.168.10.52的主机** 

```
ssh-copy-id -i 192.168.10.51
```

## 安装Nginx

```
mkdir -p /usr/servers/distribution_nginx
cd /usr/servers/distribution_nginx
yum install -y readline-devel pcre-devel openssl-devel gcc
tar -xzvf ngx_openresty-1.7.7.2.tar.gz
cd ngx_openresty-1.7.7.2
cd bundle/LuaJIT-2.1-20150120
make clean && make && make install
ln -sf luajit-2.1.0-alpha /usr/local/bin/luajit
cd ..
wget https://github.com/FRiCKLE/ngx_cache_purge/archive/2.3.tar.gz
tar -xvf 2.3.tar.gz
wget https://github.com/yaoweibin/nginx_upstream_check_module/archive/v0.3.0.tar.gz
tar -xvf v0.3.0.tar.gz
cd ..
./configure --prefix=/usr/servers/distribution_nginx --with-http_realip_module  --with-pcre  --with-luajit --add-module=./bundle/ngx_cache_purge-2.3/ --add-module=./bundle/nginx_upstream_check_module-0.3.0/ -j2
make && make install
cd ..
cd nginx/sbin/
./nginx
cd ..
cd conf
vi nginx.conf
```

http{}内添加

```
    lua_package_path "/usr/servers/distribution_nginx/lualib/?.lua;;";
    lua_package_cpath "/usr/servers/distribution_nginx/lualib/?.so;;";
    include lua.conf;
```

/usr/servers/distribution_nginx/nginx/conf  路径下 vi lua.conf

```
server {
    listen       80;
    server_name  _;  
    location /lua {
        default_type 'text/html';
        content_by_lua 'ngx.say("hello world")';
    }   
}
```

/usr/servers/distribution_nginx/nginx/sbin目录下 

```
./nginx -s reload
```

iptables -I INPUT -p tcp --dport 80 -j ACCEPT  开放端口

访问 http://192.168.10.51/lua

## 安装tcl

```
tar -xzvf tcl8.6.1-src.tar.gz
cd  unix
./configure
make && make install
```

## 安装Redis

```
tar -zxvf redis-3.2.12.tar.gz
cd redis-3.2.12
make && make install
cd utils
cp redis_init_script /etc/init.d
cp redis.conf /etc/redis
mv redis.conf 6379.conf
vi 6379.conf
mv redis_init_script redis_6379
```

```
daemonize yes
bind 192.168.10.51
dir /var/redis/6379
```

```
chmod 777 redis_6379
./redis_6379 start
iptables -I INPUT -p tcp --dport 6379 -j ACCEPT
ps -ef|grep redis
vi redis_6379  
```

开机启动

```shell
# chkconfig:   2345 90 10
# description:  Redis is a persistent key-value database
chkconfig redis_6379 on
```

 slave节点的conf

```
slaveof 192.168.10.51 6379
```

启动

```
redis-cli 192.168.10.51
info replication
```

停止firewall   并     禁止firewall开机启动

```
systemctl stop firewalld.service
systemctl disable firewalld.service
```

## 安装MySQL

```
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum install mysql-server
chkconfig mysqld on
service mysqld start
mysql -u root
set password for root@localhost=password('root');
mysql -uroot -proot
```

## vsftpd

```
yum -y install vsftpd
vi /etc/vsftpd/chroot_list 
添加ftpuser，保存关闭
vi /etc/selinux/config
修改为SELINUX=disabled，保存关闭
// 关闭防火墙
systemctl stop firewalld.service
systemctl disable firewalld.service

// 创建目录
cd /
cd /ftpfile
// 创建一个没登陆权限的用户
useradd ftpuser -d /ftpfile/ -s /sbin/nologin
//设置权限
chown -R ftpuser.ftpuser /ftpfile/
// 设置密码
passwd ftpuser

vi /etc/vsftpd/vsftpd.conf

新增
local_root=/ftpfile
anon_root=/ftpfile
user_localtime=yes
pasv_min_port=61001
pasv_max_port=62000
修改
anonymous_enable=yes // 打开匿名访问
打开chroot_local_user=YES和chroot_list_file=/etc/vsftpd/chroot_list的注释
```



## nginx安装

```
yum install -y gcc
yum -y install pcre-devel
yum install -y zlib-devel
tar -zxvf nginx-1.14.0.tar.gz 
cd nginx-1.14.0
./configure
make
make install
```



平滑重启 kill -HUP pid

tail -f file 用于监视File文件增长

