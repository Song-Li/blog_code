---
title: convert sorted list to binary search tree
date: 2016-06-15 17:44:30
tags: leetcode
---
Given a singly linked list where elements are sorted in ascending order, convert it to a height balanced BST.
<!--more-->
This is my simple code: simple recursive
```
class Solution {
    public:
        TreeNode* sortedListToBST(ListNode* head) {
            if(!head) return NULL;
            if(!(head->next)) return new TreeNode(head->val);
            ListNode *fast = head, *slow = head, *pre = NULL;
            while(fast && fast->next){
                pre = slow;
                fast = fast->next->next;
                slow = slow->next;
            }
            TreeNode *root = new TreeNode(slow->val);
            pre->next = NULL;
            root->left = sortedListToBST(head);
            root->right = sortedListToBST(slow->next);
            return root;
        }
};
```
This is a strange way:
```
class Solution {
    public:
        TreeNode* sortedListToBST(ListNode* head) {
            int size=0;
            ListNode *save=head;
            while(head){
                size++;
                head = head->next;
            }
            head = save;
            TreeNode* root = helper(head,size);
            return root;
        }
        TreeNode* helper(ListNode*& head,int size){
            if(head==NULL ||size<=0) return NULL;
            int rightSize = (size-1)/2; 
            TreeNode* left = helper(head,size-1-rightSize);
            TreeNode* root = new TreeNode(head->val);
            head = head->next;
            root->left = left;
            root->right = helper(head,rightSize);
            return root;
        }
};
```
