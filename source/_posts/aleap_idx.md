---
title: asleap中的简单文件索引机制
Status: public
url: aleap_idx
tags: 
date: 2015-02-28
toc: yes
---

asleap是一个开源的vpn破解工具，最近查看了asleap的源码，该项目[地址](http://sourceforge.net/projects/asleap/)。本文的重点是对其中的带索引的字典文件的产生过程进行介绍，产生带索引的字典文件并不复杂，但是要想用简洁易懂的语言将该问题描述明白却不容易。

asleap破解vpn的机制是通过字典文件暴力破解的方式，该字典文件有dat数据文件和idx索引文件两个文件组成，两个文件均为二进制格式。asleap工程中自带了genkey程序，可以将文本的字典文件转换为asleap程序需要的带索引的字典文件。

本文以字典文件为以下内容讲解：

```
turquoise
da
test
```

# 读取字典文件并产生md4值

md4编码占16个字节，三个字典进行md4编码后的结果分别为：

```
18 07 33 43 f6 30 b5 f8 2c 38 c0 34 37 f2 81 6b
01 19 a3 80 94 40 60 3c 57 39 5e 73 f3 60 95 98
0c b6 94 88 05 f7 97 bf 2a 82 80 79 73 b8 95 37
```

# 将字典信息写入到临时文件

为了能够对最终生成的dat文件中的内容进行排序和便于索引，程序生成了256个临时文件，文件名格式为从genk-bucket-00.tmp到genk-bucket-ff.tmp。程序根据md4编码中的第14位将字典对应的信息分别写入到临时文件中，一个字典写入到临时文件的内容如下，如果一个临时文件中存在多个字典则依次存放：

```c
struct hashpass_rec {
    unsigned char rec_size;		// 一个字典占用文件的大小，包括该变量+字典+字典对应的md4值共占用的字节数
    char          *password;	// 字典
    unsigned char hash[16];		// 字典对应的md4值
} __attribute__ ((packed));
```

本例子中turquoise对应结构体会写入到genk-bucket-81.tmp中，da和test对应结构体会依次写入到genk-bucket-95.tmp中。

# 读取临时文件并写入到dat数据文件中

最终dat文件中的数据内容为hashpass_rec的有序集合，排序的原则是按照md4的第14和15两个字节。依次读取256个临时文件中的hashpass_rec可以保证dat文件中的数据内容是按照第14字节排序的，但是不能够保证是按照第15个字节排序的。为了保证最终dat文件中的数据内容是按照第14和15字节有序的，在将一个临时文件中的内容写入到dat文件中前需要对该临时文件中的hashpass_rec结果按照hash变量的第15字节进行排序，直接使用C语言中的qsort进行排序。

该例子中da和test位于同一个临时文件中，需要根据hash变量的第15字节排序的结果为test、da，最终写入到dat文件中的排序结果为turquoise、test、da。

# 根据dat数据文件产生idx索引文件

idx索引文件中存放的是多个hashpassidx_rec结果，最多有256*256项，其结构定义如下：

```c
struct hashpassidx_rec {
    unsigned char	        hashkey[2];	// 对应md4编码的第14和15字节
    off_t                   offset;		// 第一个匹配的hashpass_rec结构在dat文件中的偏移，占用4个字节
    unsigned long long int 	numrec;		// dat文件中共有多少个匹配的hashpass_rec结果
} __attribute__ ((packed));	// 字节对齐，需要填充4个字节
```

最终完成的dat文件和idx文件的指向如下图所示：

![最终完成的dat文件和idx文件的指向](http://kuring.qiniudn.com/aleap_idx.png)

# bug

genkeys.c文件中在读取字典文件时存在bug，在文件的207行将内容更改为：

```c
while (!feof(inputfl)) {
		memset(password, 0, MAX_NT_PASSWORD + 1);
        fgets(password, MAX_NT_PASSWORD+1, inputfl);
		if (strlen(password) == 0) {
			continue;
		}
```

# 相关下载

[字典文件等相关文件下载](http://pan.baidu.com/s/1jGmVF06)

