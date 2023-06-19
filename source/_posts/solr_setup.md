---
title: 在Linux上搭建solr环境
Status: public
url: solr_setup
tags: Solr
date: 2013-07-02
---

本文采用Linux操作系统在hadoop用户下安装，solr采用3.x中的最新版本3.6.2，tomcat采用6.0.37版本，安装包可以从本文下方链接下载。
这里有两种安装方式，一种方式为利用solr自带的jetty来启动solr，默认端口为8983。另外一种方式为将solr集成到tomcat中。其中第一种方式较为简单，推荐新手采用。

# 独立启动
1. 将sorl的安装包解压到用户的根目录下，解压后文件夹为apache-solr-3.6.2。
2. 进入到example目录下，执行`java -jar start.jar`命令，solr服务启动，端口为8983。
3. 通过`http://IP地址:8983/solr/`来访问solr的web页面，进入admin页面后可以通过输入字符串来查找索引。查找索引默认显示的格式为xml格式，可以通过在url的后面加上参数`wt=json`来显示json格式的结果。

# 利用tomcat
## 安装tomcat
1\. 将apache-tomcat-6.0.37.tar.gz解压到hadoop的跟目录下。
2\. 修改hadoop用户的环境变量，执行`vi ~/.bash_profile`命令，添加如下：
```
export CATALINA_HOME=/home/hadoop/apache-tomcat-6.0.37
export CLASSPATH=.:$JAVA_HOME/lib:$CATALINA_HOME/lib
export PATH=$PATH:$CATALINA_HOME/bin
```
3\. 执行`source ~/.bash_profile `使修改的环境变量生效。
4\. 执行tomcat的bin目录下的startup.bat脚本来启动tomcat。
5\. 通过`netstat -anp | grep 8080`命令查看tomcat是否启动。

## 安装solr
1\. 将solr的dist/apache-solr-3.6.2.war文件复制到tomcat的webapps目录下，并将文件命名为solr.war。执行`cp ~/apache-solr-3.6.2/dist/apache-solr-3.6.2.war ~/apache-tomcat-6.0.37/webapps/solr.war`命令。WAR是一个完整的web应用程序，包括了Solr的jar文件和所有运行Solr所依赖的Jar文件，Jsp和很多的配置文件与资源文件。

2\. 修改`~/apache-tomcat-6.0.37/conf/server.xml`文件相应行的内容如下：
```
<Connector port="8080" protocol="HTTP/1.1" 
    connectionTimeout="20000" 
    URIEncoding="UTF-8"
    redirectPort="8443" />
```
增加`URIEncoding="UTF-8"`来支持中文。这是因为solr基于xml，json，javabin，php，python等多种格式传输请求和返回结果。

3\.复制`~/apache-solr-3.6.2/example/solr`目录到`/home/hadoop/solr`位置。该位置为solr的应用环境目录。

4\. 修改`/home/hadoop/solr/conf/solrconfig.xml`文件中的dataDir一行内容为：
```
<dataDir>${solr.data.dir:/home/hadoop/solr/data}</dataDir>
```
目的是为了指定存放索引数据的路径。

5\. 在`~/apache-tomcat-6.0.37/conf/Catalina/localhost`目录下新建文件`solr.xml`。增加内容如下：
```
<Context docBase="/home/hadoop/apache-tomcat-6.0.37/webapps/solr.war" debug="0" crossContext="true" >
    <Environment name="solr/home" type="java.lang.String" value="/home/hadoop/solr" override="true" />
</Context>
```
其中docBase为tomcat的webapps下的solr.war完整路径。Environment的value属性的值为存放solr索引的文件夹，即第三步中复制的文件夹。
需要注意的是：Catalina目录在首次启动tomcat时创建，因此在此步骤前需要启动过tomcat。

6\. 在tomcat的bin目录下通过`startup.sh`启动tomcat。

7\. 通过`http://IP地址:8080/solr/`来访问solr的web页面。

# 相关命令
## 放入数据到solr中
在apache-solr-3.6.2/example/exampledocs目录下，执行`java -jar post.jar 要存放的文件名`。这里自己新建一个文件test.xml放入到solr中，文件内容如下：
```
<add>
    <doc>  
        <field name="id">company</field>  
        <field name="text">kaitone</field>  
    </doc>
</add>
```
执行`java -jar post.jar test.xml`将数据放入solr中。
## 删除数据
新建文本文件test_delete.xml，内容如下
```
<delete>
    <id>company</id>
</delete>
```
执行`java -jar post.jar test_delete.xml`将数据从solr中删除。
另外还可以通过命令行的方式来删除，命令为`java -Ddate=args -jar post.jar '<delete><id>company</id></delete>'`。

# 在Eclipse中搭建环境操作Solr api
1\. 新建一个java工程
2\. 在工程中引入如下包：

* commons-httpclient-3.1.jar
* commons-codec-1.6.jar
* apache-solr-solrj-3.6.2.jar
* slf4j-api-1.6.1.jar
* slf4j-log4j12-1.6.1.jar
* commons-logging-1.1.3.jar
* log4j-1.2.12.jar
* httpclient-4.2.5.jar
* httpcore-4.2.4.jar
* httpmime-4.2.5.jar

