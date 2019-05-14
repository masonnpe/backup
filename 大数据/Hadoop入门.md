## 安装Hadoop

```
cd /opt
mkdir software
mkdir module
tar -zxf hadoop-2.7.2.tar.gz -C /opt/module/
echo $JAVA_HOME # 获得jdk路径
vi /opt/module/hadoop-2.7.2/etc/hadoop/hadoop-env.sh修改export JAVA_HOME
pwd获得hadoop安装路径/opt/module/hadoop-2.7.2
 vi /etc/profile  shitf+g到最下面 o修改
```

```
export HADOOP_HOME=/opt/module/hadoop-2.7.2
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

```
source /etc/profile
```

## 本地文件运行Hadoop 案例

```
mkdir input
cp etc/hadoop/*.xml input
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'
cat output/*
```

```
mkdir wcinput
cd wcinput
touch wc.input
vi wc.input
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount wcinput wcoutput
cat wcoutput/part-r-00000
```

## 启动HDFS并运行MapReduce 程序

core-site.xml

```
<property>
	<name>fs.defaultFS</name>
    <value>hdfs://192.168.10.51:9000</value>
</property>
<property>
	<name>hadoop.tmp.dir</name>
	<value>/opt/module/hadoop-2.7.2/data/tmp</value>
</property>
```

hdfs-site.xml

```
<property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>
```

启动集群

```
bin/hdfs namenode -format
sbin/hadoop-daemon.sh start namenode
sbin/hadoop-daemon.sh start datanode
```

访问 http://192.168.10.51:50070/dfshealth.html#tab-overview

```
bin/hdfs dfs -mkdir -p /user/root/input
bin/hdfs dfs -put wcinput/wc.input  /user/root/input/
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /user/root/input/ /user/root/output
```

### Hbase

- [《简明 HBase 入门教程（开篇）》](http://www.thebigdata.cn/HBase/35831.html)
- [《深入学习HBase架构原理》](https://www.cnblogs.com/qiaoyihang/p/6246424.html)
- [《Hbase与传统数据库的区别》](https://blog.csdn.net/lifuxiangcaohui/article/details/39891099)
  - 空数据不存储，节省空间，且适用于并发。
- [《HBase Rowkey设计》](https://blog.csdn.net/u014091123/article/details/73163088)
  - rowkey 按照字典顺序排列，便于批量扫描。
  - 通过散列可以避免热点。





字典表压缩数据求交集