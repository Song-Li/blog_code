title: "Count Complete Tree Nodes"
date: 2015-06-07 15:39:28
tags: leetcode
---
Given a complete binary tree, count the number of nodes.
<!--more-->
水题嘛~给一颗完全二叉树（误？），让统计一下一共有多少节点。
这个题目不知道暴力遍历会不会过。按照leetcode的尿性是没问题的吧~只不过要快点的话还是二分
##二叉树的问题##
总之一个满二叉树，总会有一些奇怪的性质。比方说下边这个图的标号：
![完全二叉树](/image/20150607154352.png)
这个图就是个完全二叉树，我们可以看到什么呢？
**父子节点标号关系**
首先来说，一个节点的标号的左右孩子节点的标号是多少呢？如果这个节点是x,左孩子就是x * 2,右孩子就是x * 2 + 1。所以如果我们知道一个节点的标号，直接除以2就是父节点标号了。
那么如果一个节点除以4呢？就是爷爷节点的标号了。除以$2^n$呢？就是第n代祖先的标号了啦。
**左节点还是右节点**
一个节点的标号如果是奇数，就是右孩子，偶数的话，就是左孩子=。=用简单的$x &１$就可以得到结果了
##本题解法##
首先我们统计一下有多少层，得到大致的数据范围嘞。
然后二分这些范围，看看二分到这个点存不存在，存在的话就往右找，不存在的话就往左找。
**ps:这里的二分有点诡异，写的时候需要注意一下。细心点就能过。**
代码如下：
```
int power(int a,int n)//快速幂，后边用的不少，干脆自己写一个
{
    int res = 1;
    while(n)
    {
        if(n & 1) res *= a;
        a *= a;
        n >>= 1;
    }
    return res;
}

bool exist(struct TreeNode *root, int level, int mid)//看看是不是存在某个标号的节点
{
    int now;
    struct TreeNode *tnode = root;
    while(level --)
    {
        if((mid / power(2,level)) & 1) tnode = tnode->right;
        else tnode = tnode->left;
    }
    if(tnode == NULL) return 0;
    return 1;
}

int countNodes(struct TreeNode* root)//就是统计节点的函数啦
{
    int level = 0;
    struct TreeNode *tnode = root;
    if(root == NULL) return 0;
    for(level = 0;; ++ level)
    {
        tnode = tnode->left;
        if(tnode == NULL) break;
    }
    int left = power(2,level);
    int right = power(2,level + 1) - 1;
    int mid;
    while(left < right - 1)
    {
        mid = (left + right) >> 1;
        if(exist(root,level,mid)) left = mid;
        else right = mid - 1;
    }
    if(exist(root,level,right)) return right;
    return left;
}
```