---
title: leetcode题目之Min Stack
Status: public
url: leetcode_min_stack
tags: 算法
date: 2014-09-27
---

# 题目

> Memory Limit Exceeded

> Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.

> push(x) -- Push element x onto stack.
> pop() -- Removes the element on top of the stack.
> top() -- Get the top element.
> getMin() -- Retrieve the minimum element in the stack.

# 错误代码

看到此题目，以为是用实现一个简单的栈结构，于是就直接写下了如下代码，采用了双向链表的方式来实现：

```c
class MinStack {
public:
	struct Node
	{
		int data;
		Node *next;
		Node *pre;
		Node() : next(NULL),pre(NULL) {}; 
	};
	
	MinStack()
	{
		header = new Node();
		tail = header;
	}
	
    void push(int x) {
        Node *node = new Node();
		node->data = x;
		node->pre = tail;
		tail->next = node;
		tail = node;
    }

    void pop() {
        Node *pre = tail->pre;
		delete tail;
		tail = pre;
		tail->next = NULL;
    }

    int top() {
		if (tail == header)
		{
			return -1;
		}
        return tail->data;
    }

    int getMin() {
		Node *begin = header->next;
		int min = INT_MIN;
        while(begin)
		{
			if (min > begin->data)
			{
				min = begin->data;
			}
			begin = begin->next;
		}
		return min;
    }
	
private:
	Node *header;
	Node *tail;
};
```

提交后提示`Memory Limit Exceeded`错误，开始考虑是不是双向链表占用的空间过多。

# 正确代码

仔细看题目后发现`retrieving the minimum element in constant time`，即时间复杂度为O(1)，上述实现代码时间复杂度为O(n)，明显不符合要求。看来理解题目有误，不是为了实现栈类，而是为了利用数据结构解决获取最小值问题。

经过考虑后可以通过在类内部维护存储最小值的栈来解决，存储最小值的栈除了需要存储最小值外，还需要维护最小值个数。

实现代码如下：

```c
class MinStack {
	struct MinUnit
	{
		int count;
		int value;
		MinUnit(int count, int value) : count(count), value(value) {};
	};
public:
    void push(int x) {
        data_stack.push(x);
		if (min_stack.empty() || x < min_stack.top()->value)
		{
			MinUnit *p_unit = new MinUnit(1, x);
			min_stack.push(p_unit);
		}
		else if (x == min_stack.top()->value)
		{
			min_stack.top()->count++;
		}
    }

    void pop() {
        if (data_stack.top() == min_stack.top()->value)
		{
			if (min_stack.top()->count == 1)
			{
				MinUnit *punit = min_stack.top();
				min_stack.pop();
				delete punit;
			}
			else
			{
				min_stack.top()->count--;
			}
		}
		data_stack.pop();
    }

    int top() {
		return data_stack.top();
    }

    int getMin() {
		return min_stack.top()->value;
    }
	
private:
	std::stack<int> data_stack;
	// 最小栈，为了便于修改栈单元中的count值，采用存储指针方式
	std::stack<MinUnit*> min_stack;
};
```
