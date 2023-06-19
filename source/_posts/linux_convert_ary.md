---
title: Linux下通过命令行进行进制转换
url: linux_convert_ary
Tags: Linux
date: 2014-09-27
---

在Windows下可以通过计算器进行进制之间的转换，非常方便。本文总结Linux下可用的进行进制之间的转换方法。

# 利用shell

shell脚本默认数值是由10进制数处理，除非这个数字某种特殊的标记法或前缀开头，才可以表示其它进制类型数值。如：以0开头就是8进制、以0x开头就是16进制数。使用“BASE#NUMBER”这种形式可以表示其它进制。BASE值的范围为2-64。

其他进制转10进制

```shell
kuring@T420:~$ echo $((16#4000000))
67108864
kuring@T420:~$ echo $((2#111))
7
kuring@T420:~$ echo $((0x10))
16
kuring@T420:~$ echo $((010))
8

// 对进制转换为10进制进行运算，稍显啰嗦
kuring@T420:~$ echo $(($((16#4000000))/1014/1024));
64
```

# 利用let命令

let用来执行算数运算和数值表达式测试。可以利用该命令完成简单的计算，并将计算结果赋给其他变量。

```shell
// 对进制转换为10进制后进行运算，比纯shell方式简洁
kuring@T420:~$ let a=$((16#4000000))/1014/1024;
kuring@T420:~$ echo $a
64
```

# 利用bc命令

该命令是一个强大的计算器软件，可以利用其中的ibase和obase进行输入进制的转换，ibase表示输入数字的进制，obase表示输出数字的进制。

```shell
// 如果没有制定，则默认为10进制
// 对于16进制，要使用F，而不能使用f
kuring@T420:~$ echo "ibase=16;FF" | bc
255
kuring@T420:~$ echo "obase=2;ibase=16;FF" | bc
11111111
```

# 参考文章

[linux shell 不同进制数据转换（二进制，八进制，十六进制，base64)](http://www.cnblogs.com/chengmo/archive/2010/10/14/1851570.html)

[Shell进制转换小结](http://blog.csdn.net/fengyuxing168/article/details/8896955)


