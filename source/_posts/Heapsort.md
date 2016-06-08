title: "Heapsort"
date: 2015-10-17 13:36:32
tags: Algorithm
---
Heap sort? What? You choose this?
I don't know why I choose heapsort. Maybe I just want to practise how to write something about heap.
If you want to know how it works, go to Wiki, or Baidu, or Google. 
It's just my implement. 
We can see that heapsort may have more actions compared with quicksort. But heapsort is always $O(n * log_2n)$, while quicksort may cause a $O(n^2)$. Especially I'm always in bad luck. T.T
Here is my code:
<!--more-->
```
#include <stdio.h>
#include <string.h>
#define MAX 10000
void swap(int *a,int *b){
    int tmp = *b;
    *b = *a;
    *a = tmp;
}
void siftdown(int * line, int root, int end){
    int tmp;
    while(root < end){
        tmp = root << 1 | 1;
        if(tmp >= end) return ;
        if(tmp + 1 < end && line[tmp + 1] > line[tmp]) ++ tmp;
        if(line[tmp] > line[root]){
            swap(&line[tmp],&line[root]);
            root = tmp;
        }else return ;
    }
}

void heapify(int * line, int n){
    for(int i = (n - 2) >> 1;i >= 0;-- i)
        siftdown(line, i, n);
}

int * heapsort(int *line, int n){
    heapify(line, n);
    for(int i = n - 1;i >= 0;-- i){
        swap(&line[i],&line[0]);
        siftdown(line, 0, i);
    }
    return line;
}
int main(){
    int n;
    int line[MAX];
    scanf("%d",&n);
    for(int i = 0;i < n;++ i){
        scanf("%d",&line[i]);
    }
    heapsort(line, n);
    for(int i = 0;i < n;++ i) printf("%d ",line[i]);
    return 0;
}

```
