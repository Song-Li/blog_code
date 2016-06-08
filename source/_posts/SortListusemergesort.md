title: "Sort List use merge sort"
date: 2015-10-16 21:08:58
tags: Algorithm
---
Why you are so sad? Just because you can not sort a list?
If someone want you to sort a list, what will you do? quicksort? No.
Merge sort is your choose! How much do you think a merge sort will cost? $O(n^2)$ ? $O(n^3)$ ? No!
It ONLY take you $O(n * log_2n)$! Buy now, you can get a input part freely!
<!--more-->
For merge sort, it's very useful. I don't want to say how merge sort works. If you want that, just Google "Merge sort".
I just want to practise. Here is my code.

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
using namespace std;
struct Node{
	int val;
	Node * next;
}node;

Node * merge(Node * s, int len){
    if(len == 1) return s;
    Node * a = s, *tmp;
    Node * res = NULL, *now = NULL;
    for(int i = 1;i < len >> 1;++ i)
        s = s -> next;
    tmp = s -> next;
    s -> next = NULL;// find the start of the second part and set the end of the first part to NULL
    s = tmp;
    a = merge(a, len >> 1);
    s = merge(s, len - (len >> 1));
    if(a -> val < s -> val) res = a, a = a -> next;
    else res = s, s = s-> next;
    now = res;
    while(a && s){
        if(a -> val < s -> val){
            now -> next = a;
            now = a;
            a = a -> next;
        }else{
            now -> next = s;
            now = s;
            s = s -> next;
        }
    }
    if(!a) now -> next = s;
    else now -> next = a;
    return res;
}
int main(){
    int n = 0;
    int val = 0;
    Node * head, * tail;
    scanf("%d",&n);
    scanf("%d",&val);
    Node * tmp = (Node *)malloc(sizeof(Node));
    head = tmp;
    head -> val = val;
    tail = head;
    for(int i = 0;i < n - 1;++ i){
        scanf("%d",&val);
        tmp = (Node *)malloc(sizeof(Node));
        tmp -> next = NULL;
        tmp -> val = val;
        tail -> next = tmp;
        tail = tmp;
    }
    head = merge(head, n);
    while(head){
        printf("%d ",head -> val);
        head = head -> next;
    }
    printf("\n");
    return 0;
	
}
```
