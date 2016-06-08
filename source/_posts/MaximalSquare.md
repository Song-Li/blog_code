title: "Maximal Square"
date: 2015-06-07 16:08:25
tags: leetcode
---
Given a 2D binary matrix filled with 0's and 1's, find the largest square containing all 1's and return its area.

For example, given the following matrix:

1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
Return 4

<!--more-->
##本题解法##
简单的DP。PS:我也不知道有没有别的更好的做法，反正第一反应就是dp，也就dp过了。就这样吧
我们最终要求的是方块全是1的最大的方块，设计三个dp的部分，分别是表示当前格子的最大方块，当前格子竖直方向连续的1的个数，当前格子水平方向连续1的个数。
因为限制某个格子左上角最大方块的只有这个点左上角的点有多大，上边的点竖直方向上连续的1的个数够不够扩展，左边的点水平方向上的1的个数够不够扩展，三个要素找一个最小的就是可扩展的边长了。
这样就可以看到，某个位置dps[i][j] = min(dph[i][j - 1],dpv[i - 1][j],dps[i - 1][j - 1]);
dph[i][j - 1](当前格子左边的一个格子水平连续1的个数)，dpv[i - 1][j](当前格子上边的一个格子竖直连续1的个数)，dps[i - 1][j - 1]当前格子左上角的最大方块的边长
##实现代码##
```
int min(int a,int b,int c){
    return (a<((b<c)?b:c)?a:((b<c)?b:c));
}
int max(int a,int b){
    if(a > b) return a;
    return b;
}
#define size 1000
int dps[size][size];
int dph[size][size];
int dpv[size][size];
int maximalSquare(char** matrix, int matrixRowSize, int matrixColSize) {
    if(matrixRowSize == 0 || matrixColSize == 0) return 0;
    int maxnum = 0;
    int temp;
    if(matrix[0][0] == '1') temp = 1;
    else temp = 0;
    dps[0][0] = temp;
    dph[0][0] = temp;
    dpv[0][0] = temp;
    maxnum = max(maxnum,temp);
    for(int i = 1;i < matrixColSize;++ i){//初始化水平方向第一行
        if(matrix[0][i] == '1'){
            dps[0][i] = 1;
            dph[0][i] = dph[0][i - 1] + 1;
            dpv[0][i] = 1;
        }else{
            dps[0][i] = dpv[0][i] = 0;
            dph[0][i] = 0;
        }
        maxnum = max(maxnum,dps[0][i]);
    }

    for(int i = 1;i < matrixRowSize;++ i){//初始化竖直方向第一行
        if(matrix[i][0] == '1'){
            dps[i][0] = 1;
            dph[i][0] = 1;
            dpv[i][0] = dpv[i - 1][0] + 1;
        }else{
            dps[i][0] = 0;
            dpv[i][0] = dph[i][0] = 0;
        }
        maxnum = max(maxnum,dps[i][0]);
    }
    for(int i = 1;i < matrixRowSize; ++i){//开始dp的过程
        for(int j = 1;j < matrixColSize;++ j){
            if(matrix[i][j] == '1') {//如果这个格子是1，就可以扩展
                dps[i][j] = min(dps[i - 1][j - 1],dpv[i - 1][j],dph[i][j - 1]) + 1;
                dph[i][j] = dph[i][j - 1] + 1;
                dpv[i][j] = dpv[i - 1][j] + 1;
            }else{//不是1就清零
                dps[i][j] = 0;
                dph[i][j] = 0;
                dpv[i][j] = 0;
            }
            maxnum = max(maxnum,dps[i][j]);
        }
    }
    return maxnum * maxnum;
}
```