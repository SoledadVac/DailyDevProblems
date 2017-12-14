#机器配置
host配置:
```

10.0.1.101 bj-esbp-mid1.w-oasis.com
10.0.1.102 bj-esbp-mid2.w-oasis.com
10.0.1.103 bj-esbp-mid3.w-oasis.com

```
各个机器部署情况：
Name | IP | 部署程序 | 运行进程
- | :-: | :-: | :-:
bj-esbp-mid1.w-oasis.com | 10.0.1.101|  
bj-esbp-mid2.w-oasis.com | 10.0.1.102|  
bj-esbp-mid3.w-oasis.com | 10.0.1.103|  


#先决环境配置
## 资源限制配置
首先执行`vim /etc/profile`,加入：`ulimit -n 10240`,执行`source /etc/profile `;

之后修改文件：`/etc/security/limits.conf`，配置打开文件数目以及用户打开进程数目
1. ulimit-修改打开文件数目
加入：
```
* soft nofile 10240
* hard nofile 10240
```
加入之后使用查看是否生效（默认为1024）：
```
[root@bj-esbp-mid1 security]# ulimit -n
10240
```
2. nproc-修改打开进程数目
加入：
```
* soft noproc 10240
* hard noproc 10240
```
加入之后使用命令`ulimit -u`查看是否生效;

## JDK
版本：
```
[root@bj-esbp-mid3 bin]# java -version
java version "1.7.0_25"
Java(TM) SE Runtime Environment (build 1.7.0_25-b15)
Java HotSpot(TM) 64-Bit Server VM (build 23.25-b01, mixed mode)
```
[安装说明](https://www.cnblogs.com/xing901022/p/5775398.html)

# Zookeeper集群

Name | IP | dataDir
- | :-: | - |
bj-esbp-mid1.w-oasis.com | 10.0.1.101| /data/woasis/zookeeper-3.4.11/data
bj-esbp-mid2.w-oasis.com | 10.0.1.102| /data/woasis/zookeeper-3.4.11/data
bj-esbp-mid3.w-oasis.com | 10.0.1.103| /data/woasis/zookeeper-3.4.11/data

server：

```
server.1=bj-esbp-mid1.w-oasis.com:2888:3888
server.2=bj-esbp-mid2.w-oasis.com:2888:3888
server.3=bj-esbp-mid3.w-oasis.com:2888:3888
```
之后写入每个server的id:
在每个`dataDir`下面执行：
```
echo "1" > myid
echo "2" > myid
echo "3" > myid
```
启动停止：
在bin下执行：
```
zkServer.sh start  #启动Zookeeper服务，正确启动只有，使用jps命令会看到QuorumPeerMain，如果该进程启动说明Zookeeper服务成功的启动
zkServer.sh status #查看Zookeeper服务的状态，你会看到哪个节点是Leader节点，哪个节点是Follower节点，并且只有一个Zookeeper节点
zkServer.sh stop  #停止Zookeeper服务，每个节点都要停止
```
## 集群之间无密码登陆
```
# ssh-keygen -t rsa
# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
# chmod 700 ~/.ssh && chmod 600 ~/.ssh/*
```
之后复制到其他机器上：
```
scp authorized_keys bj-esbp-mid2.w-oasis.com:/root/.ssh
scp authorized_keys bj-esbp-mid3.w-oasis.com:/root/.ssh
```

# 修改hadoop配置
1. 进入目录`/data/woasis/hadoop-2.6.5/etc/hadoop`:
修改`core-site.xml`文件：
```
<configuration>
<property>
    <name>hadoop.tmp.dir</name>
    <value>/data/woasis/hadoop-2.6.5/tmp</value>
</property>
<property>
   <name>fs.default.name</name>
   <value>hdfs://10.0.1.101:9000</value>
</property>
 <property>
     <name>dfs.datanode.data.dir</name>
     <value>/data/woasis/hadoop-2.6.5/hadoop/data</value>
  </property>
</configuration>
```
2. 修改`hadoop-env.sh`文件，添加jdk路径;
3. 创建hadoop的数据和用户目录:
```
[root@bj-esbp-mid1 hadoop-2.6.5]# pwd
/data/woasis/hadoop-2.6.5
[root@bj-esbp-mid1 hadoop-2.6.5]# mkdir -p /hadoop/name
[root@bj-esbp-mid1 hadoop-2.6.5]# mkdir -p /hadoop/data
```
4. 修改`hdfs-site.xml`文件：
```
<configuration>
 <property>
   <name>dfs.namenode.name.dir</name>
   <value>/data/woasis/hadoop-2.6.5/hadoop/name</value>
 </property>
 <property>
    <name>dfs.datanode.data.dir</name>
    <value>/data/woasis/hadoop-2.6.5/hadoop/data</value>
 </property>
 <property>
     <name>dfs.replication</name>
     <value>3</value>
 </property>
  <property>
       <name>dfs.namenode.rpc-address</name>
       <value>10.0.1.101:9001</value>
   </property>
</configuration>
```
5. mapred-site.xml
```
mv mapred-site.xml.template mapred-site.xml
```

添加：
```
<configuration>
  <property>
     <name>mapred.job.tracker</name>
     <value>bj-esbp-mid1.w-oasis.com:9001</value>
  </property>
</configuration>
```
6. 修改slave文件&masters文件
```
  [root@bj-esbp-mid1 hadoop]# vim slaves
```
改为：
```
  bj-esbp-mid2.w-oasis.com
  bj-esbp-mid3.w-oasis.com
```
同样，在masters文件中填入master的地址。
同步文件：
```
scp -r hadoop-2.6.5 bj-esbp-mid2.w-oasis.com:/data/woasis
scp -r hadoop-2.6.5 bj-esbp-mid3.w-oasis.com:/data/woasis
```

# 启动Hadoop集群
进入到Hadoop的bin目录下：
```
 ./hadoop namenode -format

```
格式化namenode，第一次启动服务前执行的操作，以后不需要执行。

然后启动hadoop：
```
sbin/start-all.sh
```


通过jps命令能看到除jps外有3个进程：


30613 NameNode
30807 SecondaryNameNode
887 Jps
30972 ResourceManager



#HBase配置

## 修改 JAVA_HOME

解压安装包,修改`/data/woasis/hbase-1.2.6/conf`下的`hbase-env.sh`，添加：`exportJAVA_HOME=/data/jdk1.7.0_25`;

## 修改 hbase-site.xml
