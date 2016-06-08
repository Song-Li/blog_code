title: "Heap insertion and deletion in C"
date: 2015-10-20 20:22:43
tags: Algorithm
---
It's a program about basic insertion and deletion of a heap.(include heapify)
It's insertion and deletion time complexity is $O(logn)$
Once I finished this code and began to run, it runs without bug.
Which means I finished this code Without any debuging~
I spent half an hour to write this program
This is a *small top* heap
<!--more-->
```
#include <stdio.h>
#define MAX 1000
void swap(int * a,int * b){
    int tmp = *a;
    *a = *b;
    *b = tmp;
}
void siftdown(int * line, int n,int root){
    int sift = root;
    while(root < n){
        sift = root << 1 | 1;
        if(sift + 1 < n && line[sift + 1] < line[sift]) ++ sift;
        if(sift < n && line[root] > line[sift]) {
            swap(&line[root], &line[sift]);
            root = sift;
        }else break;
    }
}
void heapify(int * line, int n){
    int now = (n - 2) >> 1;
    for(int i = now;i >= 0;-- i){
        siftdown(line, n, i);
    }
    return ;
}
void del(int * line, int n){
    line[0] = line[n - 1];
    siftdown(line, n - 1, 0);
}
void siftup(int * line, int n){
    int sift = n;
    while(n){
        sift = (n - 1) >> 1;
        if(line[sift] > line[n]) swap(&line[sift],&line[n]), n = sift;
        else break; 
    }
}
void insert(int * line, int n, int a){
    line[n] = a;
    siftup(line, n);
}
void output(int * line, int n){
    for(int i = 0;i < n;++ i) printf("%d ",line[i]);
    printf("\n");
    return ;
}
int main(){
    int n,tmp;
    int line[MAX];
    scanf("%d",&n);
    for(int i = 0;i < n;++ i){
        scanf("%d",&line[i]);
    }
    heapify(line, n);
    output(line, n);
    printf("1 a for insert a, 0 for delete the heap top\n");
    while(1){
        scanf("%d",&tmp);
        if(tmp){
            scanf("%d",&tmp);
            insert(line, n ++, tmp);
        }else if(n > 0) del(line, n --);
        else printf("No number left\n");
        output(line, n);
    }
}
```
