---
title: leetcode题目之Maximum Subarray
Status: public
url: leetcode_maximum_subarray
tags: 算法
date: 2015-01-02
toc: yes
---

# 题目

Find the contiguous subarray within an array (containing at least one number) which has the largest sum.

For example, given the array [−2,1,−3,4,−1,2,1,−5,4],
the contiguous subarray [4,−1,2,1] has the largest sum = 6.

# 分析

该题目为经典题目，存在多种解题思路。

# 动态规划

求动态规划的关键在于找到状态方程。

```c++
/**
 * 问题的关键是找到状态方式，找到状态方程后问题就迎刃而解
 * 状态方程如下：
 * b[j]表示第j处，以a[j]结尾的子序列的最大和
 * b[j]=max(a[j] + b[j-1], a[j])
 * b数据的最大值即为问题的解
 * 问题转换为求解b数组
 * 时间复杂度为O(1)，空间复杂度为(n)，空间复杂度可以降为O(1)，为了使程序易读，不做调整
 */
int maxSubArray(int A[], int n) {
	if (n == 0)
	{
		return 0;
	}
	int *b = new int[n];
	b[0] = A[0];
	int max_b = b[0];
	for (int i=1; i<n; i++)
	{
		b[i] = std::max(A[i] + b[i-1], A[i]);
		if (max_b < b[i])
		{
			max_b = b[i];
		}
	}
	delete[] b;
	return max_b;
}
```

# 分治法

《算法导论》的分治策略一章有关于该问题的详细解释。该题利用分治法来解决要比二分查找类最简单的分治算法要复杂。将数组一分为二后，最大数组存在三种情况：在左半或右半部分、跨越中点分别占据左部分一点和右部分一点。对于跨越中点的情况，转化为求从中点开始向左的最大值和从中点开始向右的最大值之和。

```c++
class Solution {
public:
    int compare_array[4];

    int maxSubArray(int A[], int n) {
        return maxSubArray(A, 0, n - 1);
    }

    /**
     * leetcode not support stdarg.h
     */
    int max(int count, ...)
    {
        va_list ap;
        va_start(ap, count);
        int max = INT_MIN;
        for (int i=0; i<count; i++)
        {
            int temp = va_arg(ap, int);
            if (max < temp)
            {
                max = temp;
            }
        }
        va_end(ap);
        return max;
    }

    int max_compare_array()
    {
        int max_num = compare_array[0];
        for (int i=1; i<4; i++)
        {
            if (max_num < compare_array[i])
            {
                max_num = compare_array[i];
            }
        }
        return max_num;
    }

    int maxSubArray(int A[], int begin, int end)
    {
        //printf("begin : %d, end : %d\n", begin, end);
        if (begin == end)
        {
            return A[begin];
        }
        else if ((end - begin) == 1)
        {
            //return max(3, A[begin], A[begin] + A[end], A[end]);
            compare_array[0] = A[begin];
            compare_array[1] = A[begin] + A[end];
            compare_array[2] = A[end];
            compare_array[3] = INT_MIN;
            return max_compare_array();
        }
        int middle = (begin + end) / 2;
        // 处理左边子数组
        int max_left = maxSubArray(A, begin, middle);
        // 处理右边子数组
        int max_right = maxSubArray(A, middle + 1, end);
        // 处理跨越中点的情况
        int max_cross = maxCrossMiddle(A, begin, end);

        printf("begin : %d, end : %d, max_left = %d, max_right = %d, max_cross = %d\n", begin, end, max_left, max_right, max_cross);

        // 返回三者中的最大值
        compare_array[0] = max_left;
        compare_array[1] = max_right;
        compare_array[2] = max_cross;
        compare_array[3] = INT_MIN;
        return max_compare_array();
    }

    /**
     * 处理跨越中点的情况
     */
    int maxCrossMiddle(int A[], int begin, int end)
    {
        if (begin == end)
        {
            return A[begin];
        }
        int middle = (begin + end) / 2;
        // 求得[begin -- middle-1]的最大值
        int max_left = A[middle - 1];
        int sum = 0;
        for (int i=middle - 1; i>=begin && i >= 0; i--)
        {
            sum += A[i];
            if (max_left < sum)
            {
                max_left = sum;
            }
        }

        // 求得[middle+1 -- end]的最大值
        int max_right = A[middle + 1];
        sum = 0;
        for (int i=middle + 1; i<=end; i++)
        {
            sum += A[i];
            if (max_right< sum)
            {
                max_right = sum;
            }
        }

        compare_array[0] = A[middle];
        compare_array[1] = A[middle] + max_left;
        compare_array[2] = A[middle] + max_right;
        compare_array[3] = A[middle] + max_left + max_right;
        return max_compare_array();
    }
};
```

# 扫描算法

《编程珠玑》一书8.4节提到该算法，时间复杂度为O(1)，是解决该问题最好的算法。

```c++
int maxSubArray(int A[], int n) {
	if (n == 0)
	{
	    return 0;
	}
	int current_sum = 0;
	int max_sum = INT_MIN;

	for (int i=0; i<n; i++)
	{
	    if (current_sum <= 0)
	    {
		current_sum = A[i];
	    }
	    else
	    {
		current_sum += A[i];
	    }
	    if (current_sum > max_sum)
	    {
		max_sum = current_sum;
	    }
	}
	return max_sum;
}
```
