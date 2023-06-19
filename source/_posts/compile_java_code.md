---
title: 通过命令编译java程序
Status: public
url: compile_java_code
tags: Java
date: 2013-07-01 11:04:01
---

通过Eclipse编写java程序久了，发现已经不会用命令来编译java程序了。今天在windows下搭建了一个solr环境，想放到linux下去跑一下，在windows上打成jar包后放在linux下不能运行，是时候回顾一下java的编译命令了。而且网上的资料比较零散，没有特别系统的资料。

本文在linux测试，同windows下的命令行工具差别不大。

# 编译并执行单个文件
1\. 在目录下`~/test_java/com/kuring`下新建HelloWorld.java的文件，文件内容为
```java
package com.kuring;

public class HellowWorld {
    public static void main(String[] args) {
        System.out.println("hello world");
    }
}
```

2\. 在目录`~/test_java`下执行`javac com/kuring/HelloWorld.java`命令来编译文件。此时会在HelloWorld.java文件所在的目录下生成HelloWorld.class的二进制文件。

3\. 在目录`~/test_java`下执行`java com.kuring.HelloWorld`来执行HelloWorld.class。屏幕会输出`hello world`，说明文件执行成功。
也可以在任意路径下指定classpath路径来执行，命令为`java -classpath 
~/test_java com.kuring.HelloWorld`，其中classpath指定了类的搜索路径。

# 编译并执行多个文件
1\. 在目录下`~/test_java/com/kuring`下新建HelloWorld2.java和Main.java的文件，HelloWorld2.java文件内容为
```java
package com.kuring;

public class HellowWorld2 {
    public void print() {
		System.out.println("hello world too");
	}
}
```
Main.java的文件内容为
```java
package com.kuring;

public class Main {
	public static void main(String[] args) {
		HelloWorld2 hello = new HelloWorld2();
		hello.print();
	}
}
```

2\. 在目录`~/test_java`下执行`javac com/kuring/Main.java`命令来编译文件。此时会在Main.java文件所在的目录下生成Main.class和HelloWorld2.class两个文件，可以看出javac有自动推导编译的功能。

3\. 在目录`~/test_java`下执行`java com.kuring.Main`。屏幕会输出`hello world too`，说明文件执行成功。

# 打包
将上述例子中的程序打成jar包，可以在`~/test_java`目录下通过执行命令`jar cvf my.jar com`来生成jar文件。其中my.jar为要生成的jar文件的名字。
通过`java -classpath my.jar com.kuring.Main`来执行jar文件。
上述命令需要指定要执行的类名Main，如果想通过`java -jar my.jar`命令即可执行程序需要在jar包的META-INF/MANIFEST.MF文件中增加一行
```
Main-Class: SolrTest
```
来执行含有main函数的类。然后通过`jar -cfm my.jar MANIFEST.MF路径 要打包的目录或文件`来重新生成jar包。这样就可以通过`java -jar my.jar`来执行jar包了。

关于如何创建并执行引用了其他jar包的jar包，可以参考我的另外一篇博客《[在Linux上搭建solr环境](/post/solr_setup)》，这里不再赘述。

# 常用jar命令

<table border="1" cellpadding="3" cellspacing="0" summary="" width="100%">
    <tr>
        <td><strong>功能</strong></td>
        <td><strong>命令</strong></td>
    </tr>
    <tr>
        <td>用一个单独的文件创建一个 JAR 文件</td>
        <td>jar cf jar-file input-file...</td>
    </tr>
    <tr>
        <td>用一个目录创建一个 JAR 文件</td>
        <td>jar cf jar-file dir-name</td>
    </tr>
    <tr>
        <td>创建一个未压缩的 JAR 文件</td>
        <td>jar cf0 jar-file dir-name</td>
    </tr>
    <tr>
        <td>更新一个 JAR 文件</td>
        <td>jar uf jar-file input-file...</td>
    </tr>
    <tr>
        <td>查看一个 JAR 文件的内容</td>
        <td>jar tf jar-file</td>
    </tr>
    <tr>
        <td>提取一个 JAR 文件的内容</td>
        <td>jar xf jar-file</td>
    </tr>
    <tr>
        <td>从一个 JAR 文件中提取特定的文件</td>
        <td>jar xf jar-file archived-file...</td>
    </tr>
    <tr>
        <td>运行一个打包为可执行 JAR 文件的应用程序</td>
        <td>java -jar app.jar</td>
    </tr>
</table>

# 参考文档
[JAR 文件揭密](http://www.ibm.com/developerworks/cn/java/j-jar/)
[Java程序的编译、执行和打包](http://blog.sina.com.cn/s/blog_774c75080100q73w.html)