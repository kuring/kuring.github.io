---
title: leetcode题目之Majority Element
Status: public
url: leetcode_majority_element
tags: 算法
date: 2015-03-11
toc: yes
---

# 题目

> Given an array of size n, find the majority element. The majority element is the element that appears more than ⌊ n/2 ⌋ times.
> You may assume that the array is non-empty and the majority element always exist in the array.

# 分析

本题是一道非常简单的题目，但我能想到的思路有限，仅能想到排序法和哈希法两种算法，在Solution中提供了另外几种方法，这是非常值得我学习和思考的。本文仅将网站的思路拿过来，可以直接看该问题的[Solution](https://leetcode.com/problems/majority-element/solution/)。

# 解答

## 暴力枚举法

最原始的解决办法，逐个元素比较是否为该数组中的最多元素，只要满足条件即可终止。时间复杂度为O(n^2)。

## 哈希表法

将数组中的元素遍历一遍，并将数组中元素的个数保存到哈希中。然后遍历哈希，从哈希中找到最多元素。时间复杂度O(n)，但需要占用一定的空间。

```c++
int majorityElement(vector<int> &num) {
        std::map<int, int> result_map;
        for (vector<int>::iterator iter = num.begin(); iter != num.end(); iter++)
        {
            if (result_map.find(*iter) == result_map.end())
            {
                result_map.insert(map<int, int>::value_type(*iter, 1));
            }
            else
            {
                result_map[*iter]++;                                                                                                                                            
            }
        }

        int max_count = 0;
        int result;
        for (std::map<int, int>::iterator iter = result_map.begin(); iter != result_map.end(); iter++)
        {
            if (iter->second > max_count)
            {
                result = iter->first;
                max_count = iter->second;
            }
        }
        return result;
    }
```

## 排序法

直接对元素进行排序，排序后元素的中间元素即为要求的最多元素。时间复杂度为O(nlogn)。

```c++
int majorityElementSort(vector<int> &num) {
   sort(num.begin(), num.end());
   return num[num.size() / 2];
}
```
## 随机抽取法

随机从数组中抽取元素，然后遍历数组判断该元素是否为最多元素。该算法利用了最多元素被随机抽取的概率最大的特点，但该算法效率的随机性较大，最好时间复杂度为O(n)，最坏情况下一直随机不到最多元素。

## 二分法

将数组均分为两份，分别求出两个数组中的最多元素A和B，则整个数组中的最多元素必然在两个子数组的最多元素A和B中，这一点可以通过举例子的方式来证明，但是仅凭感觉不太容易得出该结论。如果A==B，则结果就是A。如果A!=B，则分别求出A和B在这个数组中的元素个数。时间复杂度接近O(nlogn)。

## 其他

还有一些比较不容易想到的算法，这里就不列举了。至少我看过一次之后，下次这些算法仍然是记不住的。

