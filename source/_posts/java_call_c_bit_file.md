---
title: Java读取C语言写的二进制文件
Status: public
url: java_call_c_bit_file
tags: Java C语言
date: 2013-07-19 15:47:40
---

本程序将讲解java调用C语言写的二进制文件，并将二进制文件中的内容利用Java读出。

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

union data  
{  
    int inter;  
    char ch;    
};

struct Test
{
    int length;
    char arr[20];

    void toBigEndian()
    {   
        union data c;
        c.inter = 1;
        if(c.ch == 1)
        {   
            // 小端
            unsigned char temp;
            unsigned char *tempData = (unsigned char *)&length;
            for (int i=0; i < sizeof(int) / 2; i++)
            {   
                temp = tempData[i];
                tempData[i] = tempData[sizeof(int) - i - 1]; 
                tempData[sizeof(int) - i - 1] = temp;
            }   
        }   
    }   
};

int main()
{
    Test test;
    memset(&test, 0, sizeof(Test));
    test.length = 0x12345678;
    strcpy(test.arr, "hello world");
    test.toBigEndian();
    FILE *file = fopen("test.txt", "w+");
    fwrite(&test, sizeof(Test), 1, file);
    fclose(file);
    return 1;
}
```
本例子中的C程序将一个包含int变量和char数组的结构体写入文件中。

其中需要考虑到机器的大小端问题，java程序采用的大端字节序，因此这里将C的结构体在写入文件时转换成大端字节序。在将结构体写入到文件时，将其中的int类型变量转换成大端字节序，如果机器本身即为大端字节序则不需要转换字节序。

java端读取的文件代码如下：

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;


public class Main {
	
	public static int byte2int(byte[] res) {
		int targets = (res[3] & 0xff) | (res[2] << 8) | (res[1] << 16) | (res[0] << 24);
		return targets; 
	}
	
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		File file = new File("test.txt");
		if (!file.exists()) {
			System.out.println("文件不存在");
			return;
		}
		
		byte[] data = new byte[50];
		
		try {
			FileInputStream fis = new FileInputStream(file);
			int size = fis.read(data);
			System.out.println("读取到" + size + "个字节的数据");
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		
		// 转换完成的int值
		int value = byte2int(data);
		System.out.printf("%x\n", value);
		
		
		StringBuffer sb = new StringBuffer();
		for (int i=4; i<24; i++) {
			System.out.print((char)data[i]);
		}
	}
}
```

