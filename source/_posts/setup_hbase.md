---
title: 在Linux上搭建HBase集群环境
Status: public
url: setup_hbase
tags: HBase
date: 2013-02-26
---

本文是在安装完成Hadoop的基础之上进行的，Hadoop的安装戳[这里](/post/hadoop_setup)。
本文采用的Hadoop版本为0.20.2，HBase版本为0.90.6，ZooKeeper的版本为3.3.2（stable版）。
本文仍然采用了Hadoop安装的环境，机器如下：

<table>
<tr>
    <td>机器名</td>
    <td>IP地址</td>
    <td>用途</td>
    <td>Hadoop模块</td>
    <td>HBase模块</td>
    <td>ZooKeeper模块</td>
</tr>
<tr>
    <td>server206</td>
    <td>192.168.20.6</td>
    <td>Master</td>
    <td>NameNode、JobTracker、SecondaryNameNode</td>
    <td>HMaster</td>
    <td>QuorumPeerMain</td>
</tr>
<tr>
    <td>ap1</td>
    <td>192.168.20.36</td>
    <td>Slave</td>
    <td>DataNode、TaskTracker</td>
    <td>HRegionServer</td>
    <td>QuorumPeerMain</td>
</tr>
<tr>
    <td>ap2</td>
    <td>192.168.20.38</td>
    <td>Slave</td>
    <td>DataNode、TaskTracker</td>
    <td>HRegionServer</td>
    <td>QuorumPeerMain</td>
</tr>
</table>

# 安装ZooKeeper
由于HBase默认集成了ZooKeeper，可以不用单独安装ZooKeeper。本文采用独立安装ZooKeeper的方式。
1\. 将zookeeper解压到/home/hadoop目录下。
2\. 将/home/hadoop/zookeeper-3.3.2/conf目录下的zoo_sample.cfg文件拷贝一份，命名为为“zoo.cfg”。
3\. 修改zoo.cfg文件，修改后内容如下：
```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
dataDir=/home/hadoop/zookeeper-3.3.2/zookeeper_data
# the port at which the clients will connect
clientPort=2181
dataLogDir=/home/hadoop/zookeeper-3.3.2/logs
server.1=192.168.20.6:2888:3888
server.2=192.168.20.36:2888:3888
server.3=192.168.20.38:2888:3888
```
其中，2888端口号是zookeeper服务之间通信的端口，而3888是zookeeper与其他应用程序通信的端口。
这里修改了dataDir和dataLogDir的值。
需要特别注意的是：如果要修改dataDir的值不能将原来的行在前面加个“#”注释掉后在后面再增加一行，这样是不起作用的。可以参考bin目录下的zkServer.sh文件中的72行，内容如下：
```
ZOOPIDFILE=$(grep dataDir "$ZOOCFG" | sed -e 's/.*=//')/zookeeper_server.pid
```
这里通过grep和sed命令来获取dataDir的值，对于行前面添加“#”注释是不起作用的。
4\. 创建zoo.cfg文件中的dataDir和dataLogDir所指定的目录。
5\. 在dataDir目录下创建文件`myid`。
6\. 通过scp命令将zookeeper-3.3.2目录拷贝到其他节点机上。
7\. 修改`myid`文件，在对应的IP的机器上输入对应的编号，该编号要和zoo.cfg中的一致。本例中在192.168.20.6上文件内容为1；在192.168.20.36上文件内容为2；在192.168.20.38上文件内容为3。至此ＺooＫeeper的安装已经完成。

# 运行ZooKeeper
1\. 在节点机上依次执行`/home/hadoop/zookeeper-3.3.2/bin/zkServer.sh start`脚本。运行第一个ZooKeeper的时候会因等待其他节点而出现刷屏现象，等启动起第二个节点上的ZooKeeper后就正常了。运行完成之后该脚本会出现刷屏现象，我这里没有理会。
2\. 通过jps命令来查看各节点机上是否含有QuorumPeerMain进程。
3\. 通过`/home/hadoop/zookeeper-3.3.2/bin/zkServer.sh status`命令来查看状态。本例中有三个节点机，其中必有一个leader，两个follower存在。
4\. 在各节点机上依次执行`/home/hadoop/zookeeper-3.3.2/bin/zkServer.sh stop`来停止ZooKeeper服务。

