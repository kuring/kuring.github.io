---
title: 在Linux上搭建Hadoop集群环境
Status: public
url: hadoop_setup
tags: Hadoop
date: 2013-06-26
---


本文选择安装的hadoop版本为网上资料较多的0.20.2，对于不懂的新技术要持保守态度。遇到问题解决问题的痛苦远比体会用不着功能的新版本的快感来的更猛烈。

# 安装环境
本文选择了三台机器来搭建hadoop集群，1个Master和2个Slave。本文中的master主机即namenode所在的机器，slave即datanode所在的机器。节点的机器名和IP地址如下

<table>
<tr>
    <td>机器名</td>
    <td>IP地址</td>
    <td>用途</td>
    <td>运行模块</td>
</tr>
<tr>
    <td>server206</td>
    <td>192.168.20.6</td>
    <td>Master</td>
    <td>NameNode、JobTracker、SecondaryNameNode</td>
</tr>
<tr>
    <td>ap1</td>
    <td>192.168.20.36</td>
    <td>Slave</td>
    <td>DataNode、TaskTracker</td>
</tr>
<tr>
    <td>ap2</td>
    <td>192.168.20.38</td>
    <td>Slave</td>
    <td>DataNode、TaskTracker</td>
</tr>
</table>


# 安装Java
1. 检查本机是否已安装Java
在命令行中输入`java -version`判断是否已经安装。如果已经安装检查Java的版本，某些操作系统在安装的时候会安装Jdk，但可能版本会太低。如果版本过低，需要将旧的版本删除。在Redhat操作系统中可以通过rpm命令来删除系统自带的Jdk。
2. 安装java
本文选择jdk1.6安装，将解压出的文件夹jdk1.6.0_10复制到/usr/java目录下。
3. 设置java的环境变量
添加系统环境变量，修改/etc/profile文件，在文件末尾添加如下内容：
> export JAVA_HOME=/usr/java/jdk1.6.0_10
> export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar
> export PATH=$JAVA_HOME/bin:$PATH
> export JRE_HOME=$JAVA_HOME/jre

    修改完profile文件后要执行`source /etc/profile`命令才能使刚才的修改在该命令行环境下生效。
4. 检查java是否安装成功
在命令行中输入java -version、javac命令来查看是否安装成功及安装版本。

# 配置hosts文件
本步骤必须操作，需要root用户来操作，修改完成之后立即生效。在三台机器的/etc/hosts文件末尾添加如下内容：
> 192.168.20.6   server206
> 192.168.20.36   ap1
> 192.168.20.38   ap2

修改完成之后可以通过`ping 主机名`的方式来测试hosts文件是否正确。

# 新建hadoop用户
在三台机器上分别新建hadoop用户，该用户的目录为/home/hadoop。利用useradd命令来添加用户，利用passwd命令给用户添加密码。

# 配置SSH免登录
该步骤非必须，推荐配置，否则在Master上执行start-all.sh命令来启动hadoop集群的时候需要手动输入ssh密码，非常麻烦。
原理：用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录shell，不再要求输入密码。
在本例中，需要实现的是192.168.20.6上的hadoop用户可以无密码登录自己、192.168.20.36和192.168.20.38的hadoop用户。需要将192.168.20.6上的ssh公钥复制到192.168.20.36和192.168.20.38机器上。
1. 在192.168.20.6上执行`ssh-keygen –t rsa`命令来生成ssh密钥对。会在/home/hadoop/.ssh目录下生成id_rsa.pub和id_rsa两个文件，其中id_rsa.pub为公钥文件，id_rsa为私钥文件。
2. 在192.168.20.6上执行`cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`命令将公钥添加到授权的key里。
3. 权限设置。在192.168.20.6上执行`chmod 600 ~/.ssh/authorized_keys`来修改authorized_keys文件的权限，执行`chmod 700 ~/.ssh`命令将.ssh文件夹的权限设置为700。如果权限不对无密码登录就配置不成功，而且没有错误提示，这一步特别注意。
4. 在本机上测试是否设置无密码登录成功。在192.168.20.6上执行`ssh -p 本机SSH服务端口号 localhost`，如果不需要输入密码则登录成功。
5. 利用scp命令将192.168.20.6上的公钥文件id_rsa.pub追加到192.168.20.36和192.168.20.38机器上的~/.ssh/authorized_keys文件中。scp命令的格式如下：
```
scp -P ssh端口号 ~/.ssh/id_rsa.pub hadoop@192.168.20.36:~/id_rsa.pub
```
在192.168.20.36和192.168.20.38机器上分别执行`cat ~/id_rsa.pub >> ~/.ssh/authorized_keys`命令将192.168.20.6机器上的公钥添加到authorized_keys文件的尾部。
6. 配置无密码登录完成，在Master机器上执行`ssh -P 本机SSH服务端口号 要连接的服务器IP地址`命令进行测试。

