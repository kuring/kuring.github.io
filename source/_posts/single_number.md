---
title: leetcode题目之Single Number
Status: public
url: leetcode_single_number
tags: 算法
date: 2015-03-31
toc: yes
---

# 题目一 Single Number

> Given an array of integers, every element appears twice except for one. Find that single one.
> Note:
> Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

# 题目二 Single Number II

> Given an array of integers, every element appears three times except for one. Find that single one.
> Note:
> Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

# 题目一分析及解答

针对题目一，一看就能看出是考察异或操作的特点，并迅速写出了解答方法。

```c++
class Solution {
public:
    int singleNumber(int A[], int n) {
        int result = 0;
        for (int i = 0; i < n; i++)
        {   
            result ^= A[i];
        }   
        return result;
    }
};
```

# 题目二分析及解答

要想实现时间复杂度为O(n)，空间复杂度为O(1)的算法，还是跟题目一一样需要充分利用位操作特性，但是并没有直接可用的位操作特性可以完成，于是想到肯定是各种位操作的组合操作，但是并没有继续向下想到具体的算法。本质上该题目就是模拟一个三进制的操作，当一个位的最大值为2，当为3时直接清0。

参照网上的算法，利用一个int类型的数组来模拟一个三进制数，每个int值的最大值为3，当然这样存在一定空间上的浪费。算法需要将A中的每个值通过移位运算获取到该位的状态，并将值添加到用来模拟三进制的int数组中相应的位置，最后将模拟三进制int数组中的值为3的更改为0。

```c++
class Solution {
public:
    int singleNumber(int A[], int n) {
        int count[32] = {0};
        int result = 0;
        for (int i = 0; i < 32; i++) {
            for (int j = 0; j < n; j++{
                if ((A[j] >> i) & 1) {
                    count[i]++;
                }   
            }   
            result |= ((count[i] % 3) << i); 
        }   
        return result;
    }
};
```

另外，还有上述算法的改进算法，更为节省空间，效率更高，但是确实不容易理解和记忆，属于下次仍然无法记忆的算法类型。这里仅提供代码，不再给出解释，自己领悟。

```c++
int singleNumber(int A[], int n) {
    int ones = 0, twos = 0, threes = 0;
    for (int i = 0; i < n; i++) {
        twos |= ones & A[i];
        ones ^= A[i];// 异或3次 和 异或 1次的结果是一样的
       //对于ones 和 twos 把出现了3次的位置设置为0 （取反之后1的位置为0）
        threes = ones & twos;
        ones &= ~threes;
        twos &= ~threes;
    }
    return ones;
}
```

