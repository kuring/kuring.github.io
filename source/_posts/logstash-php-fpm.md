---
title: 使用logstash收集php-fpm slow log
Status: public
url: logstash-php-fpm
date: 2016-06-10
toc: yes
---

目前php-fpm的服务部署在了docker中，对php-fpm的log和php error log可以通过syslog协议的形式发送出去，而php-fpm的slow log却不能配置为syslog协议，只能输出到文件中，因为一条slow log的是有多行组成的。

在docker中使用时发现fpm-slowlog不能正常输出，后经发现是docker默认没有ptrace系统调用的权限，而slow log的产生需要该系统调用。通过在docker启动的时候增加"--cap-add SYS_PTRACE"启动项可修正该问题。

为了收集slow log，可以通过logstash、flume等工具进行收集，本文采用logstash对slow log进行收集，并将收集的log写入到kafka中，便于后续的处理。logstash的input采用读取文件的方式，即跟tail -f的原理类似。为了能够将多行日志作为一行，采用了filter中的multiline来对多行日志进行合并操作。logstash的配置如下：

```
input {
    file {
        path => [“/var/log/php-fpm/fpm-slow.log"]
    }
}

filter {
    multiline {
        pattern => "^$"
        negate => true
        what => "previous"
    }
}

output {
    stdout{codec => rubydebug}
    kafka {
        codec => plain {
           format => “tag|%{host}%{message}"
        }
        topic_id => "fpm-slowlog"
        bootstrap_servers => “kafka1.hostname:8082,kafka2.hostname:8082"
    }
}
```