# 搭建单机版hadoop
在192.168.20.6上首先搭建单机版hadoop进行测试。
1. 将hadoop-0.20.2.tar.gz文件解压到hadoop用户的目录下。
2. 配置hadoop的环境变量。修改/etc/profile文件，在文件的下面加入如下：
> HADOOP_HOME=/home/hadoop/hadoop-0.20.2
> export HADOOP_HOME
> export HADOOP=$HADOOP_HOME/bin
> export PATH=$HADOOP:$PATH

修改完成之后执行`source /etc/profile`使修改的环境变量生效。
3. 配置hadoop用到的java环境变量
修改conf/hadoop-env.sh文件，添加`export JAVA_HOME=/usr/java/jdk1.6.0_10`。
4. 修改conf/core-site.xml的内容如下：
```
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://localhost:9000</value> 
    </property>
    
    <property>
        <name>dfs.replication</name> 
        <value>1</value> 
    </property>
    
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/hadoop-0.20.2/tmp</value> 
    </property>
</configuration>
```
5. 修改/conf/mapred-site.xml的内容如下：
```
<configuration>
    <property>   
        <name>mapred.job.tracker</name>  
        <value>localhost:9001</value>   
    </property>
</configuration>
```
6. 至此单击版搭建完毕。可以通过hadoop自带的wordcount程序测试是否运行正常。下面为运行wordcount例子的步骤。
7. 在hadoop目录下新建input文件夹。
8. 将conf目录下的内容拷贝到input文件夹下，执行`cp conf/* input`。
9. 通过start-all.sh脚本来启动单机版hadoop。
10. 执行wordcount程序：`hadoop jar hadoop-0.20.2-examples.jar wordcount input output`。
11. 通过stop-all.sh脚本来停止单机版hadoop。

# 搭建分布式hadoop
在上述基础之上，在192.168.20.6上执行如下操作。
1\. 修改/home/hadoop/hadoop-0.20.2/conf目录下的core-site.xml文件。
```
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://192.168.20.6:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/hadoop-0.20.2/tmp</value>
    </property>
</configuration>
```
如没有配置hadoop.tmp.dir参数，此时系统默认的临时目录为：/tmp/hadoo-hadoop。而这个目录在每次重启后都会被干掉，必须重新执行format才行，否则会出错。
2\. 修改/home/hadoop/hadoop-0.20.2/conf目录下的hdfs-site.xml文件。
```
<configuration>
    <property>
            <name>dfs.replication</name>
            <value>1</value>
    </property>
    <property>
            <name>dfs.support.append</name>
            <value>true</value>
    </property>
</configuration>
```
3\. 修改/home/hadoop/hadoop-0.20.2/conf目录下的mapred-site.xml文件。
```
<configuration>
    <property>
            <name>mapred.job.tracker</name>
            <value>http://192.168.20.6:9001</value>
    </property>
</configuration>
```
4\. 修改/home/hadoop/hadoop-0.20.2/conf目录下的masters文件。
将Master机器的IP地址或主机名添加进文件，如192.168.20.6。
5\. 修改/home/hadoop/hadoop-0.20.2/conf目录下的slaves文件。*Master主机特有*
在其中将slave节点的Ip地址或主机名添加进文件中，本例中加入
```
192.168.20.36
192.168.20.38
```
6\. hadoop主机的master主机已经配置完毕，利用scp命令将hadoop-0.20.2目录复制到两台slave机器的hadoop目录下。命令为：`scp -r /home/hadoop hadoop@服务器IP:/home/hadoop/`。注意slaves文件在master和slave机器上是不同的。

# 常用命令
* hadoop dfsadmin -report 查看集群状态
* http://192.168.20.6:50070/dfshealth.jsp  查看NameNode状态
* http://192.168.20.6:50030/jobtracker.jsp Map/Reduce管理
* hadoop fs -mkdir input 在HDFS上创建文件夹
* hadoop fs -put ~/file/file*.txt input 将文件放入HDFS文件系统中
 
# 参考文档
* [细细品味Hadoop系列](http://www.cnblogs.com/xia520pi/archive/2012/05/16/2503949.html)。超详细的hadoop教程，作者非常用心。

# 下载链接
[http://pan.baidu.com/share/link?shareid=1235031445&uk=3506813023](http://pan.baidu.com/share/link?shareid=1235031445&uk=3506813023) 提取码：v8ok