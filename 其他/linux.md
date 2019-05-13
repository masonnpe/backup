## [Linux命令大全](http://man.linuxde.net/)

查询登录当前系统的用户信息：w

输出系统日志最后10行：dmesg | tail

内存分析命令：free m

tail  -10 查看文件的后10行  tail -f可以对某个文件进行动态监控，例如tomcat的日志文件

find /home -name "*.txt"  查找在/home目录下查找以.txt结尾的文件名

mv 可以重命名文件和目录  也可以剪切文件的目录

cp -r  递归拷贝

touch abc.txt 创建文件

more 可以显示百分比，回车可以向下一行；空格可以向下一页，q可以退出查看

less 可以使用键盘上的PgUp和PgDn翻页，q结束查看

tar -zcvf 打包压缩后的文件名 要打包压缩的文件

tar -xvf xxx.tar.gz 解压到当前目录  可以通过-C解压到指定目录下可以

ps -ef / ps aux 查看进程

kill -9 进程的pid  强制终止进程

grep  查找文件里的字符串  grep "moo" target*

awk 对文件内容做统计

**top命令**

第二行Tasks：

total 进程总数；running 正在运行的进程数；sleeping 睡眠的进程数；stopped 停止的进程数；zombie 僵尸进程数
%Cpu(s): us 用户空间占用CPU百分比；sy 内核空间占用CPU百分比；ni 用户进程空间内改变过优先级的进程占用CPU百分比；id 空闲CPU百分比；wa 等待输入输出的CPU时间百分比；hi：硬件CPU中断占用百分比；si：软中断占用百分比；st：虚拟机占用百分比
KiB Mem：total 物理内存总量；free 空闲内存总量；used 使用的物理内存总量 ；buff/cache 用作内核缓存的内存量
KiB Swap：total 交换区总量；free 空闲交换区总量；sed 使用的交换区总量；avail Mem缓冲的交换区总量,内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖

PID     进程id
USER    进程所有者的用户名
PR      优先级
NI      nice值。负值表示高优先级，正值表示低优先级
VIRT    进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
RES     进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
SHR     共享内存大小，单位kb
S       进程状态(D=不可中断的睡眠状态,R=运行,S=睡眠,T=跟踪/停止,Z=僵尸进程)
%CPU    上次更新到现在的CPU时间占用百分比
%MEM    进程使用的物理内存百分比

yum install sysstat

**iostat**

Iostat提供三个报告：CPU利用率、设备利用率和网络文件系统利用率，使用-c，-d和-h参数可以分别独立显示这三个报告。

tps：该设备每秒的传输次数（Indicate the number of transfers per second that were issued to the device.）。"一次传输"意思是"一次I/O请求"。多个逻辑请求可能会被合并为"一次I/O请求"。"一次传输"请求的大小是未知的。
kB_read/s：每秒从设备（drive expressed）读取的数据量；
kB_wrtn/s：每秒向设备（drive expressed）写入的数据量；
kB_read：读取的总数据量；
kB_wrtn：写入的总数量数据量；这些单位都为Kilobytes。

**mpstat**

查看CPU的占用情况

%user      在internal时间段里，用户态的CPU时间(%)，不包含nice值为负进程  (usr/total)*100
%nice      在internal时间段里，nice值为负进程的CPU时间(%)   (nice/total)*100
%sys       在internal时间段里，内核时间(%)       (system/total)*100
%iowait    在internal时间段里，硬盘IO等待时间(%) (iowait/total)*100
%irq         在internal时间段里，硬中断时间(%)     (irq/total)*100
%soft       在internal时间段里，软中断时间(%)     (softirq/total)*100
%idle       在internal时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间(%) (idle/total)*100

**Linux用户管理相关命令:**

- `useradd 选项 用户名`:添加用户账号
- `userdel 选项 用户名`:删除用户帐号
- `passwd 用户名`:更改或创建用户的密码
- `passwd -d 用户名`:  清除用户密码

**Linux的权限命令**



## 安装软件

配置yum

```
yum clean all
yum makecache
yum install wget
```

gcc

```
yum install -y gcc
```

yum -y install libaio-devel 

yum -y install perl perl-devel 

安装jdk zookeeper

```
vi /etc/profile
export JAVA_HOME=/usr/local/jdk1.8.0_181
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.4.13
export CLASSPATH=.:%JAVA_HOME%/lib/dt.jar:%JAVA_HOME%/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin:$ZOOKEEPER_HOME/bin
source /etc/profile

cd conf
cp zoo_sample.cfg zoo.cfg

dataDir=/usr/local/zookeeper-3.4.13/dataDir
dataLogDir=/usr/local/zookeeper-3.4.13/dataLogDir
mkdir dataDir
mkdir dataLogDir

cd bin
./zkServer.sh start
```

