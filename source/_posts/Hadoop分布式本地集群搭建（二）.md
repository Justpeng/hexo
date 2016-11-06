---
title: Hadoop分布式集群搭建（二）
date: 2016-10-22 17:58:27
description: "Hadoop分布式集群搭建"
categories: [Hadoop]
tags: [Hadoop]
top: 14
---
# Hadoop
## 基础环境
- centOS 64(已配置ssh,jdk,hosts,防火墙关闭) 
- hadoop 1.2.1 (推荐)

所有主机都需要配置，其中个别端口号和地址需要按照主机进行修改，已知/etc/hosts配置

主机 | ip
---|---|
master | 192.168.1.122
slave1 |192.168.1.123
slave2 |192.168.1.124
slave3 |192.168.1.125

## 安装hadoop
在目录/home/hadoop/ 下，这里统一使用hadoop用户安装，在安装过程中必须保证使用hadoop用户安装，否则在后面可能会出现无权限的情况，增加工作量，当然，若都使用root用户操作也可。
```
 tar -zxvf hadoop-1.2.1.tar.gz
```
- 环境变量配置(/etc/profile)
```
export HADOOP_HOME=/home/hadoop/hadoop-1.2.1
export PATH=$PATH:$HADOOP_HOME/bin
```
source /etc/profile 重启加载生效


## hadoop配置文件

### 修改hadoop配置文件 hadoop-env.sh 

```
[hadoop@master conf]$ vim hadoop-env.sh

```
添加jdk目录
```
export JAVA_HOME=/opt/jdk1.7.0_67
```

### 配置core-site.xml

```
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://master:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/hadoop-data/tmp</value>
    </property>
</configuration>
```
属性：
- fs.default.name：是NameNode的URI：hdfs://主机名:端口
- hadoop.tmp.dir ：目录可自己创建，Hadoop的默认临时路径，如进程文件存储，这个最好配置，如果在新增节点或者其他情况下莫名其妙的DataNode启动不了，就删除此文件中的tmp目录即可。不过如果删除了NameNode机器的此目录，那么就需要重新执行NameNode格式化的命令。

注意：此处的/home/hadoop/hadoop-data/tmp 可先进行创建

- 配置 hdfs-site.xml

```
<configuration>
<property>
    <name>dfs.replication</name>
    <value>3</value>
</property>
<property>
    <name>dfs.name.dir</name>
    <value>/home/hadoop/hadoop-data/hdfs/name</value>
</property>
<property>
    <name>dfs.data.dir</name>
    <value>/home/hadoop/hadoop-data/hdfs/data</value>
</property>
<property>
    <name>dfs.permissions</name>
    <value>false</value>
</property>
</configuration>
                
```
属性：
- dfs.replication ：几个从节点，有slave1,slave2,slave3三个
- dfs.name.dir: NameNode元数据存储位置，持久存储名字空间及事务日志的本地文件系统路径。 当这个值是一个逗号分割的目录列表时，nametable数据将会被复制到所有目录中做冗余备份
- dfs.data.dir: DataNode元数据存储本地文件系统路径，存储所有用户数据，逗号分割的列表。 当这个值是逗号分割的目录列表时，数据将被存储在所有目录下，通常分布在不同设备上
- dfs.permissions: 为false不考虑权限

注意：此处的/home/hadoop/hadoop-data/hdfs/name 与 /home/hadoop/hadoop-data/hdfs/data目录不能预先创建，hadoop格式化时会自动创建


### 配置 mapred-site.xml

```
<configuration>
   <property>
         <name>mapred.job.tracker</name>
        <!--<value>192.168.1.122:9001</value>-->
        <value>master:9001</value>
   </property>
    <property>
        <name>mapred.local.dir</name>
        <value>/home/hadoop/hadoop-data/tmp</value>
    </property>
    <property>
        <name>mapred.tasktracker.map.tasks.maximum</name>
        <value>3</value>
    </property>
    <property>
        <name>mapred.tasktracker.reduce.tasks.maximum</name>
        <value>3</value>
    </property>
</configuration>

```
属性：
- mapred.job.tracker 为jobtracker 的主机名和端口
- mapred.tasktracker.map.tasks.maximum 为最大map的任务个数
- mapred.tasktracker.reduce.tasks.maximum 为最大reduce任务的个数

