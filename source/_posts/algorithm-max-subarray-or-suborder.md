---
title: 最长公共子序列和最长公共子串问题
date: 2019-04-05 22:18:05
tags:
---

最长公共子串及公共子序列问题属于一类问题，都可以使用动态规划的算法来解析，且动态规划方式比较类似，比较容易混淆。

## 定义

最长公共子串：两个字符串中，相同的最长子串，字符必须是连续的

最长公共子序列：两个字符串中，相同的最长序列，字符不一定是连续的

比如：a[] = "abcde" b[] = "bce"

那么：
最长子串："bc"
最长子序列："bce"

## 解法

假设A、B分别表示两个字符串

### 最长公共子串

dp[i][j]表示子串A[:i]、B[:j]必须以A[i]、B[j]为结尾的两个字符串的最大子串长度

```
if A[i] == B[j] {
    dp[i][j] = dp[i-1][j-1]
} else {
    dp[i][j] = 0
}
```

最终dp二维数组中的最大值即为结果

### 最长公共子序列

dp[i][j]表示子串A[:i]、B[:j]的两个字符串的最大子序列

```
if A[i] == B[j] {
    dp[i][j] = dp[i-1][j-1]
} else {
    dp[i][j] = max(dp[i-1][j], dp[i][j-1])
}
```

最终dp[i-1][j-1]为结果

## leetcode

- [Delete Operation for Two Strings](https://leetcode.com/problems/delete-operation-for-two-strings/) （最长公共子序列）
- [Maximum Length of Repeated Subarray](https://leetcode.com/problems/maximum-length-of-repeated-subarray/) （最长公共子串）
