title: 证书技术
date: 2022-01-07 10:34:15
tags:
---
# 使用openssl创建自签名证书
创建CA
```powershell
openssl genrsa -out root.key 4096
openssl req -new -x509 -days 1000 -key root.key -out root.crt
openssl x509 -text -in root.crt -noout
```
创建私钥和公钥文件
```powershell
# 用来产生私钥文件server.key
openssl genrsa -out server.key 2048
# 产生公钥文件
openssl rsa -in server.key -pubout -out server.pem
```
创建签名请求
```powershell
openssl req -new -key server.key -out server.csr

# 查看创建的签名请求
openssl req -noout -text -in server.csr
```
创建自签名证书
新建自签名证书的附加信息server.ext，内容如下
```powershell
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
IP.2 = 127.0.0.1
IP.3 = 10.66.3.6
```

使用ca签发ssl证书，此时会产生server.crt文件，即为证书文件
```powershell
openssl x509 -req -in server.csr -out server.crt -days 3650 \
  -CAcreateserial -CA root.crt -CAkey root.key \
  -CAserial serial -extfile server.ext
```

查看证书文件内容
```powershell
openssl x509 -in server.crt   -noout -text
```

使用ca校验证书是否通过
```powershell
openssl verify -CAfile root.crt server.crt
```

# 使用cfssl签发证书

cfssl是CloudFlare开源的一款tls工具，使用go语言编写，地址：https://github.com/cloudflare/cfssl。

## 安装

mac用户：brew install cfssl

linux用户可以直接下载二进制文件：https://github.com/cloudflare/cfssl/releases

## 签发证书

待补充

# 证书格式
证书按照格式可以分为二进制和文本文件两种格式。

二进制格式分为：

1. *.der或者*.cer：用来存放证书信息，不包含私钥。

文本格式分为：

1. *.pem：存放证书或者私钥。一般是*.key文件存放私钥信息。对于pem或者key文件，如果存在**——BEGIN CERTIFICATE——**，则说明这是一个证书文件。如果存在**—–BEGIN RSA PRIVATE KEY—–**，则说明这是一个私钥文件。
2. *.key：用来存放私钥文件。
3. *.crt：证书请求文件，格式的开头为：-----BEGIN CERTIFICATE REQUEST-----

## 证书格式的转换
将cert证书转换为pem格式
```powershell
openssl rsa -in server.key -text > server-key.pem
openssl x509 -in server.crt -out server.pem
```
将pem格式转换为cert格式

# 证书查看

查看特定域名的证书内容：`echo | openssl s_client  -connect www.baidu.com:443 -servername www.baidu.com 2>/dev/null | openssl x509 -noout -text`

获取特定域名的证书内容：`openssl s_client -servername www.baidu.com -connect www.baidu.com:443 </dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p'`

在 k8s 中，证书通常存放在 secret 中，可以使用 kubeclt + openssl 命令查看证书内容：`kubectl get secret -n test-cluster sharer -o jsonpath='{.data.tls\.crt}' | base64 --decode | openssl x509 -text -noout`

# 证书的使用

curl命令关于证书的用法：

- --cacert：指定ca来校验server端的证书合法性
- --cert：指定客户端的证书文件，用在双向认证mTLS中
- --key：私钥文件名，用在双向认证mTLS中

# 参考链接
- [https://gist.github.com/liuguangw/4d4b87b750be8edb700ff94c783b1dd4](https://gist.github.com/liuguangw/4d4b87b750be8edb700ff94c783b1dd4)
- [https://coolshell.cn/articles/21708.html](https://coolshell.cn/articles/21708.html)
- [https://help.aliyun.com/document_detail/160093.html](https://help.aliyun.com/document_detail/160093.html)
