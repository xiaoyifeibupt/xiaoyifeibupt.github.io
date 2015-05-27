---
layout: post
title: Algorithms——反转链表
categories: [Algorithms]
---

假设单链表的数据结构定义如下：

```cpp
struct ListNode {
	int data;
	ListNode* next;
}
```
###调整指针前后指向

	a -> b -> ... -> h -> i -> j -> ...
	a <- b <- ... <- h <- i    j -> ...

由于结点i的next指向了它的前一个结点，导致我们无法在链表中遍历到j。为了避免链表在结点i处断开，我们在调整之前先把结点j保存下来。

```cpp
ListNode* reverseLinkedList(ListNode* head) {
	ListNode* reverseHead = NULL;
	ListNode* pNode = head;
	ListNode* prev = NULL;
	while(pNode) {
		ListNode* pNext = pNode -> next;
		if(pNext == NULL)
			reverseHead = pNode;
		
		pNode -> next = prev;
		
		prev = pNode;
		pNode = pNext;
	}
	return reverseHead;
}
```

###重建一个新的链表

```cpp
ListNode* reverseLinkedList(ListNode* head) {
	if(head == Null) return NULL;
	
	ListNode* reverseHead = NULL;
	ListNode* pNode = head;
	ListNode* qNode = NULL;
	
	reverseHead -> data = head -> data;
	reverseHead -> next = NULL;
	
	while(pNode -> next) {
		qNode = reverseHead -> next;
		reverseHead -> next = pNode -> next;
		pNode -> next = pNode -> next -> next;
		reverseHead -> next -> next = qNode;
	}
	return reverseHead;
}
```

###后面的结点往前插

为了反转这个单链表，我们先让头结点的next域指向结点2，再让结点1的next域指向结点3，最后将结点2的next域指向结点1，就完成了第一次交换，顺序就变成了Header-结点2-结点1-结点3-结点4-NULL，然后进行相同的交换将结点3移动到结点2的前面，然后再将结点4移动到结点3的前面就完成了反转

```cpp
ListNode* reverseLinkedList(ListNode* head) {
	if(head == Null) return NULL;
	ListNode* pNode = NULL;
	ListNode* qNode = NULL;
	
	pNode = head -> next;
	while(pNode -> next) {
		qNode = pNode -> next;
		pNode -> next = qNode -> next;
		qNode -> next = pNode;
		head -> next = qNode;
	}
	return head;
}
```

本方法是将后面的结点向前移动到头结点的后面，另外一种方法是将前面的结点移动到原来的最后一个结点的后面

###递归

在递归算法中的做法是：

1找到最后一个节点和倒数第二个节点，把最后一个节点设为头节点的后继

2反转这两个节点

3倒数第三个和第四个节点重复执行步骤2

其中注意，链表是以节点后继为NULL结束的，在更改指针的过程中要把改后的节点后继改为NULL

代码如下：

```cpp
void Inversion_Recursion(ListNode* p,ListNode* Head) {
	if(p->next==NULL) {
		Head->next=p;
		return;//找到最后一个节点
	}
	Inversion_Recursion(p->next,Head);
	p->next->next=p;//反转节点
	p->next=NULL;//第一个节点反转后其后继应该为NULL
}
```

