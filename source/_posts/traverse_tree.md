---
title: 二叉树的遍历
Status: public
url: traverse_tree
tags: 算法
date: 2014-09-28
---

[TOC]

近期准备复习数据结构和算法的知识，参照了网络上各路大神的学习攻略，大部分算法学习的思路为参照一些经典书籍（如算法导论）并结合一些代码的实践来完成，并未找到一条适合我的算法学习之路。经过思考后决定采用代码编写曾经的教科书中代码实例的方式来学习，曾经接触的算法教科书包括《数据结构（C语言版）》和《计算机算法基础》。一来这些算法已经基本熟悉，只是时间久远有些已经忘记；二来，通过思考后编写代码增强自己的记忆。

同时我编写的这些实例可以作为leetcode上的很多题目的基础，为下一个阶段刷leetcode上的题目打好基础。

树的存储形式包括了顺序存储（采用数组形式）和链式存储，其中链式存储更为灵活，可以表示任意形式的树，本文中的代码将采用树的链式存储方式。

# 树的构建

树的构建有多种方式，本文使用字符串采用了自顶向下、自左到右的顺序构建树，跟leetcode的形式一致。其中'#'表示该节点为空，如果该节点为空节点，其左右子孩子节点也要用'#'表示，而不能不用任何字符表示。

```c++
/**
 * 二叉树的链式存储结构
 */
struct TreeNode
{
    char data;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(char data) : data(data), left(NULL), right(NULL) {}
};

/**
 * 构造二叉树，要构造的字符串采用了自顶向下、自左到右的顺序，跟leetcode的形式一致
 */
TreeNode *create_binary_tree(const char *str)
{
    if (!str || strlen(str) == 0)
    {
        return NULL;
    }

    // 对每个节点分配存储空间
    int node_size = strlen(str);
    TreeNode **tree = new TreeNode*[node_size];
    for (int i=0; i<node_size; i++)
    {
        if (str[i] == '#')
        {
            tree[i] = NULL;
        }
        else
        {
            tree[i] = new TreeNode(str[i]);
        }
    }

    for (int i=0, j=0; i<node_size && j<node_size; i++)
    {
        if (tree[i] != NULL)
        {
            if ((j + 1) < node_size)
            {
                tree[i]->left = tree[++j];
                tree[i]->right = tree[++j];
            }
        }
        else
        {
            j += 2;
        }
    }
    return *tree;
}
```

# 二叉树的先序遍历

## 递归实现

```
/**
 * 先序遍历二叉树的递归形式
 */
void preorder_traverse_recursion(TreeNode *root)
{
    if (!root)
    {
        return;
    }
    
    printf("%c\t", root->data);
    
    preorder_traverse_recursion(root->left);

    preorder_traverse_recursion(root->right);
}
```

## 非递归实现

```c++
/**
 * 先序遍历的非递归形式
 */
void preorder_traverse_not_recursion(TreeNode *root)
{
    if (!root)
    {
        return ;
    }
    std::stack<TreeNode *> tree_stack;
    tree_stack.push(root);
    while (tree_stack.size() > 0)
    {
        TreeNode *node = tree_stack.top();
        tree_stack.pop();
        printf("%c\t", node->data);
        if (node->right != NULL)
        {
            tree_stack.push(node->right);   
        }
        if (node->left != NULL)
        {
            tree_stack.push(node->left);
        }
    }
}
```

# 二叉树的中序遍历

## 递归实现

```c++
/**
 * 中序遍历二叉树的递归形式
 */
void inorder_traverse_recursion(TreeNode *root)
{
    if (!root)
    {
        return;
    }
    
    inorder_traverse_recursion(root->left);
    
    printf("%c\t", root->data);

    inorder_traverse_recursion(root->right);
}
```

## 非递归实现

```c++
/**
 * 中序遍历二叉树的非递归形式
 */
void inorder_traverse_not_recursion(TreeNode *root)
{
    if (!root)
    {
        return ;
    }
    std::stack<TreeNode *> tree_stack;
    while (root != NULL || tree_stack.size() > 0)
    {
        // 遍历到左子树的叶子节点
        while (root)
        {
            tree_stack.push(root);
            root = root->left;
        }

        // 遍历栈顶节点
        root = tree_stack.top();
        tree_stack.pop();
        printf("%c\t", root->data);

        root = root->right;
    }
}
```

# 二叉树的后序遍历

## 递归实现

```c++
/**
 * 后序遍历二叉树的递归形式
 */
void postorder_traverse_recursion(TreeNode *root)
{
    if (!root)
    {
        return;
    }
    
    postorder_traverse_recursion(root->left);
    
    postorder_traverse_recursion(root->right);

    printf("%c\t", root->data);
}
```

## 非递归实现

仅用一个栈不能够实现后序遍历非递归算法，需要保存一个上次访问过节点的变量。

```c++
/**
 * 后序遍历二叉树的非递归形式
 */
void postorder_traverse_not_recursion(TreeNode *root)
{
    if (!root)
    {
        return ;
    }
    std::stack<TreeNode *> tree_stack;
    TreeNode *visited = NULL;
    while (root != NULL || tree_stack.size() > 0)
    {
        // 遍历到左子树的叶子节点
        while (root)
        {
            tree_stack.push(root);
            root = root->left;
        }

        root = tree_stack.top();

        if (root->right == NULL || root->right == visited)
        {
            // 如果没有右孩子，或者右孩子刚刚访问过，则访问当前节点
            printf("%c\t", root->data);
            tree_stack.pop();
            visited = root;
            root = NULL;
        }
        else
        {
            root = root->right;
        }
    }
}
```

# 总结

二叉树遍历的递归形式程序结构类似，编写相对简单。但是递归方法在C语言中存在执行效率差（需要维护函数栈），容易出现栈溢出的异常的问题。任何递归问题问题都可以转化为非递归问题，转化的思路包括了直接转化法和间接转化法。直接转化法可以通过循环来解决，间接转化法需要借助栈加循环来解决。

二叉树遍历的非递归形式相对复杂，二叉树的先序遍历的非递归形式容易理解，二叉树的中序遍历稍微困难，后序遍历的非递归形式最复杂。

# 相关下载

[程序源代码](http://pan.baidu.com/s/1eQmweSM)
