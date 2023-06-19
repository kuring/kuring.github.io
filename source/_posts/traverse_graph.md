---
title: 图的存储和遍历
Status: public
url: traverse_graph
tags: 算法
date: 2014-09-28
---

# 图的存储

在leetcode中图的存储形式如下，这种形式的图只能适合用来存储是连通图的情况，且根据leetcode提供的_{0,1,2#1,2#2,2}_格式的字符串通过程序来自动构造图比较麻烦，预知字符串的含义请移步到[leetcode的解释](https://oj.leetcode.com/problems/clone-graph/)。

```c
/**
 * Definition for undirected graph.
 * struct UndirectedGraphNode {
 *     int label;
 *     vector<UndirectedGraphNode *> neighbors;
 *     UndirectedGraphNode(int x) : label(x) {};
 * };
```

本文为了能够用字符串表示所有图，并且便于程序的构造，使用了邻接表的形式来对图进行存储，即可以用来存储有向图，有可以存储无向图。图一个节点的结构如下：

```c
/**
 * 图节点的邻接表表示形式
 */
struct GraphNode {
    std::string label;
    std::vector<GraphNode *> neighbors;
    bool visited;			// 深度优先搜索和广度优先搜索的遍历都需要visited数组，为了简化程序，直接在节点的存储结构中设置visited变量
    GraphNode(std::string x) : label(x), visited(false) {};
};
```

图的创建方面为了简化算法实现，对程序的效率没做太多关注，算法复杂度稍高。本算法的难点在于对字符串的拆解，并根据字符串找到对应的节点指针。

```c++
/**
 * 图节点的邻接表表示形式
 */
struct GraphNode {
    std::string label;
    std::vector<GraphNode *> neighbors;
    bool visited;
    GraphNode(std::string x) : label(x), visited(false) {};
};

/**
 * 通过字符串的值，找到该字符串对应的图节点
 */ 
GraphNode *get_one_node(const std::vector<GraphNode *> &node_vector, std::string str)
{
    for (std::vector<GraphNode *>::const_iterator iter = node_vector.begin(); iter != node_vector.end(); iter++)
    {
        if ((*iter)->label == str)
        {
            return *iter;
        }
    }
    return NULL;
}

/**
 * 时间复杂度高,对图的构建一般效率要求较低
 * 对于查找某个节点的邻接点的指针操作可以使用map来提高查询效率
 * 或者可以通过不需要初始化所有节点的方式来构造图，而是采用需要哪个节点构造哪个节点的方式
 */
std::vector<GraphNode *> create_graph(std::string str)
{
    std::vector<GraphNode *> node_vector;
   
    // init all nodes
    for (size_t pos = 0; pos < str.length() - 1;)
    {
        size_t end = str.find(',', pos);
        if (end != std::string::npos)
        {
            GraphNode *node = new GraphNode(str.substr(pos, end - pos));
            node_vector.push_back(node);
        }
        else
        {
            break;
        }

        pos = str.find('#', pos);
        if (pos == std::string::npos)
        {
            break;
        }
        else
        {
            pos++;
        }
    }

    // add neighbors in every node
    for (size_t pos = 0; pos < str.length() - 1; )
    {
        GraphNode *current_node = NULL;
        size_t current_end = str.find(',', pos);
        if (current_end != std::string::npos)
        {
            current_node = get_one_node(node_vector, str.substr(pos, current_end - pos));
            pos = current_end + 1;
        }
        else
        {
            break;
        }

        size_t node_end = str.find('#', pos);   // 当前节点的字符串的结束位置
        if (node_end == std::string::npos)
        {
            node_end = str.length();
        }
        else
        {
            node_end--;
        }

        for ( ; ; )
        {
            current_end = str.find(',', pos);
            if (current_end > node_end || current_end == std::string::npos)
            {
                // 一个节点的最后一个邻接点
                current_end = node_end;
            }
            else 
            {
                current_end--;
            }

            GraphNode *node = get_one_node(node_vector, str.substr(pos, current_end - pos + 1)); 
            if (node != NULL)
            {
                current_node->neighbors.push_back(node);
                std::cout << current_node->label << " add " << node->label << std::endl;
            }
            if (current_end == node_end)
            {
                // 一个节点的最后一个邻接点
                break;
            }
            else
            {
                pos = current_end + 2;  // 该节点之后还有其他邻接点
            }
        }
        pos = node_end + 2;
    }
    return node_vector;
}
```

# 深度优先搜索

深度优先搜索遵循贪心算法的原理，如果孩子节点不为空，则一直遍历下去。

## 递归算法


```c++
void DFS_traverse_recursion(GraphNode *node)
{
    if (!node->visited)
    {
        std::cout << node->label << '\t';
        node->visited = true;
    }
    for (std::vector<GraphNode *>::iterator iter = node->neighbors.begin(); iter != node->neighbors.end(); iter++)
    {
        if (!(*iter)->visited)
        {
            DFS_traverse_recursion(*iter);
        }
    }
}

/**
 * 图的深度优先搜索的递归形式
 */
void DFS_traverse_recursion(std::vector<GraphNode *> &graph)
{
    for (std::vector<GraphNode *>::iterator iter = graph.begin(); iter != graph.end(); iter++)
    {
        (*iter)->visited = false;
    }
    for (std::vector<GraphNode *>::iterator iter = graph.begin(); iter != graph.end(); iter++)
    {
        if (!(*iter)->visited)
            DFS_traverse_recursion(*iter);
    }
}
```
## 非递归算法

使用栈来实现非递归。

```c++
/**
 * 图的深度优先搜索的非递归形式
 */
void DFS_traverse_not_recursion(std::vector<GraphNode *> &graph)
{
    for (std::vector<GraphNode *>::iterator iter = graph.begin(); iter != graph.end(); iter++)
    {
        (*iter)->visited = false;
    }

    for (std::vector<GraphNode *>::iterator iter = graph.begin(); iter != graph.end(); iter++)
    {
        std::stack<GraphNode *> node_stack;
        if ((*iter)->visited)
        {
            continue;
        }
        node_stack.push(*iter);
        while (!node_stack.empty())
        {
            GraphNode *node = node_stack.top();
            node_stack.pop();
            if (node->visited)
                continue;
            std::cout << node->label << '\t';
            node->visited = true;
            /* 使用反向迭代器遍历后将节点加入到栈中 */
            for (std::vector<GraphNode *>::reverse_iterator iter2 = node->neighbors.rbegin(); iter2 != node->neighbors.rend(); iter2++)
            {
                if (!(*iter2)->visited)
                {
                    node_stack.push(*iter2);
                }
            }
        }
    }
}
```

# 广度优先搜索

该算法不存在递归算法，仅有非递归版本。需要利用队列来保存需要遍历的节点，占用的存储空间稍多。

```c++
/**
 * 图的广度优先搜索的非递归形式
 */
void BFS_traverse_not_recursion(std::vector<GraphNode *> &graph)
{
    for (std::vector<GraphNode *>::iterator iter = graph.begin(); iter != graph.end(); iter++)
    {
        (*iter)->visited = false;
    }

    for (std::vector<GraphNode *>::iterator iter = graph.begin(); iter != graph.end(); iter++)
    {
        std::queue<GraphNode *> node_queue;
        if ((*iter)->visited)
        {
            continue;
        }
        node_queue.push(*iter);
        while (!node_queue.empty())
        {
            GraphNode *node = node_queue.front();
            node_queue.pop();
            if (node->visited)
                continue;
            std::cout << node->label << '\t';
            node->visited = true;
            for (std::vector<GraphNode *>::iterator iter2 = node->neighbors.begin(); iter2 != node->neighbors.end(); iter2++)
            {
                if (!(*iter2)->visited)
                {
                    node_queue.push(*iter2);
                }
            }
        }
    }
}
```

# 相关下载

[本文相关源码](http://pan.baidu.com/s/1i3GIj9V)
