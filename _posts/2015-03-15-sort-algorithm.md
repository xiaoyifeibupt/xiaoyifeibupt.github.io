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

```cpp
void SelectionSort(int* pDataArray, int iDataNum) {  
    for (int i = 0; i < iDataNum - 1; i++) {		//从第一个位置开始  
        int index = i;  
        for (int j = i + 1; j < iDataNum; j++)    //寻找最小的数据索引   
            if (pDataArray[j] < pDataArray[index])  
                index = j;  
  
        if (index != i)    //如果最小数位置变化则交换  
            Swap(&pDataArray[index], &pDataArray[i]);
    }  
}  
```

###归并排序

特点：stable sort、Out-place sort

思想：运用分治法思想解决排序问题。

最坏情况运行时间：O(nlgn)

最佳运行时间：O(nlgn)

```cpp
void merge(int *data, int p, int q, int r)  
{  
    int n1, n2, i, j, k;  
    int *left=NULL, *right=NULL;  
   
    n1 = q-p+1;   
    n2 = r-q;  
   
    left = new int[n1];   
    right = new int[n2];  
    for(i=0; i<n1; ++i)  //对左数组赋值  
        left[i] = data[p+i];  
    for(j=0; j<n2; ++j)  //对右数组赋值  
        right[j] = data[q+1+j];  
   
    i = j = 0;   
    k = p;  
    while(i<n1 && j<n2) //将数组元素值两两比较，并合并到data数组  
    {  
        if(left[i] <= right[j])  
            data[k++] = left[i++];  
        else  
            data[k++] = right[j++];  
    }  
   
    while(i<n1) //如果左数组有元素剩余，则将剩余元素合并到data数组  
        data[k++] = left[i++];  
    while(j<n2) //如果右数组有元素剩余，则将剩余元素合并到data数组  
        data[k++] = right[j++];  
}  
   
void mergeSort(int *data, int p, int r)  
{  
    int q;  
    if(p < r) //只有一个或无记录时不须排序   
    {  
        q = (int)((p+r)/2);      //将data数组分成两半     
        mergeSort(data, p, q);   //递归拆分左数组  
        mergeSort(data, q+1, r); //递归拆分右数组  
        merge(data, p, q, r);    //合并数组  
    }  
}
```

###快速排序

特性：unstable sort、In-place sort。

最坏运行时间：当输入数组已排序时，时间为O(n^2)，当然可以通过随机化来改进（shuffle array 或者 randomized select pivot）,使得期望运行时间为O(nlgn)。

最佳运行时间：O(nlgn)

快速排序的思想也是分治法。

当输入数组的所有元素都一样时，不管是快速排序还是随机化快速排序的复杂度都为O(n^2)，而在算法导论第三版的思考题7-2中通过改变Partition函数，从而改进复杂度为O(n)。

注意：只要partition的划分比例是常数的，则快排的效率就是O(nlgn)

```cpp
int partition(int a[],int p,int r) {
	int x = a[r];
	int i = p - 1;
	for(int j = p; j != r;++j) {
		if(a[j] <= x) {
			i = i + 1;
			swap(a[i],a[j]);
		}
	}
	swap(a[i + 1],a[r]);
	return i+1;
}

void quicksort(int a[],int p,int r) {
	if(p < r) {
		int q = partition(a,p,r);
		quicksort(a,p,q - 1);
		quicksort(a,q + 1,r);
	}
}
```

###堆排序

特性：unstable sort、In-place sort。

最优时间：O(nlgn)

最差时间：O(nlgn)

堆还能用于构建优先队列,优先队列应用于进程间调度、任务调度等,堆数据结构应用于Dijkstra、Prim算法。

```cpp
void max_heapify(int a[],int i,int heap_size) {
	int le = (i<<1) + 1;
	int ri = le+1;
	int large;
	if(le < heap_size && a[le] > a[i])
		large = le;
	else
		large = i;
	if(ri < heap_size && a[ri] > a[large])
		large = ri;
	if(large != i)
	{
		swap(a[i],a[large]);
		max_heapify(a,large, heap_size);
	}
}

int heap_size = length;
for(int i = (length-1)/2; i != -1; --i)
max_heapify(aa,i, heap_size);
for(int j = length-1; j != 0; --j){
	swap(aa[0] ,aa[j]);
	--heap_size;
	max_heapify(aa,0, heap_size);
}
```

