---
URL: coolshell_puzzle
title: coolshell博客解谜题游戏
Status: public
date: 2014-08-20
---


有些闲暇时间了解了下酷壳的谜题活动，共10道题，每道题都不是非常简单，我这里参考着攻略做了下。

# 字符替换题

我编写的C++语言程序

```c++
#include <stdio.h>
#include <string.h>

char change(char input)
{
    char *after = "abcdefghijklmnopqrstuvwxyz";
    char *before = "pvwdgazxubqfsnrhocitlkeymj";
    for (int i=0; i<strlen(before); i++)
    {
        if (before[i] == input)
        {
            return after[i];
        }
    }
    return input;
}

int main()
{
    char *input = "Wxgcg txgcg ui p ixgff, txgcg ui p epm. I gyhgwt mrl lig txg ixgff wrsspnd tr irfkg txui hcrvfgs, nre, hfgpig tcm liunz txg crt13 ra \"ixgff\" tr gntgc ngyt fgkgf.";
    char *output = new char[strlen(input) + 1];
    for (int i=0; i<strlen(input); i++)
    {
        output[i] = change(input[i]);
    }
    output[strlen(input)] = '\0';
    printf("%s\n", output);
    return 1;
}
```

好久没有用shell了，又写了个shell版本的解题方法，该问题可能有更简单的shell解决办法，我这里肯定写复杂了。

```bash
#!/bin/bash

result=''

function change()
{
    result=$1
    before='pvwdgazxubqfsnrhocitlkeymj'
    after='abcdefghijklmnopqrstuvwxyz'
    for ((i=0; i <= ${#before}; i++))
    do
        if [[ ${before:${i}:1} = ${1} ]]
        then
            result=${after:${i}:1}
            break
        fi
    done
}

input='Wxgcg txgcg ui p ixgff, txgcg ui p epm. I gyhgwt mrl lig txg ixgff wrsspnd tr irfkg txui hcrvfgs, nre, hfgpig tcm liunz txg crt13 ra "ixgff" tr gntgc ngyt fgkgf.'

output=''
j=0
while [ "$j" -le ${#input} ]
do
    change "${input:${j}:1}"
    output="${output}${result}"
    j=$((j+1))
done
echo ${output}
```

关于rol13的转码可以采用[rot13](http://rot13.de/index.php)这个网址来在线转码。

# 穷举变量题

该题需要不断的请求url来获取最终的网址，我这里写一个shell脚本来穷举。

```bash
#!/bin/bash

res=2014
while [ ${#res} > 0 ]
do
res=`curl -s "http://fun.coolshell.cn/n/${res}"`
echo $res
done
```

得到答案tree

# 参考

[游戏页面](http://fun.coolshell.cn/)
[CoolShell puzzle game 攻略](http://blog.e10t.net/coolshell-puzzle-game-walkthrough/)
[我也不产生代码 -- Coolshell 谜题一游 ](http://joseph.yy.blog.163.com/blog/static/5097395920147782756165/)