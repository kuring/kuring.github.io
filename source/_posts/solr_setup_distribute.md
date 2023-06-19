---
title: 搭建分布式的solr环境
Status: private
url: solr_setup_distribute
tags: Solr
date: 2013-07-02 14:29:23
---


本文以《[在Linux上搭建solr环境](/post/solr_setup)》为基础，假设已经在192.168.20.6和192.168.20.38上搭建了单机版solr环境。

# 主服务器配置
找到solr的环境目录下的conf文件夹下的solrconfig.xml文件，我的是在`/hadoop/solr/conf/solrconfig.xml`目录下，打开后找到如下行
```
<requestHandler name="/replication" class="solr.ReplicationHandler" >
```
默认是被注释的，将其修改为
```
<requestHandler name="/replication" class="solr.ReplicationHandler" >
       <lst name="master">
         <str name="replicateAfter">commit</str>
         <str name="replicateAfter">startup</str>
         <str name="confFiles">schema.xml,stopwords.txt</str>
       </lst>
</requestHandler>
```
replicateAfter表示solr会在什么情况下复制，可选项包括：commit、startup、optimize，这里保持默认。
confFiles表示要分发的配置文件。

# 从服务器配置
在从服务器上，将`/hadoop/solr/conf/solrconfig.xml`文件相应的修改为
```
<requestHandler name="/replication" class="solr.ReplicationHandler" >
       <lst name="slave">
         <str name="masterUrl">http://192.168.20.6:8080/solr/replication</str>
         <str name="pollInterval">00:00:60</str>
       </lst>
</requestHandler>
```
masterUrl为服务器的url地址。
pollInterval为从服务器的同步时间间隔。