其中`commons-httpclient-3.1.jar`、`commons-codec-1.6.jar`、`apache-solr-solrj-3.6.2.jar`、`slf4j-api-1.6.1.jar`可以从solr的目录`apache-solr-3.6.2`中的dist目录下找到。

`slf4j-log4j12-1.6.1.jar`可以从slf4j的压缩包中`slf4j-1.6.1.tar.gz`找到。

`commons-logging-1.1.3.jar`可以从slf4j的压缩包中`commons-logging-1.1.3-bin.zip`找到。

`log4j-1.2.12.jar`可以从log4j的压缩包中`logging-log4j-1.2.12.tar.gz`找到。

`httpclient-4.2.5.jar`、`httpcore-4.2.4.jar`、`httpmime-4.2.5.jar`在`httpcomponents-client-4.2.5-bin.tar.gz`文件中。

具体的API编程可以参考[Solr开发文档](http://www.cnblogs.com/hoojo/archive/2011/10/21/2220431.html)。

# 在linux上编译并执行程序
1\. 将工程中用到的jar包复制到Linux机器上，这里复制到`/home/hadoop/test_solr/lib`目录下。

2\. 将测试程序的源码放到Linux机器上，这里复制到`/home/hadoop/test_solr`目录下。其中源码包括三个文件：SolrTest.java、SolrClient.java、Index.java。该三个文件将会包含在下面相关下载中的Eclipse工程中。

3\. 在`/home/hadoop/test_solr`目录下执行
```
javac -cp lib/apache-solr-solrj-3.6.2.jar:lib/commons-httpclient-3.1.jar:lib/log4j-1.2.12.jar:lib/commons-codec-1.6.jar:lib/commons-logging-1.1.3.jar:lib/slf4j-api-1.6.1.jar:lib/httpclient-4.2.5.jar:lib/httpcore-4.2.4.jar:lib/httpmime-4.2.5.jar:. SolrTest.java
```
其中-cp等同于-classpath参数，指定编译SolrTest.java文件需要的ClassPath路径，不要忘记路径后面的`.`表示当前路径，否则找不到当前目录下的其他java文件。
命令执行后会在`/home/hadoop/test_solr`目录下生成Index.class、SolrClient.class、SolrTest.class三个class文件。

4\. 在`/home/hadoop/test_solr`目录下执行
```
java -cp lib/apache-solr-solrj-3.6.2.jar:lib/commons-httpclient-3.1.jar:lib/log4j-1.2.12.jar:lib/commons-codec-1.6.jar:lib/commons-logging-1.1.3.jar:lib/slf4j-api-1.6.1.jar:lib/httpclient-4.2.5.jar:lib/httpcore-4.2.4.jar:lib/:httpmime-4.2.5.jar:. SolrTest
```
来运行程序。

# 在Linux上打包并执行
1\. 在上面步骤基础上，为了方便执行，可以将class文件打成jar包来执行，这样在使用java命令执行的时候就不用指定classpath路径了，只需要在jar包的MANIFEST.MF文件中指定classpath。

2\. 在`/home/hadoop/test_solr`下新建一个文件，文件名可以随便，这里取名为MANIFEST.MF，与生成的jar包中的文件名一致，文件内容为
```
Manifest-Version: 1.0
Created-By: 1.6.0_10 (Sun Microsystems Inc.)
Main-Class: SolrTest
Class-Path: /home/hadoop/test_solr/lib/apache-solr-solrj-3.6.2.jar /home/hadoop/test_solr/lib/commons-httpclient-3.1.jar /home/hadoop/test_solr/lib/log4j-1.2.12.jar /home/hadoop/test_solr/lib/commons-codec-1.6.jar /home/hadoop/test_solr/lib/commons-logging-1.1.3.jar /home/hadoop/test_solr/lib/slf4j-api-1.6.1.jar
/home/hadoop/test_solr/lib/httpclient-4.2.5.jar
/home/hadoop/test_solr/lib/httpcore-4.2.4.jar
/home/hadoop/test_solr/lib/httpmime-4.2.5.jar
```
其中Main-Class指定main函数所在的类。
Class-Path指定用到的jar所在的路径。其中Class-Path的各个jar文件之间通过空格分隔而不是通过`:`分隔。

3\. 将class文件打包成jar文件。执行
```
jar -cfm solrtest.jar MANIFEST.MF Index.class SolrClient.class SolrTest.class 
```
会在此目录下生成solrtest.jar文件。jar命令会根据指定的MANIFEST.MF文件来产生jar包中的META-INF/MANIFEST.MF文件。两个文件内容并不完全一致，jar命令会根据格式对内容进行调整。

4\. 运行jar文件。通过`java -jar solrtest.jar`来执行。

# 相关下载
[本文中用到的安装包](http://pan.baidu.com/share/link?shareid=2915568646&uk=3506813023)

# 参考文档
[简单的Solr安装配置](http://demi-panda.com/2013/03/13/install-solr/)
[官方安装教程](https://lucene.apache.org/solr/3_6_2/doc-files/tutorial.html)
[Solr初体验系列](http://cxshun.iteye.com/blog/1039445)讲的非常详细，适合初学者
[Solr开发文档](http://www.cnblogs.com/hoojo/archive/2011/10/21/2220431.html)