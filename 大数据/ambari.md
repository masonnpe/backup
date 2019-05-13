https://blog.51cto.com/tryingstuff/2066561


yum install -y wget
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
所有节点  yum install java-1.8.0-openjdk -y
systemctl stop firewalld 关防火墙
systemctl disable firewalld.service 
配置免密登陆
各节点
vi /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=node1
vi /etc/hosts
ssh-keygen -t rsa


cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys
scp /root/.ssh/id_rsa.pub node2:/root/.ssh/authorized_keys
scp /root/.ssh/id_rsa.pub node3:/root/.ssh/authorized_keys


节点1 yum install httpd -y
安装包放到  /var/www/html
tar xf 解压所有

repo 文件放到/etc/yum.repo.d/

systemctl start httpd
yum clean all
 yum makecache fast

java_home :/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64/jre


yum update openssl
/etc/python/cert-verification.cfg  disable
/etc/ambari-agent/conf/ambari-agent.ini   force_https_protocol=PROTOCOL_TLSv1_2

sudo rpm -Uvh http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm