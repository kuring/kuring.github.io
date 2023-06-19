---
title: Unique Binary Search Trees II
Status: public
url: leetcode_unique_binary_search_trees_ii
tags: 算法
date: 2014-09-29
---

# 问题

Given n, generate all structurally unique BST's (binary search trees) that store values 1...n.

For example,
Given n = 3, your program should return all 5 unique BST's shown below.

<pre style="font-family: Courier, monospace;">
   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
</pre>

# 分析

要想能够生成多个树并存储到vector中，最容易想到的就是递归算法。要想能够递归，题目中提供的函数仅有一个参数，结合题目不能够完成递归的条件，考虑到unique binary search trees中的解法，需要递归具有两个参数的函数。

考虑到了递归的问题，还需要利用循环不断将树添加到vector中，这编写起来也是比较有难度，需要掌握循环的次数和什么时候将树添加到vector中。

# 代码

```c++
vector<TreeNode *> generateTrees(int n) {
	vector<TreeNode *> sub_tree = generateTrees(1, n);
	return sub_tree;
}

vector<TreeNode *> generateTrees(int low, int high) {
	vector<TreeNode *> result;
	if (low > high)
	{
		result.push_back(NULL);
		return result;
	}
	else if (low == high)
	{
		TreeNode *node = new TreeNode(low);
		result.push_back(node);
		return result;
	}
	
	for (int i=low; i<=high; i++)
	{
		vector<TreeNode *> left = generateTrees(low, i - 1);
		vector<TreeNode *> right = generateTrees(i + 1, high);
		for (int j=0; j<left.size(); j++)
		{
			for (int k=0; k<right.size(); k++)
			{
				TreeNode *root = new TreeNode(i);
				root->left = left[j];
				root->right = right[k];
				result.push_back(root);
			}
		}
	}
	return result;
}
```
