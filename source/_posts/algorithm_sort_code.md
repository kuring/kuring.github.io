---
title: 常用排序算法整理及代码实现
Status: public
url: algorithm_sort_code
tags: 算法
date: 2015-02-25
toc: yes
---

本文对我编写的常用的排序算法进行整理和总结，方便用时进行查阅和参考。

# 快速排序

快速排序是实际应用中的最好选择，采用了分治法的思想。通过一趟排序将待排序记录分割成独立的两部分，其中一部分的关键字均比另外一部分的小，分别对这两部分记录进行排序，已达到整个有序。

是否稳定：不稳定

时间复杂度：O(nlogn)

空间复杂度：O(logn)，需要栈来实现递归用

```c++
#include <stdio.h>
#include <vector>
#include <iostream>

int partition(std::vector<int> &numbers, int low, int high)
{
    int pivotkey = numbers[low];
    while (low < high)
    {
        while (low < high && numbers[high] >= pivotkey)
        {
            --high;
        }
        numbers[low] = numbers[high];
        while (low < high && numbers[low] <= pivotkey)
        {
            ++low;
        }
        numbers[high] = numbers[low];
        numbers[low] = pivotkey;
    }
    return low;
}


void quick_sort(std::vector<int> &numbers, int low, int high)
{
    if (low < high)
    {
        int pivotloc = partition(numbers, low, high);
        quick_sort(numbers, low, pivotloc - 1);
        quick_sort(numbers, pivotloc + 1, high);
    }
}

void quick_sort(std::vector<int> &numbers)
{
    quick_sort(numbers, 0, numbers.size() - 1);
}

int main()
{
    std::vector<int> numbers = {49, 38, 65, 97, 76, 13, 27, 49};
    quick_sort(numbers);
    for (int i=0; i<numbers.size(); i++)
    {
        printf("%d\t", numbers[i]);
    }
    printf("\n");
}
```

# 归并排序

将两个或两个以上的有序表组合成一个新的有序表。合并两个有序表的方法为：比较两个有序表中第一个数，谁小先取谁。继续进行比较，只要有一个有序表为空，直接将另一个有序表取出即可。

是否稳定：稳定

时间复杂度：O(nlogn)

空间复杂度：O(n) （当使用顺序存储时，为了能够实现两个有序表之间的合并），或O(1)（当使用链式存储的时候，不再需要临时的空间来存储排序的结果）

## 顺序存储代码

以下为采用顺序存储结构的代码：

```c++
/**
 * 归并排序使用递归算法的效率比较低，具体应用中会采用非递归算法代替
 */
void merge(std::vector<int> &numbers, std::vector<int> &extra, int low, int high, int middle)
{
    int i = low, j = middle+1, k = low;
    for (; i<=middle && j<=high; k++)
    {
        if (numbers[i] <= numbers[j])
        {
            extra[k] = numbers[i];
            i++;
        }
        else
        {
            extra[k] = numbers[j];
            j++;
        }
    }
    
    while (i <= middle)
    {
        extra[k++] = numbers[i++];
    }

    while (j <= middle)
    {
        extra[k++] = numbers[j++];
    }

    for (int m = low; m <= high; m++)
    {
        numbers[m] = extra[m];
    }
}


void merge_sort(std::vector<int> &numbers, std::vector<int> &extra, int low, int high)
{
    if (low == high)
    {
        return ;
    }
    int middle = (low + high) / 2;
    merge_sort(numbers, extra, low, middle);
    merge_sort(numbers, extra, middle + 1, high);
    merge(numbers, extra, low, high, middle);
}

void merge_sort(std::vector<int> &numbers)
{
    // 申请额外的存储空间来用于排序处理
    std::vector<int> extra = numbers;
    merge_sort(numbers, extra, 0, numbers.size() - 1);
}

int main()
{
    std::vector<int> numbers = {49, 38, 65, 97, 76, 13, 27, 49};
    merge_sort(numbers);

    for (int i=0; i<numbers.size(); i++)
    {
        printf("%d\t", numbers[i]);
    }
    printf("\n");
}
```

## 链式存储代码

