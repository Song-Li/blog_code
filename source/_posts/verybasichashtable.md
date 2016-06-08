title: "Very basic hashtable"
date: 2015-10-20 19:59:30
tags: Algorithm
---
This is only a very basic hashtable
I use half an hour to write it(such a easy program takes me so long)
The hash function of string is simply sum all of the charactors up and times a prime number
This program is used to calculate how many times of every input string appears
The structure of this hashtable is like this:
<!--more-->
![hashtable](/image/20151020195930.jpg)
Just a simple implement without any optimize
The input is 
n//The number of strings
string a
...
string n
The output is the hashtable
```
#include <stdio.h>
#include <iostream>
#include <string>
using namespace std;
#define HASH 7
struct Node{
    string val;
    int num;
    Node * next;
};

Node * hashTable[HASH];

int hash(string s){// stupid way to hash a string
    int res = 0;
    for(int i = 0;i < s.length();++ i) res += s[i];
    res = res * 97 * 31 / 37;
    return res;
}


bool insert(string s){
    int h = hash(s) % HASH;
    Node * node = hashTable[h];
    if(!node) {
        node = new Node;
        node -> val = s;
        node -> num = 1;
        node -> next = NULL;
        hashTable[h] = node;
    }else{
        while(node){
            if(node -> val == s){
                (node -> num) ++;
                return true;
            }
            node = node -> next;
        }
        node = new Node;
        node -> val = s;
        node -> num = 1;
        node -> next = hashTable[h];
        hashTable[h] = node;
    }
    return false;
}
void output(){
    Node * node;
    for(int i = 0;i < HASH;++ i){
        if(hashTable[i]){
            node = hashTable[i];
            while(node){
                cout << node -> val << ' ' << node -> num << ' ';
                node =  node -> next;
            }
            cout << endl;
        }
    }
    return ;
}

int main(){
    int n;
    string s;
    cin >> n;
    for(int i = 0;i < n;++ i){
        cin >> s;
        insert(s);
    }
    output();
    return 0;
}


```
