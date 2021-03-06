title: Minimum Dominating Set
date: 2015-02-03 19:50:26
tags: Algorithm
categories: Algorithm
---
#问题描述#
![](/image/20150203202449.png)
<!--more-->
![](/image/20150203202510.png)
![](/image/20150203202517.png)
![](/image/20150203202458.png)
#解题思路#
这是个覆盖集的题目，题目的意思是，能够在一个区间集合S中选出来一些个区间，这些个区间集合叫做*Minimun dominating set*, 也就是最小区域集？整个的问题也就是围绕这个定义展开的。
##a问题##
a问题呢，他定义了一个$I^*$，这个的含义就是他非要在$S$里边挑出来几个区间，组成这个集合。这个集合里边的所有区间都必须在这个Minimun dominating set当中。他让我们证明，就算在Minimun dominating set当中强行加上这个挑出来的集合里边的区域，也可以继续递归的算下去，当作另外一个求$S'$的问题。恩，我们这么说吧。他要求我们给出一个策略，让我们在求解Minimun dominating set的过程当中，这个过程可以递归进行，

接下来呢，我们的策略是这样的：把挑出来的那部分区间，也就是$I^*$,里边的所有区间元素，去掉，并且把不属于这个集合当中的其他区间，和属于这个集合的区间重合的部分修剪掉。上图：

![](/image/20150203204412.png)

上边的图中，我们的$I^*$集合中只包括那个红色的区间。我们需要修剪的就是重合的中间的部分，修剪完后是这样的。

![](/image/20150203204711.png)

剩下的这部分就是$S'$啦，我们只需要继续递归的求解剩下这部分就可以了。如果实在不明白，看b问题吧。

##b问题##
b问题是这样的，他让我们判断两件事情的正确错误，正确的话证明，错误的话找个反例。
当然是找反例简单啦，证明多麻烦！所以第一个论断肯定是错误的！（这个因果关系有点奇怪啊）
第一个论断是这样的：
>我们在$S$中选择一个最长的，然后用a问题的方法处理掉这个最长的，然后得到$S'$,然后在剩下的$S'$当中，我们继续找最长的处理，递归下去直到结束。

想法倒是挺美好。其实实在是有点问题，找个反例吧～～

![](/image/20150203205237.png)

呐，上边这个图我们看到，红色的是最长的。如果按照这个算法，我们要先选择这个红色的，然后下边连着的那两个区间就被分开了，接下来我们需要选择下排连着的两个区间以及上排右边的区间，一共选择了四个区间。
好的，正确答案肯定是三个区间就够了，中间那个红色的区间根本没用。
错误的根本在于，虽然题目的前提要求是这个选出来的区间不能是其他区间的子区间，但是如果其他的两个区间合起来包括了这个区间，也是可能出问题的。

第二个论断是这样的：

>我们在$S$中选择最左边起始区间里边最长的那个，然后用a问题的方式处理这个区间，然后继续递归下去得到结果。

我们不能找反例了，因为这个是正确的。
因为首先我们看啊，不论如何最开始的$S$区间的最左边一定是要找一个区间的。既然要找，找最大的也没什么影响，然后切割后，剩下的一样，也是要包括最左边的，找最长的那个肯定没错。空口无言，来个图证明一下吧。

![](/image/20150203204253.png)

也没啥说的，至于怎么证明。。只能说这个贪心策略是对的，。。（摊手）

##c问题##
算法设计嘛，这个最擅长了，也蛮简单的。
题目是这个样子的，$S$是一大堆区间，$[a_1,b_1][a_2,b_2][a_3,b_3]...$，这里边有个条件，$a_1 {\le} a_2 {\le} a_3...$，也就是说每个区间的起始点是单调不递减的。让我们设计一个算法，在$O(n)$的时间复杂度下找出这个Minimun dominating set。
这个还是挺简单的，而且还给了hint就更没难度了。首先我们只需要找一个标记，算是end吧，这个标记当前过去的这些节点的最后的位置。如果遍历下去发现有一个区间的结束位置比这个end靠前，说明这个区间是不用的，删掉就是了。如果这个区间的结束位置比这个end考前，那么就是结果Minimun dominating set的一部分啦。
**还有一个需要注意的问题，可能出现起始位置相同，结束位置覆盖的情况，需要在第一遍完成了之后，倒着来一遍去掉这种情况**
![](/image/20150203212252.png)
这种情况，如果黄的块第一个选到了，上边的白色的块也会选到。但是很明显下边的块是不需要的，我们需要删除这些重叠的块，这个问题的根源是$a_1 {\le} a_2$而不是$a_1<a_2$
代码在下边：


```
struct interval{					// the defination of basic struct
	int start,end;
}S[MAX];
int end = -1,start = -1,out[MAX]; 			// end means the end point of the current loop
//start used in the second loop, means the start point of the current loop
memset(out,0,sizeof(out));				// initiate the flag part
for(int i = 0;i < MAX;++ i){				// kick out the useless parts
	if(S[i].end <= end) out[i] = 1;
	else end = S[i].end;
}
for(int i = MAX - 1,i >= 0; --i){
	if(!out[i]){
		if(S[i].start == start) out[i] = 1;	// kick out the useless parts
		start = S[i].start;
	}
}
```

最后out里边为0的结果就是选择到的Minimun dominating set的元素啦。

算法基于b问题的第二种方法，这是一种正确的方法。因为每个区间的开始位置是不递减的，