以下为采用链式存储结构的代码，本答案为我在LeetCode上的[Sort List ](https://leetcode.com/problems/sort-list/)题目的答案，源码放在[我的Github上](https://github.com/kuring/leetcode/blob/master/src/sortList/sort_list.cpp)。

```c++
struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};

/**
 * 使用归并排序方法，核心思想为将数组拆分为两半，分别对两半进行排序，排序完成后再进行一次排序，排序算法就可以采用插入排序的方式。
 * 对两半排序的算法仍然采用归并排序算法，即问题为递归问题
 * 在使用线性存储结果的归并排序算法中，会使用额外的空间来存储临时结果，空间复杂度为O(n)，而在链式存储中，空间复杂度为O(1)
 * 归并排序的时间复杂度为O(nlogn)
 * 如果存储结构为双向链表，可以使用快速排序
 */
ListNode* sortList(ListNode* head)
{
    if (head == nullptr || head->next == nullptr)
    {
        return head;
    }

    if (head->next->next == nullptr)
    {
        // 仅有两个元素，对两个元素进行排序后直接返回
        if (head->val < head->next->val)
        {
            return head;
        }
        else
        {
            ListNode *tmp = head->next;
            tmp->next = head;
            tmp->next->next = nullptr;
            return tmp;
        }
    }

    // 为了找到中间节点，这里采用快慢指针的方式，否则需要使用先遍历一次取长度，然后找到中间位置的两次遍历方式
    ListNode *fast = head;
    ListNode *slow = head;
    ListNode *slow_prev = nullptr;
    while (fast != nullptr && fast->next != nullptr)
    {
        fast = fast->next->next;
        slow_prev = slow;
        slow = slow->next;
    }
    fast = slow_prev->next;
    slow_prev->next = nullptr;

    // 分别对两段链表进行排序
    slow = sortList(head);
    fast = sortList(fast);

    // 对两段链表进行合并
    ListNode *node = nullptr, *result = nullptr;
    while (slow != nullptr && fast != nullptr)
    {
        if (slow->val < fast->val)
        {
            if (result != nullptr)
            {
                node->next = slow;
                node = node->next;
            }
            else
            {
                node = slow;
                result = slow;
            }
            slow = slow->next;
        }
        else
        {
            if (result != nullptr)
            {
                node->next = fast;
                node = node->next;
            }
            else
            {
                node = fast;
                result = fast;
            }
            fast = fast->next;
        }
    }
    if (slow != nullptr)
    {
        node->next = slow;
    }

    if (fast != nullptr)
    {
        node->next = fast;
    }

    return result;
}
```

# 直接插入排序

该排序算法的时间复杂度为O(n^2)，算法复杂度过高。分为顺序存储和链式存储两种算法，其中顺序存储每比较一个元素是从该元素往前比较的，而链式存储是从链头开始比较的，这点有所不同，造成不同的是由存储结构决定的。

## 顺序存储代码

```
#include <stdio.h>
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

void insertion_sort(std::vector<int> &numbers)
{
	if (numbers.size() <= 1)
	{
		return;
	}
	for (int i = 1; i < numbers.size(); i++)
	{
		for (int j = i; j > 0; j--)
		{
			if (numbers[j] < numbers[j - 1])
			{
				swap(numbers[j], numbers[j - 1]);
			}
			else
			{
				break;
			}
		}
	}
}

int main()
{
	int a[] = { 49, 38, 65, 97, 76, 13, 27, 49 };
	vector<int> numbers(a, a + sizeof(a) / sizeof(int));
	insertion_sort(numbers);
	for (int i = 0; i<numbers.size(); i++)
	{
		printf("%d\t", numbers[i]);
	}
	return 0;
}
```

## 链式存储代码

以下代码为LeetCode上的链式存储的情况时的直接插入排序算法的代码。

```c++
ListNode* insertionSortList(ListNode* head) 
{
    if (head == NULL)
    {
        return NULL;
    }
    
    // 初始化
    ListNode *node = head->next;
    ListNode *new_head = head;
    new_head->next = NULL;
    while (node != NULL)
    {
        ListNode *node_next = node->next;   // 先将当前遍历的下一个节点保存
        
        // 将当前节点插入到新链表中
        ListNode *new_node_tmp = new_head;
        if (node->val < new_node_tmp->val)
        {
            // 当前节点插入新链表的第一个位置
            node->next = new_head;
            new_head = node;
        }
        else
        {
            // 将当前节点插入到中间
            while (new_node_tmp->next != NULL)
            {
                if (node->val < new_node_tmp->next->val)
                {
                    node->next = new_node_tmp->next;
                    new_node_tmp->next = node;
                    break;
                }
                new_node_tmp = new_node_tmp->next;
            }
            
            // 将该节点插入到最后位置
            if (new_node_tmp->next == NULL)
            {
                new_node_tmp->next = node;
                node->next = NULL;
            }
        }
            
        // 开始遍历当前节点的下一个节点
        node = node_next;
    }
    return new_head;
}
```


