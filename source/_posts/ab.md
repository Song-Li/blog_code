title: a+b
date: 2014-12-13 15:40:12
tags: Algorithm
categories: Algorithm_Road to Success
---
开宗明义a+b
<!--more-->
##a+b##
时间限制: 1秒  内存限制: 64M
###Problem Description###
Now let’s calculate the answer of a + b ~
###Input###
The input will consist of a set of pairs of integers for a and b(-10^1000 <= a, b <= 10^1000). The input is ended by EOF.
###Output###
For each test case you should output the answer of a + b.
###Sample Input###
```
1 1
1 -1
```
###Sample Output###
```
2
0
```
##解答

开宗明义啊，a+b肯定是第一道题。但是这个肯定不是传统意义上的printf("%d\n",a+b);这种啦，这里绝壁是一个大数加法。想来写个大数加法也不是什么难事。
这里有一个要介绍一下的关键点，就是所谓的十进制补码。减去一个数等价于加上这个数的十进制补码啦，下边的这个change函数就是用来干这件事情的
关于补码，我有话要说：
从刚上大学开始老师就教我们，二进制负数的补码怎么求怎么求，从来没人说过到底是为什么。补码是什么呢？正数的补码是本身，至于负数，先看一个公式：
> (a + -a) % p = 0
> $p = base ^ {(n - 1)}$

**这里注意base是进制数，n是机器位数**
这两个公式其实就告诉了我们什么是补码。补码其实就是机器取机器最大表示数的余数。
这里我们就可以解释一下为啥减去一个数等于加上这个数的补码，这里有两点
**1,减去一个数可以认为是加上这个数的负数形式。
2,加上这个数的负数形式在模余概念上等于加上这个数的补码。**
举个例子吧，250-123这个例子。这里呢，250-123 = 250 + (-123)。如果机器最大容纳1000，那么就是
>(250 + (-123)) % 1000 = (250 + 1000 + (-123)) % 1000 = (250 + 877) % 1000;

这里877是123的补码。所以大概知道为什么了吧。
下边的代码就是检测如果输入了负数，变成补码，最后统一处理加法就好了。写的很容易懂。
**代码是大一写的，看起来没啥问题，这种题目就不重写了，凑合看吧**
```
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<math.h>
#define MAX 1024

void change(int *a)//转换十进制补码
{
	int i,x = 1;
	for(i = MAX - 1;i > 0;i --)
	{
		a[i] = 9 - a[i];
		a[i] += x;
		x = a[i] > 9;
		if(a[i] > 9) a[i] -= 10;
	}
}

int input(int *a) //input 的部分啦，
{
	int len,i,j = MAX;
	char tem[MAX];
	mem(a,0);
	mem(tem,0);
	if(scanf("%s",tem) == EOF) return 0;
	len = strlen(tem);
	for(i = len - 1;i >= 0 && tem[i] != '-';i --)
	{
		a[-- j] = tem[i] - '0';
	}
	if(tem[0] == '-') change(a);//如果是负数的话就变一下形式。这里用了十进制补码
	return 1;
}

void HPplus(int *a,int *b,int *res)
{
	int i,x = 0;
	mem(res,0);
	for(i = MAX - 1;i > 0;i --)
	{
		res[i] = a[i] + b[i] + x;
		x = (res[i] >= 10);
		if(res[i] >= 10) res[i] -= 10;
	}
}

void output(int *res)
{
	int i,flag = 0;
	if(res[1] == 9)
	{
		change(res);
		printf("-");
	}
	for(i = 1;i < MAX && res[i] == 0;i ++);
	for(;i < MAX;i ++)
	{
		printf("%d",res[i]);
		flag = 1;
	}
	if(!flag) printf("0");
	printf("\n");
}

int main()
{
	int a[MAX],b[MAX];
	int res[MAX];
	while(input(a) && input(b))
	{
		HPplus(a,b,res);
		output(res);
	}
}
```