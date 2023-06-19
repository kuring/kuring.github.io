---
title: 在tomcat7.0.41上搭建solr4.3.1
Status: public
url: solr4.3.1_setup
tags: Solr
date: 2013-07-03 10:00:30
---


前几天写了篇《[在Linux上搭建solr环境](/post/solr_setup)》的博文，是基于solr3.6.2的安装。本文仅记录在tomcat7.0.41上搭建solr4.3.1搭建过程中需要注意的地方，其他地方可以参考上一篇博文。

配置完成之后发现http://192.168.20.38:8090/solr无法访问，但是http://192.168.20.38:8090/却可以访问，通过查看tomcat的日志文件localhost.2013-07-03.log，发现里面有如下错误提示。
```
严重: Exception starting filter SolrRequestFilter
org.apache.solr.common.SolrException: Could not find necessary SLF4j logging jars. If using Jetty, the SLF4j logging jars need to go in the jetty lib/ext directory. For other containers, the corresponding directory should be used. For more information, see: http://wiki.apache.org/solr/SolrLogging
```
解决办法：将~/solr-4.3.1/example/lib/ext目录下的所有jar文件复制到~/apache-tomcat-7.0.41/lib目录下，然后重启tomcat即可。

# 相关下载
[用到的文件](http://pan.baidu.com/share/link?shareid=1811860312&uk=3506813023)