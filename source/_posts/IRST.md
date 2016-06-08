title: IRST
date: 2015-02-09 00:46:28
tags: Algorithm
categories: Algorithm
---
##开篇##
IRST,互关联后继树，是国内胡运发教授根据序列字符的有序性和冗余性提出的一种新型的**海量全文存储,索引**模型。因为刚刚写了一个关于这个的东西，暂且学习学习这是个什么东西吧。
<!--more-->
##说明##
前驱后继的概念不解释了。重点是我觉得这种树很有意思。理论什么的一句都不说，直接上例子。
![](/image/20150209005007.png)

大体解释一下这个例子吧。全文是abcabaabc，首先从a开始，下边走第一个分支是(b,1)，表示下边一个字符是b开头的树的第一个分支，我们来到根为b的这个树，看到第一个分支是(c,1)，说明下一个是c为根的第一个分支，所以就是走到c树，看到第一个分支是(a,2)，说明下一个字符是a的第二个分支，然后我们来到了a的第二个分支，发现是(b,2)，说明下一个是b的第二个分支。以此类推。

这么做有什么好处呢？我们来看看这个树首先干了什么吧。所谓**互关联后继树**，他就是一颗后继树。说白了呢就是标明了每个字符位置的后继位置。用二元组的方式，标出了后继字符在这个字符的第几次出现。假如说是a为根的树，下边有一个(b,2)，就说明现在这个位置，下一个字符是b，而且这个b是全文第二个b。

这样子下来呢，就很容易找到某个字符一共出现了多少次（树的叶子节点的大小），以及后继结点的标号以及出现在什么位置。甚至进一步的运用，可以涉及到识别字串出现的次数。

这里先说一下具体生成这棵树的方法。

![](/image/20150209005837.png)
![](/image/20150209005849.png)

具体代码实现的时候会有点变化。因为我在写代码的时候发现先做第四步，在前驱节点树的节点添加会更快捷方便。所以我的代码写成了这样。

```
while(!input.eof()){
        input >> N[i].length >> N[i].data;
        //添加到上一个节点树
        if(i){
            temp = get(N[i - 1].data);
            SI[temp].child[SI[temp].child.size() - 1].data = N[i].data;
            if(get(N[i].data) != -1) SI[temp].child[SI[temp].child.size() - 1].pos = SI[get(N[i].data)].child.size();
            else SI[temp].child[SI[temp].child.size() - 1].pos = 0;
        }
        // 添加本节点树
        if(get(N[i].data) == -1){
            TSI.child.clear();
            TSI.id = N[i].data;
            SI[cnt ++] = TSI;
        }

        temp = get(N[i].data);
        TNo.length = N[i].length;
        TNo.data = -1;
        TNo.pos = -1;
        SI[temp].child.push_back(TNo);
        ++ i;
}
```
这个代码呐，一会会放出能运行的完整版本。先凑合看吧。

##具体应用##
这里重点记录一个应用，就是说寻找所谓的频繁因子吧。别管这个因子是什么。能寻找出频繁存在的串。

这种东西，借鉴一下一个概念，所谓SIRST，**时序互关联后继树**，听起来如何的高大上，其实道理很简单。就是在IRST的节点二元组(a,1)类似的东西前边，加上一个属于这个二元组的label，我们可以计算出这个label出现的次数大约多少次的串等等。

好处是每个串只需要计算一次。因为我们这样子很轻松的能统计出来串出现的次数。比方说刚刚的例子，我们要统计一下ab这个串出现的次数，只需要从a开始，计算一下孩子结点中b的个数就行了。另外如果要求abc这个串出现的次数，那我们就从a开始，递归的求下去就行了。这里不多说了，从a开始，找到孩子中有b的叶子，假如说是(b,1)(b,2),我们接着去b树里边找1，2分支的，如果有c就继续递归就好了。这样统计字串个数倒是挺简单的。

这里我写的示例程序，主要是解决找label出现次数多余Fmin的子串。不断的递归寻找，最后输出找到的子串以及对应的次数。

