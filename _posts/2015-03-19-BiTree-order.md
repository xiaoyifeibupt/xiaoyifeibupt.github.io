---
layout: post
title:  algorithms——二叉树遍历
categories: [Algorithms]
---

1）定义：有且仅有一个根结点，除根节点外，每个结点只有一个父结点，最多含有两个子节点，子节点有左右之分。  
2）存储结构  
二叉树的存储结构可以采用顺序存储，也可以采用链式存储，其中链式存储更加灵活。  
在链式存储结构中，与线性链表类似，二叉树的每个结点采用结构体表示，结构体包含三个域：数据域、左指针、右指针。  

```cpp
template <class T>
struct BiNode   //二叉树的结点结构
{
  T data;       
  BiNode<T> *lchild, *rchild;
};
```

 二叉树遍历通常借用“栈”这种数据结构实现，有两种方式：递归方式及非递归方式。  
在递归方式中，栈是由操作系统维护的，用户不必关心栈的细节操作，用户只需关心“访问顺序”即可。因而，采用递归方式实现二叉树的遍历比较容易理解，算法简单，容易实现。  
递归方式实现二叉树遍历的C++语言代码如下：

```cpp
void preOrder(BiNode *root, void(*visit)(T)) {
    if(root) {
        visit(root -> data);
        preOrder(root -> lchild,visit);
        preOrder(root -> rchild,visit);
    }
}

void inOrder(BiNode *root, void(*visit)(T)) {
    if(root) {
        inOrder(root -> lchild,visit);
        visit(root -> data);
        inOrder(root -> rchild,visit);
    }
}

void postOrder(BiNode *root, void(*visit)(T)) {
    if(root) {
        postOrder(root -> lchild,visit);
        postOrder(root -> rchild,visit);
        visit(root -> data);
    }
}
```
以上代码中，`visit`为一函数指针，用于传递二叉树中对结点的操作方式

首先考虑非递归先序遍历（NLR）。在遍历某一个二叉（子）树时，以一当前指针记录当前要处理的二叉（左子）树，以一个栈保存当前树之后处理的右子树。首先访问当前树的根结点数据，接下来应该依次遍历其左子树和右子树，然而程序的控制流只能处理其一，所以考虑将右子树的根保存在栈里面，当前指针则指向需先处理的左子树，为下次循环做准备；若当前指针指向的树为空，说明当前树为空树，不需要做任何处理，直接弹出栈顶的子树，为下次循环做准备。

```cpp
void preOrder(BiNode *root, void(*visit)(T)) {
    if(root == NULL) return;
    stack<T> rs;
    BiNode *p = root;
    while(p || !rs.empty()) {
        if(p) {
            visit(p -> data);
            rs.push(p);
            p = p -> lchild;
        }
        else {
            p = rs.pop();
            p = p -> rchild;
        }
    }
}
```
对于非递归中序遍历，若当前树不为空树，则访问其根结点之前应先访问其左子树，因而先将当前根节点入栈，然后考虑其左子树，不断将非空的根节点入栈，直到左子树为一空树；当左子树为空时，不需要做任何处理，弹出并访问栈顶结点，然后指向其右子树，为下次循环做准备。

```cpp
void inOrder(BiNode *root, void(*visit)(T)) {
    if(root == NULL) return;
    stack<T> rs;
    BiNode *p = root;
    while(p || !rs.empty()) {
        if(p) {
            rs.push(p);
            p = p -> lchild;
        }
        else {
            p = rs.pop();
            visit(p -> data);
            p = p -> rchild;
        }
    }
}
```
最后谈谈非递归后序遍历。由于在访问当前树的根结点时，应先访问其左、右子树，因而先将根结点入栈，接着将右子树也入栈，然后考虑左子树，重复这一过程直到某一左子树为空；如果当前考虑的子树为空，若栈顶不为空，说明第二栈顶对应的树的右子树未处理，则弹出栈顶，下次循环处理，并将一空指针入栈以表示其另一子树已做处理；若栈顶也为空树，说明第二栈顶对应的树的左右子树或者为空，或者均已做处理，直接访问第二栈顶的结点，访问完结点后，若栈仍为非空，说明整棵树尚未遍历完，则弹出栈顶，并入栈一空指针表示第二栈顶的子树之一已被处理。

```cpp
void postOrder(BiNode *root, void(*visit)(T)) {
    stack<T> rs;
    BiNode *p = root;
    while(1) {
        if(p) {
            rs.push(p);
            rs.push(p -> rchild);
            p = p -> lchild;
        }
        else if(!p) {
            p = rs.pop();
            if(!p) {
                p = rs.pop();
                visit(p -> data);
                if(rs.empty()) break;
                p = rs.pop();
            }
            rs.push(NULL);
        }
    }
}
```

层序遍历

```cpp
void levelOrder(BiNode *root, void(*visit)(T)) {
    if(root == NULL) return;
    deque<T*> Q;
    Q.push_back(root);
    while(!Q.empty()) {
        BiNode *p = Q.front();
        Q.pop_front();
        visit(p);
        if(p -> lchild)
            Q.push_back(p -> lchild);
        if(p -> rchild)
            Q.push_back(p -> rchild);
    }
}
```
