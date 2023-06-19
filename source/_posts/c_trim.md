---
title: 用C语言实现的trim函数
Status: public
url: c_trim
tags: C语言
date: 2014-02-18
---

trim函数在其他语言中比较常见，这里用C语言实现一个，不使用C语言的库函数。该例子中不需要额外的申请空间，算法的时间复杂度为O(1)。

```
#include <stdio.h>

char *trim(char * str)
{
	char *buf1, *buf2;
	int i;
	if (str == NULL)
	{
		return NULL;
	}
	
	// 处理字符串前面的空格	
	for (buf1=str; *buf1 && *buf1==' '; buf1++);

	// 将去掉前面空格的字符串向前复制
	for (buf2=str, i=0; *buf1;)
	{
		*buf2++ = *buf1++;
		i++;
	}
	*buf2 = '\0';

	// 处理字符串后面的空格
	while (*--buf2 == ' ')
	{
		*buf2 = '\0';
	}
	return str;	
}

int main(int argc, char *argv[])
{
      printf("trim(\"%s\") ", argv[1]);
      printf("returned \"%s\"\n", trim(argv[1]));
      return 0;
}

```