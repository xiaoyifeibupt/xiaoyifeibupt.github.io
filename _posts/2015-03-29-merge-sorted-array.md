---
layout: post
title:  Algorithms——合并子数组
categories: [Algorithms]
---

设子数组a[0:k-1]和a[k,n-1]已经排好序(0<=k<=n-1).设计一个合并这两个子数组的排序好的数组a[0:n-1]的算法。要求算法最坏情况下所用的计算时间为O(n)，且只用到O(1)的辅助空间。

解答此题可以用向右(左)循环换位合并的思想，先用二分搜索查找a[0]在数组a[k:n-1]中的位置p使得a[p]<a[0]<=a[p+1].然后数组a[0:p]向右循环换位p-k+1个位置，换位完成后，数组a[0]及其左边的元素都排序好的元素。对剩下的数组元素重复上述的过程，知道只剩下一个数组段，此刻整个数组已排好序。

```cpp
int binarySearch(int *a, int &x, int left, int right){  
    int middle;  
    while (left <= right)  
    {  
        middle = left + ((right - left)>>1);
        if(x == a[middle])  
            return middle;  
        if(x > a[middle]){  
            left = middle + 1;  
        }else{  
            right = middle - 1;  
        }  
    }  
    if(x > a[middle])  
        return middle;  
    return middle - 1;  
}  
void shiftRight(int *a, int s, int t, int k){  
    for(int i = 0; i < k; i++){  
        int temp = a[t];  
        for(int j = t; j > s; j--){  
            a[j] = a[j - 1];  
        }  
        a[s] = temp;  
    }  
}  
void mergeArray(int *a, int k, int n){  
    int i = 0;  
    int j = k;  
    while (i < j&&j < n)  
    {  
        int p = binarySearch(a, a[i], j, n - 1);  
        shiftRight(a, i, p, p-j+1);  
        j = p + 1;  
        i += p - j + 2;  
    }  
}
```