zookeeper基本数据模型

树形结构

打开命令行后台

```
./zkCli.sh
```

作用：

1. master节点选举 主节点挂了后，从节点接收工作且唯一，首脑模式，高可用集群
2. 统一配置文件管理，同步更新配置文件
3. 发布订阅
4. 提供分布式锁
5. 集群管理，保证数据的强一致性

命令：

1. ls子节点 ls2状态信息
2. stat /
3. get /
4. create  [-s 顺序节点[-e 临时节点 path data
5. set path data

session基本原理

1. 客户端服务器连接存在会话
2. 绘画可以设置超时时间
3. 心跳结束session过期
4. session过期，临时节点znode会被抛弃
5. 心跳机制

watcher机制

1. 每个节点的操作都会有一个监督者
2. znode发生变化出发watcher事件
3. 一次性，触发后销毁



redis

```
yum install -y gcc

cd  /usr/local/tcl8.6.1/unix/
./configure  
make && make install

cd redis
make MALLOC=libc
make test && make install

cd tests
cd inter...
vi ......tcl
修改参数为50000
```

（1）redis utils目录下，有个redis_init_script脚本
（2）将redis_init_script脚本拷贝到linux的/etc/init.d目录中，将redis_init_script重命名为redis_6379，6379是我们希望这个redis实例监听的端口号
（3）修改redis_6379脚本的第6行的REDISPORT，设置为相同的端口号（默认就是6379）
（4）创建两个目录：/etc/redis（存放redis的配置文件），/var/redis/6379（存放redis的持久化文件）
（5）修改redis配置文件（默认在根目录下，redis.conf），拷贝到/etc/redis目录中，修改名称为6379.conf

修改redis.conf中的部分配置为生产环境

```
daemonize	yes							让redis以daemon进程运行
pidfile		/var/run/redis_6379.pid 	设置redis的pid文件位置
port		6379						设置redis的监听端口号
dir 		/var/redis/6379				设置持久化文件的存储位置
```

启动redis

```
cd /etc/init.d
chmod 777 redis_6379
./redis_6379 start
```

确认redis进程是否启动

```
ps -ef | grep redis
```

让redis跟随系统启动自动启动,在redis_6379脚本中，最上面加入两行注释

```
# chkconfig:   2345 90 10
# description:  Redis is a persistent key-value database
 chkconfig redis_6379 on
```



安装perl

```
tar -xzf perl-5.16.1.tar.gz
cd perl-5.16.1
./Configure -des -Dprefix=/usr/local/perl
make && make test && make install
perl -v
```



安装docker  yum -y update

yum install -y docker

service docker start 

service docker stop 

service docker restart



docker search java

docker pull ...

docker images

导出镜像  docker save java > /home/java.tar.gz

删除 docker rmi ...

docker load < /home/java.tar.gz



暂停容器 docker pause myjava

unpause

stop

start -i





docker pull percona/percona-xtradb-cluster



修改镜像名字  docker tag old new 

创建内部网络  

docker network create net1

查看 inspect net1

删除 rm net1





创建docker卷  

docker volume create --name v1



docker run -d -p 3306:3306

-v  v1:/var/lib/mysql

-e MYSQL_ROOT_PASSWORD =a123

-e CLUSTER_NAME=PXC

-e XTRABACKUP_PASSWORD=a123

--provileged --name=node1 --net=net1 --ip 172.18.0.2  pxc





进入容器 docker exec -it h1 bash  



2、在每个CentOS中都安装Java和Perl

WinSCP，就是在windows宿主机和linux虚拟机之间互相传递文件的一个工具

（1）安装JDK

1、将jdk-7u60-linux-i586.rpm通过WinSCP上传到虚拟机中
2、安装JDK：rpm -ivh jdk-7u65-linux-i586.rpm
3、配置jdk相关的环境变量
vi ~/.bashrc
export JAVA_HOME=/usr/java/latest
export PATH=$PATH:$JAVA_HOME/bin
source ~/.bashrc
4、测试jdk安装是否成功：java -version


**集群搭建**  
vi /etc/hosts  
ssh-keygen -t rsa  
cp id_rsa.pub authorized_keys  
ssh-copy-id -i hostname  
ssh hostname  

安装数据库  
yum install mariadb-server -y  
systemctl start mariadb  
mysql_secure_installation   
mysql -u root -p  
grant all privileges on *.* to myname@localhost identified by 'pwd';  
grant all privileges on *.* to myname@'%' identified by 'pwd';  


