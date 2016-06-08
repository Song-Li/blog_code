title: "Tower of Hanoi"
date: 2015-10-21 20:44:42
tags: Algorithm
---
I'm so tierd and upset today. I want to practise a DFS, BFS and a recursive.
I find it's easy for me to write a BFS or DFS. So I just wrote a Hanoi tower.
It's a very basic algorithm. This is my code.
<!--more-->
```
#include <stdio.h>
int hanoi(int n, int posi, int to){
    if(!n) return 0;
    int res = 0;
    res += hanoi(n - 1, posi, 3 - posi - to);
    printf("Move %d to %d\n",posi, to);
    res += hanoi(n - 1, 3 - posi - to, to);
    res ++;
    return res;
}
int main(){
    int n;
    scanf("%d",&n);
    printf("Total: %d\n",hanoi(n,0,1));
    return 0;
}

```
