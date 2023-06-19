---
title: ELK解析nginx日志
Status: public
url: elk_nginx
tags: ELK
date: 2016-02-18
toc: yes
---

ELK解析nginx日志

最近使用ELK搭建了一个nginx的日志解析环境，中间遇到一些挫折，好不容易搭建完毕，有必要记录一下。

#  nginx

nginx配置文件中的日志配置如下：

```
error_log /var/log/nginx/error.log;
log_format  main  '$remote_addr [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

access_log /var/log/nginx/access.log  main;
```

# logstash

由于是测试环境，我这里使用logstash读取nginx日志文件的方式来获取nginx的日志，并且仅读取了nginx的access log，对于error log没有关心。

使用的logstash版本为2.2.0，在log stash程序目录下创建conf文件夹，用于存放解析日志的配置文件，并在其中创建文件test.conf，文件内容如下：

```
input {
    file {
        path => ["/var/log/nginx/access.log"]
    }
}
filter {
    grok {
        match => {
            "message" => "%{IPORHOST:clientip} \[%{HTTPDATE:time}\] \"%{WORD:verb} %{URIPATHPARAM:request} HTTP/%{NUMBER:httpversion}\" %{NUMBER:http_status_code} %{NUMBER:bytes} \"(?<http_referer>\S+)\" \"(?<http_user_agent>\S+)\" \"(?<http_x_forwarded_for>\S+)\""
        }
    }   
}
output {
    elasticsearch {
        hosts => ["10.103.17.4:9200"]
        index => "logstash-nginx-test-%{+YYYY.MM.dd}"
        workers => 1
        flush_size => 1
        idle_flush_time => 1
        template_overwrite => true
    }
    stdout{codec => rubydebug}
} 
```

需要说明的是，filter字段中的grok部分，由于nginx的日志是格式化的，logstash解析日志的思路为通过正则表达式来匹配日志，并将字段保存到相应的变量中。logstash中使用grok插件来解析日志，grok中message部分为对应的grok语法，并不完全等价于正则表达式的语法，在其中增加了变量信息。

具体grok语法不作过多介绍，可以通过logstash的官方文档中来了解。但grok语法中的变量类型如IPORHOST并未找到具体的文档，只能通过在logstash的安装目录下通过`grep -nr "IPORHOST" .`来搜索具体的含义。

配置文件中的stdout部分用于打印grok解析结果的信息，在调试阶段一定要打开。

可以通过[这里](http://grokconstructor.appspot.com/do/match)来验证grok表达式的语法是否正确，编写grok表达式的时候可以在这里编写和测试。

对于elasticsearch部分不做过多介绍，网上容易找到资料。

# kibana

kibana不做过多介绍，使用可以查看官方文档和自己摸索。


# reference

[logstash中的grok插件介绍](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html)
