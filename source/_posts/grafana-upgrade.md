---
title: grafana升级
date: 2017-09-13 00:22:51
tags:
---

本次grafana的升级从版本3.1.1，变更为4.4.3，涉及到一个大的版本跨度。同时之前在使用的存储为sqlite，趁着这次升级更改为mysql。

# grafana升级

直接从官网下载对应的4.4.3版本的二进制包，修改部分配置即可，该部分没任何难度。

# sqlite to mysql

由于grafana使用的表结构在3.1.1到4.4.3之间有变更，不能直接将3.1.1版本的sqlite中的数据导入到4.4.3的mysql中。我的方法为先使用3.1.1版grafana将数据从sqlite导入到mysql中，然后再升级grafana的版本，grafana可以自动修改表结构。

在的mysql中创建grafana的数据库，并修改数据库的编码为utf-8.

修改grafana 3.1.1配置文件conf/defaults.ini如下：

```
[database]
# You can configure the database connection by specifying type, host, name, user and password
# as separate properties or as on string using the url property.

# Either "mysql", "postgres" or "sqlite3", it's your choice
type = mysql
host = xx.xx.xx.xx:3306
name = grafana
user = dev
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
password = dev
```

启动grafana后会自动在grafana数据库中创建相应的表结构，接下来就是将sqlite中的数据导入到mysql中。

在data目录下增加如下脚本sqlitedump.sh，并执行`sh sqlitedump.sh grafana.db > grafana.sql`。

```
#!/bin/sh
DB=$1
TABLES=$(sqlite3 $DB .tables | grep -v migration_log)
for t in $TABLES; do
    echo "TRUNCATE TABLE $t;"
done
for t in $TABLES; do
    echo -e ".mode insert $t\nselect * from $t;"
done | sqlite3 $DB
```

然后将grafana.sql导入到新创建的mysql。

将grafana 3.1.1版本停掉，将grafana 4.4.3版本的配置指向到mysql数据库，启动grafana 4.4.3后，mysql中的表结构会自动变更。

至此，grafana的升级完成。