目前发现这个奇怪的树的作用也就是找子串出现次数很有意思。

```
#include <stdio.h>
#include <vector>
#include <iostream>
#include <string.h>
#include <fstream>
#include <stdlib.h>

#define MAX 100
int Fmin = 2;

using namespace std;
int cnt = 0;


struct Node{
    int length;
    int data;
    int pos;
};

struct SIRST{
    int id;
    vector<Node> child;
};

SIRST SI[MAX];

int get(int data){
    for(int i = 0;i < cnt;++ i){
        if(SI[i].id == data) return i;
    }
    return -1;
}

string getstring(int n){
    string temp = "",temp1 = "";
    while(n){
        temp1 += (n % 10) + '0';
        n /= 10;
    }
    for(int i = temp1.length() - 1;i >= 0;-- i){
        temp += temp1[i];
    }
    return temp;
}

void getres(string now, vector<int> it, int root){
    int num_l[MAX];
    memset(num_l,0,sizeof(num_l));
    vector<int> next[MAX];
    string newnow = "";
    int flag = 0;
    for(int i = 0;i < it.size();++ i){
        num_l[SI[root].child[it[i]].length] ++;
    }
    for(int i = 0;i < MAX - 10 ; ++ i){
        if(num_l[i] && (num_l[i] >= Fmin)){
           newnow = now + "(";
           newnow += getstring(i) + ',';
           newnow += getstring(SI[root].id) + ')';
           cout << newnow << "  " << num_l[i] << endl;
            for(int j = 0;j < it.size();++ j){
                int temp = it[j];
                if(SI[root].child[temp].length == i){
                    next[get(SI[root].child[temp].data)].push_back(SI[root].child[temp].pos);
                    //cout << "pos" << SI[root].child[temp].pos << endl;
                }
            }
            for(int j = 0;j < cnt;++ j){
                if(next[j].size() >= Fmin){
                    getres(newnow, next[j], j);
                }
                next[j].clear();
            }
        }
    }
    return ;
}


int main(){

    Node N[1000],TNo;
    SIRST TSI;
    int temp;
    int temp1;
    printf("Please input the Fmin\n");
    scanf("%d",&Fmin);
    ifstream input;
    input.open("in.txt");
    //freopen("out.txt","w",stdout);
    int i = 0;

    //算法一

    while(!input.eof()){
        input >> N[i].length >> N[i].data;
        //添加到上一个节点树
        if(i){
            temp = get(N[i - 1].data);
            SI[temp].child[SI[temp].child.size() - 1].data = N[i].data;
            if(get(N[i].data) != -1) SI[temp].child[SI[temp].child.size() - 1].pos = SI[get(N[i].data)].child.size();
            else SI[temp].child[SI[temp].child.size() - 1].pos = 0;
        }
        // 添加本节点树
        if(get(N[i].data) == -1){
            TSI.child.clear();
            TSI.id = N[i].data;
            SI[cnt ++] = TSI;
        }

        temp = get(N[i].data);
        TNo.length = N[i].length;
        TNo.data = -1;
        TNo.pos = -1;
        SI[temp].child.push_back(TNo);
        ++ i;
    }

    for(int i = 0;i < cnt;++ i){
        printf("%d\n",SI[i].id);
        for(int j = 0;j < SI[i].child.size();++ j){
            printf("(%d,%d,%d) ",SI[i].child[j].length,SI[i].child[j].data,SI[i].child[j].pos == -1 ? -1 : (SI[i].child[j].pos + 1));
        }
        printf("\n");
    }


    //算法二
    vector<int> VTemp;
    string STemp = "";
    for(int i = 0;i < cnt;++ i){
        VTemp.clear();
        for(int j = 0;j < SI[i].child.size();++ j){
            VTemp.push_back(j);
        }
        getres(STemp,VTemp,i);
    }


    return 0;
}
```
如有问题，欢迎提出。