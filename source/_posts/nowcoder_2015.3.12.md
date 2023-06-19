---
title: 牛客网内推笔试卷题目2015.3.12
Status: public
url: nowcoder_2015.3.12
tags: 算法
date: 2015-03-25
toc: yes
---

前段时间参加了牛客网的答题活动，共两套试题，每套题目3个算法题，我只做了每套题的前两道。最近想查看之前做的题目的答案，却发现非常不方便，特此将我做过的4道题目记录一下，算法的思路就不再解释了。

# 题目一 奇数位上都是奇数或者偶数位上都是偶数

> 给定一个长度不小于2的数组arr。 写一个函数调整arr，使arr中要么所有的偶数位上都是偶数，要么所有的奇数位上都是奇数上。 要求：如果数组长度为N，时间复杂度请达到O(N)，额外空间复杂度请达到O(1),下标0,2,4,6...算作偶数位,下标1,3,5,7...算作奇数位，例如[1,2,3,4]调整为[2,1,4,3]即可。

```c++
class Solution {
public:
    void oddInOddEvenInEven(vector<int>& arr, int len) {
        int odd = 1;
        int even = 0;
        while (odd < len && even < len)
        {
            if (arr[odd] % 2 == 0)
            {
                while (arr[even] % 2 == 0)
                {
                    even += 2;
                }
                if (even < len)
                {
                    int tmp = arr[even];
                    arr[even] = arr[odd];
                    arr[odd] = tmp;
                }
                else
                {
                    break;
                }
            }
            else
            {
                odd += 2;
            }
        }
    }
};
```

# 题目二 求正数数组的最小不可组成和

> 给定一个全是正数的数组arr，定义一下arr的最小不可组成和的概念： 1，arr的所有非空子集中，把每个子集内的所有元素加起来会出现很多的值，其中最小的记为min，最大的记为max； 2，在区间[min,max]上，如果有一些正数不可以被arr某一个子集相加得到，那么这些正数中最小的那个，就是arr的最小不可组成和； 3，在区间[min,max]上，如果所有的数都可以被arr的某一个子集相加得到，那么max+1是arr的最小不可组成和； 举例： arr = {3,2,5} arr的min为2，max为10，在区间[2,10]上，4是不能被任何一个子集相加得到的值中最小的，所以4是arr的最小不可组成和； arr = {3,2,4} arr的min为2，max为9，在区间[2,9]上，8是不能被任何一个子集相加得到的值中最小的，所以8是arr的最小不可组成和； arr = {3,1,2} arr的min为1，max为6，在区间[2,6]上，任何数都可以被某一个子集相加得到，所以7是arr的最小不可组成和； 请写函数返回arr的最小不可组成和。

```c++
class Solution {
public:
    int getFirstUnFormedNum(vector<int>& arr, int len) {
        set<int> res;
        for (int i=0; i<len; i++)
        {
            set<int> tmp = res;
            for (set<int>::iterator iter = res.begin(); iter != res.end(); iter++)
            {
                tmp.insert(*iter + arr[i]);
            }
            res = tmp;
            res.insert(arr[i]);
        }

        set<int>::iterator iter = res.begin();
        int before = *iter;
        iter++;
        for (; iter != res.end(); iter++)
        {
            if (*iter - before > 1)
            {
                return before + 1;
            }
            before = *iter;
        }
        return before + 1;
    }
};
```

# 题目三 最大的LeftMax与rightMax之差绝对值

> 给定一个长度为N的整型数组arr，可以划分成左右两个部分： 左部分arr[0..K]，右部分arr[K+1..arr.length-1]，K可以取值的范围是[0,arr.length-2] 求这么多划分方案中，左部分中的最大值减去右部分最大值的绝对值，最大是多少？ 例如： [2,7,3,1,1] 当左部分为[2,7]，右部分为[3,1,1]时，左部分中的最大值减去右部分最大值的绝对值为4; 当左部分为[2,7,3]，右部分为[1,1]时，左部分中的最大值减去右部分最大值的绝对值为6; 最后返回的结果为6。 注意：如果数组的长度为N，请尽量做到时间复杂度O(N)，额外空间复杂度O(1)

```c++
class Solution {
public:
    int getMaxABSLeftAndRight(vector<int> vec, int len) {
        if (len == 0)
        {
            return 0;
        }

        // find the max in array
        int max = vec[0];
        for (int i=1; i<(int)vec.size(); i++)
        {
            if (vec[i] > max)
            {
                max = vec[i];
            }
        }

        // compare the head and tail in array
        if (vec[0] < vec[len - 1])
        {
            return max - vec[0];
        }
        return max - vec[len - 1];
    }
};

```

# 题目四 按照左右半区的方式重新组合单链表

> 给定一个单链表的头部节点head，链表长度为N。 如果N为偶数，那么前N/2个节点算作左半区，后N/2个节点算作右半区； 如果N为奇数，那么前N/2个节点算作左半区，后N/2+1个节点算作右半区； 左半区从左到右依次记为L1->L2->...，右半区从左到右依次记为R1->R2->...。请将单链表调整成L1->R1->L2->R2->...的样子。 例如： 1->2->3->4 调整后：1->3->2->4 1->2->3->4->5 调整后：1->3->2->4->5 要求：如果链表长度为N，时间复杂度请达到O(N)，额外空间复杂度请达到O(1)

```c++
class Solution {
public:
    void relocateList(struct ListNode* head) {
        if (head == NULL || head->next == NULL)
        {
            return ;
        }

        // use one loop, find the right head
        ListNode *right_head = head;
        ListNode *node = head;
        while (node != NULL)
        {
            if (node->next == NULL)
            {
                break;
            }
            if (node->next->next == NULL)
            {
                right_head = right_head->next;
                break;
            }
            right_head = right_head->next;
            node = node->next->next;
        }

        ListNode *left_node = head;
        ListNode *right_node = right_head;
        while (left_node->next != right_head)
        {
            ListNode *tmp = left_node->next;
            left_node->next = right_node;
            right_node = right_node->next;
            left_node->next->next = tmp;
            left_node = left_node->next->next;
        }

        left_node->next = right_node;
    }
};
```