### 配置 slaves、masters文件

使用ip或者主机名，做好使用主机名
- slaves
```
slave1
slave2
slave3

```
- masters

```
master
```
注意: 
- 其他三台主机也需要进行jdk等基础环境的配置
- 从机中的hadoop配置是相同的，如 core-site.xml的配置是相同的
```
scp -r  hadoop-1.2.1 hadoop@slave1:/home/hadoop/
scp -r  hadoop-1.2.1 hadoop@slave2:/home/hadoop/
scp -r  hadoop-1.2.1 hadoop@slave3:/home/hadoop/

```

## 格式化文件系统
在 /安装目录/bin目录下

```
[hadoop@master bin]$ hadoop namenode -format
```
可在之前配置的数据目录中，看到生成的文件夹
```
[hadoop@master ~]$ cd /home/hadoop/hadoop-data/
[hadoop@master hadoop-data]$ ls
hdfs  tmp
[hadoop@master hadoop-data]$ cd hdfs/
[hadoop@master hdfs]$ ls
name

```
## 启动Hadoop
终于到了启动的时刻了！

```
[hadoop@master hadoop-1.2.1]$ bin/start-all.sh

```
输出：

```
Warning: $HADOOP_HOME is deprecated.

starting namenode, logging to /home/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-hadoop-namenode-master.out
slave3: starting datanode, logging to /home/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-hadoop-datanode-slave3.out
slave2: starting datanode, logging to /home/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-hadoop-datanode-slave2.out
slave1: starting datanode, logging to /home/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-hadoop-datanode-slave1.out
The authenticity of host 'master (192.168.1.122)' can't be established.
ECDSA key fingerprint is 3b:14:33:d5:5a:c4:92:c5:62:65:49:3d:4f:90:07:16.
Are you sure you want to continue connecting (yes/no)? yes
master: Warning: Permanently added 'master,192.168.1.122' (ECDSA) to the list of known hosts.
master: starting secondarynamenode, logging to /home/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-hadoop-secondarynamenode-master.out
starting jobtracker, logging to /home/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-hadoop-jobtracker-master.out
slave1: starting tasktracker, logging to /home/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-hadoop-tasktracker-slave1.out
slave3: starting tasktracker, logging to /home/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-hadoop-tasktracker-slave3.out
slave2: starting tasktracker, logging to /home/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-hadoop-tasktracker-slave2.out

```
查看是不是配置成功了!!!

- [x] - 在master中，显示四个进程!
```
[hadoop@master ~]$ jps
15575 SecondaryNameNode
15723 JobTracker
15924 Jps
15361 NameNode

```
- [x] 在三个从节点中显示三个进程!

```
[hadoop@slave1 ~]$ jps
10195 TaskTracker
10549 Jps
9935 DataNode

```

心里感觉很舒畅，但是还没有完，要验证是否每个datanode都是正常与master链接，如果防火墙不关闭的话，会有问题的！

- http://master:50070
如下图所示，可以看到 容量、LiveNodes=3

![image](http://og6e19zce.bkt.clouddn.com/hadoop_done.png)

- http://master:50030

![image](http://og6e19zce.bkt.clouddn.com/hadoop_node.jpg)

以上两个地址正常出现，则表示集群正式搭建成功了！ Congratulation!


此外，如果从机节点不是全部启用的话，可以单独在从机节点，启动hdfs和mpared
```
start-dfs.sh
start-mapred.sh
```






### 关闭Hadoop

```
[hadoop@master ~]$ hadoop-1.2.1/bin/stop-all.sh
```

### END
接下来，就是对集群的使用了 ~ 