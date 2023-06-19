---
title: google autheticator应用现状
date: 2018-02-27 22:34:17
tags:
---

通过使用Google的登陆二步验证（即Google Authenticator服务），我们在登陆时需要输入额外由手机客户端生成的一次性密码。大大提高登陆的安全性。

实现Google Authenticator功能需要服务器端和客户端的支持。服务器端负责密钥的生成、验证一次性密码是否正确。客户端记录密钥后生成一次性密码。

google实现了基于时间的TOTP算法（Time-based One-time Password），客户端实现包括了android和ios。

算法为公开算法，google没有提供服务端的实现，各个语言都有单独的实现。自己系统使用可以直接使用网上的代码。

linux下有libpam-google-authenticator模块，可以使用yum或者源码编译安装，github上有源码，编译出来的为so文件，可以加到sshd的配置文件中，用于给sshd提供二次认证机制。

客户端和服务端存在时间差的问题，google authenticator的超时时间为30s，服务端可以使用两个30s的时间来验证，包括当前和上一个30s。
