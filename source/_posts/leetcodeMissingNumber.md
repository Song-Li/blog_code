title: "leetcode Missing Number"
date: 2015-10-26 21:45:20
tags: leetcode
---
Use the sumup of all the number subtract the sum of them.
```
int missingNumber(int* nums, int numsSize) {
    long long int sum = 0;
    for(int i = 0;i < numsSize;++ i) sum += nums[i];
    return (numsSize + 1) * numsSize / 2 - sum;
}
```