# 配置时间同步ntp服务
HBase在运行的时候各个节点之间时间不同步会存在莫名其妙的问题，这里选择以192.168.20.36机器作为时间同步服务器，其他机器从该机器同步时间。
在192.168.20.36上通过service ntpd start命令来启动ntp服务。
在其他机器上配置crontab命令，增加下面一行：
```
0 */1 * * * /usr/sbin/ntpdate 192.168.20.36 && /sbin/hwclock -w
```
这里采用一个小时同步一次时间的方式。

# 安装HBase
1\. 在HMaster机器上将HBase解压到/home/hadoop目录下。
2\. 修改配置文件hbase-env.sh，使`export HBASE_MANAGES_ZK=false`。如果想要HBase使用自带的ZooKeeper则使用设置为true。使`export JAVA_HOME=/usr/java/jdk1.6.0_10`来指定java的安装路径。
3\. 修改配置文件hbase-site.xml如下：
```
<configuration>
  <property>  
    <name>hbase.rootdir</name>  
    <value>hdfs://server206:9000/hbase</value>  
  </property>
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
</property>
<property>
    <name>hbase.master.port</name>
    <value>60000</value>
</property>
<property>
    <name>hbase.zookeeper.property.clientPort</name>
    <value>2181</value>    
</property>
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>server206,ap1,ap2</value>
</property>
<property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/home/hadoop/zookeeper-3.3.2/zookeeper_data</value>    
</property>
<property>
    <name>dfs.support.append</name>
    <value>true</value>
</property>
<property>
    <name>hbase.regionserver.handler.count</name>
    <value>100</value>
</property> 
</configuration>
```
4\. 修改regionservers，添加HRegionServer模块所运行机器的主机名。在本例中内容如下：
```
ap1
ap2
```
5\. 为了确保HBase和Hadoop的兼容性，这里将/home/hadoop/hadoop-0.20.2/hadoop-0.20.2-core.jar文件复制到/home/hadoop/hbase-0.90.6/lib目录下，并将原先的Hadoop的jar文件删掉或重命名为其他后缀的文件。
6\. 将hbase-0.90.6文件夹通过scp命令复制到其他节点机上。

# HBase的启动
1. 在Hadoop的NameNode所在的机器上使用start-all.sh脚本来启动Hadoop集群。
2. 在各个节点机上调用zkServer.sh脚本来启动ZooKeeper。
3. 在HMaster所在的机器上使用start-hbase.sh脚本来启动HBase集群。

HBase启动后会在HDFS自动创建/hbase的文件夹，可以通过`hadoop fs -ls /hbase`命令来查看，该目录不需要自动创建。如果在安装HBase的过程中失败需要重新启动，最好将此目录从集群中删除，通过命令`hadoop fs -rmr /hbase`来删除。

需要特别注意的是在hadoop集群中`hadoop fs -ls /hbase`目录和`hadoop fs -ls hbase`目录并非一个目录，通过`hadoop fs -ls hbase`查看到的目录实际上为/user/hadoop/hbase目录。

# HBase管理界面
Master的界面：http://192.168.20.6:60010/master.jsp
RegionServer的界面：http://192.168.20.36:60030/regionserver.jsp和http://192.168.20.38:60030/regionserver.jsp

# 参考资料
* [http://blog.csdn.net/gudaoqianfu/article/details/7327191](http://blog.csdn.net/gudaoqianfu/article/details/7327191)
* [http://thomas0988.iteye.com/blog/1309867](http://thomas0988.iteye.com/blog/1309867)

# 下载链接
[http://pan.baidu.com/share/link?shareid=1235031445&uk=3506813023](http://pan.baidu.com/share/link?shareid=1235031445&uk=3506813023) 提取码：v8ok