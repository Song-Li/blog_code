title: "leetcode Ugly Number II"
date: 2015-10-26 20:44:58
tags: leetcode
---
This is not a difficult one.
Basic idea is, assum that we have a self-sorted queue. Everytime, we choose the smallest one *n*, insert 2\*n , 3\*n , 5\*n into this queue. After n times, we will get the n-th Ugly number.
The time will be $O(nlogn)$. Do we have a better way? The answer is YES!
<!--more-->
Remeber the key point is we need to always get the smallest number of this queue. If we use three queue, every time we choose the smallest number of this three queue rears n. And put 2\*n to the *L2* queue. put 3\*n to the *L3* queue, put 5\*n to the *L5* queue. We can confirm that these queues are in order. Because every time the choosen number *n* is larger than the one before that. So the in every queue, the new numbers will be the largest one.

```
int min(int a,int b){
    if(a > b) return b;
    return a;
    
}
int nthUglyNumber(int n) {
    int l2[10000], l3[10000],l5[10000];
    int c2 = 1,c3 = 1,c5 = 1;
    int h2 = 0,h3 = 0,h5 = 0;
    int now = 1;
    l2[0] = l3[0] = l5[0] = 1;
    for(int i = 0;i < n;++ i){
        now = min(l2[h2],min(l3[h3],l5[h5]));
        if(now == l2[h2]) h2 ++;
        if(now == l3[h3]) h3 ++;
        if(now == l5[h5]) h5 ++;
        l2[c2 ++] = now * 2;
        l3[c3 ++] = now * 3;
        l5[c5 ++] = now * 5;
        cout << now << endl;
    }
    return now;
}
```

