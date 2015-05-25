---
layout: post
title:  Algorithms——排序算法总结
categories: [Algorithms]
---

###插入排序

特点：stable sort、In-place sort

最优复杂度：当输入数组就是排好序的时候，复杂度为O(n)，而快速排序在这种情况下会产生O(n^2)的复杂度。

最差复杂度：当输入数组为倒序时，复杂度为O(n^2)

插入排序比较适合用于“少量元素的数组”。

```cpp
void InsertSort(int a[], int length) {
    for(int i = 1; i != length; ++i) {
	    int key = a[i];
		int j = i - 1;
		while(j != -1 && a[j] > key) {
			swap(a[j],a[j + 1]);
			j = j - 1;
		}
		aa[j + 1] = key;
	}
```

###冒泡排序

特点：stable sort、In-place sort

思想：通过两两交换，像水中的泡泡一样，小的先冒出来，大的后冒出来。

最坏运行时间：O(n^2)

最佳运行时间：O(n^2)（当然，也可以进行改进使得最佳运行时间为O(n)）

```cpp
void BubbleSort(int a[], int length) {
    for(int i = 0; i != length - 1; ++i) {
        for(j = length - 1;j != i + 1; --j) {
            if(a[j] < a[j - 1])
                swap(a[j], a[j - 1]);
        }
    }
}
```

改进版冒泡排序

最佳运行时间：O(n)

最坏运行时间：O(n^2)

第一步优化，如果上面代码中，里面一层循环在某次扫描中没有执行交换，则说明此时数组已经全部有序列，无需再扫描了。因此，增加一个标记，每次发生交换，就标记，如果某次循环完没有标记，则说明已经完成排序。

```cpp
void improved_BubbleSort(int a[], int length) {
    bool flag = true;
    for(int i = 0; i != length - 1; ++i) {
        flag = false
        for(j = n - 1;j != i + 1; --j) {
            if(a[j] < a[j - 1]) {
                swap(a[j], a[j - 1]);
                flag = true;
            }
        }
        if(!flag) break;
    }
}
```

第二步优化，在第一步优化的基础上发进一步思考：如果R[0..i]已是有序区间，上次的扫描区间是R[i..n]，记上次扫描时最后 一次执行交换的位置为lastSwapPos，则lastSwapPos在i与n之间，不难发现R[i..lastSwapPos]区间也是有序的，否则这个区间也会发生交换；所以下次扫描区间就可以由R[i..n] 缩减到[lastSwapPos..n]。

```cpp
void improved_BubbleSort(int a[], int size) {  
    int lastSwapPos = 0,lastSwapPosTemp = 0;
    for (int i = 0; i < size - 1; i++) {  
        lastSwapPos = lastSwapPosTemp;  
        for (int j = size - 1; j >lastSwapPos; j--) {  
            if (a[j - 1] > a[j])  {  
                int temp = a[j - 1];  
                a[j - 1] = a[j];  
                a[j] = temp;  
                lastSwapPosTemp = j;  
            }  
        }  
        if (lastSwapPos == lastSwapPosTemp)  
            break;  
    }  
}  
```
###选择排序

特性：In-place sort，unstable sort。

思想：每次找一个最小值。

最好情况时间：O(n^2)。

最坏情况时间：O(n^2)。

