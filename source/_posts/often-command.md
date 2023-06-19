title: 日常工作中经常用到的命令
date: 2019-08-16 20:26:03
tags:
---
shell的命令千千万，工作中总有些命令是经常使用到的，本文记录一些常用到的命令，用于提高效率。

## python

快速开启一个http server `python -m SimpleHTTPServer 8080`。

在python3环境下该命令变更为：`python3 -m http.server 8080`

格式化 json: `cat 1.json | python -m json.tool`

## awk

按照,打印出一每一列 `awk -F, '{for(i=1;i<=NF;i++){print $i;}}'`

## docker registry

- 列出镜像：`curl http://127.0.0.1:5000/v2/_catalog?n=1000`
- 查询镜像的tag: `curl http://127.0.0.1:5000/v2/nginx/tags/list`，如果遇到镜像名类似`aa/bb`的情况，需要转移一下 `curl http://127.0.0.1:5000/v2/aa\/bb/tags/list`
