# 单链表基本操作


C语言的链表基本操作。

<!--more-->

## 为何需要链表？

学习 C 语言的开始，我们最早接触的数据类型是数组，它有两个特点：

1. 顺序存储所有的元素，**连续**占用内存空间；
2. 创建数组时需要事先知道存储元素的个数，确定数组的长度；
3. 查询元素复杂度是 O(1)O(1)，删除和插入的复杂度是 O(n)O(n)。

数组元素非常适合元素个数基本保持不变的场景，特别便于查阅操作，但是如果数据存储的元素个数经常发生变化，操作起来复杂度就很高，因此急需一种数据结构便于插入和删除，链表就很合适。

### 概念

一个链表的示意图如下所示

![](http://c.biancheng.net/cpp/uploads/allimg/120129/fsds34343fgrt.gif)

链表在内存中不是连续的顺序存储的，链表包括表头，表尾和中间结点。

- 中间结点包括数据域和指针域
  - 数据域用于储存当前结点的数据，通常是数据类型
  - 指针域指示下一个结点在内存中的地址，是一个指针
- 表头是特殊的中间结点，它的数据域可以包括表的长度或者什么都没有，但是指针域必须指向单链表的第一个中间结点的位置
- 表尾也是特殊的中间结点，数据域可以存储数据，因为已经没有后继结点，指针域为空

### 优势

单链表和数组相比的优势体现在如下两个方面：

1. 空间存储更灵活，单链表不需要**连续占用**内存空间，因为每个结点都知道它的后继结点在内存中的位置，所以不需要连续存储；
2. 空间申请更灵活，可以使用动态内存分配，在程序运行时候按需申请内存大小，避免数组申请过大造成的浪费。
3. 空间释放和增加更灵活，删除一个元素仅仅需要将该元素的前置元素指针指向它的后继结点，释放该结点即可，增加元素只需要更改指针指向即可，复杂度是 O(1)O(1)

但是，单链表的劣势也很明显，单链表中的元素仅仅知道后继结点的位置，如果需要查询某个元素，只能从头结点开始一个一个地去遍历，而不像数组直接给出下标一步就能定位到，因此单链表的查询复杂度是 O(n)O(n)。

## 代码示例

假定单链表的数据元素的数据类型是`ElemType`，那单链表可以定义如下

```c
typedef struct Node {
    ElemType data;
    struct Node *next;
}Node;
typedef struct Node * LinkList;
```

特别注意，有个特点，单链表有一个指向自身的指针，表示下一个结点`next`的位置。单链表的常用操作包括：创建，查询，插入，删除，清除。

### 创建列表

创建有两种方法，头插法和尾插法，第一种是保持新插入的元素始终在表的第一个元素，第二个保持新插入的元素始终在表的最后一个元素。

```c
/*头插法创建链表*/
void CreateListHead(LinkList *L, int n){
    LinkList p;
    int i;
    srand(time(0));
    *L = (LinkList)malloc(sizeof(Node));
    (*L)->next = NULL;
    
    for(i = 0; i < n; i++){  
      p = (LinkList)malloc(sizeof(Node));
      p->data = rand() % 100 + 1;
      p->next = (*L)->next;
      (*L)->next = p;       
    }
}
```

尾插法代码如下

```c
/*尾插法创建链表*/
void CreateListTail(LinkList *L, int n){
    LinkList p,r;
    int i;
    srand(time(0));
    *L = (LinkList)malloc(sizeof(Node));
    (*L)->next = NULL;
    
    r = *L;
    for(i = 0; i < n; i++){  
      p = (LinkList)malloc(sizeof(Node));
      p->data = rand() % 100 + 1;
      r->next = p;
      r = p;       
    }
    
    r->next = NULL;
}
```

### 查询元素

工作的基本原理就是**工作指针后移**，这个技巧也是单链表操作的基础和关键。

```c
/* 单链表已经存在，用e返回L中的第i个元素的值*/
Status GetElem(LinkList L, int i, ElemType *e)
{
    int j = 1;    
    LinkList p = L->next;
    
    while(p && j < i) //p不为空并且j还没有和i相等时，继续往后查找
    {
        p = p->next; //一直指向下一个结点，顺藤摸瓜
        ++j;
    }
    
    if(!p || j > i)
        return ERROR;
    
    *e = p->data;
    return OK;
}
```

### 插入元素

```c
/* 单链表已经存在，在第i个元素位置插入新的元素e，L的表长加1*/
Status ListInsert(LinkList *L, int i, ElemType e)
{
    int j = 1;    
    LinkList p = *L->next, s;
    
    while(p && j < i) //p不为空并且j还没有和i相等时，继续往后查找
    {
        p = p->next; //一直指向下一个结点，顺藤摸瓜
        ++j;
    }
    
    if(!p || j > i)
        return ERROR;
    
    s = (LinkList)malloc(sizeof(Node));
    
    s->data = e;
    s->next = p->next;
    p->next = s;
    return OK;
}
```

### 删除元素

```c
/* 单链表已经存在，删除链表L的第i个元素，并用e返回其值，表长减1*/
Status ListInsert(LinkList *L, int i, ElemType *e)
{
    int j = 1;    
    LinkList p = *L->next, q;
    
    while(p && j < i) //p不为空并且j还没有和i相等时，继续往后查找
    {
        p = p->next; //一直指向下一个结点，顺藤摸瓜
        ++j;
    }
    
    if(!p || j > i)
        return ERROR;
    
    q = p->next;
    p->next = q->next;
    *e = q->data;
    free(q);    
    return OK;
}
```

### 删除整表

```c
/*顺序链表L已经存在，将L重置为空表*/
Status ClearList(LinkList *L)
{
    LinkList p, q;
    p = (*L)->next;
    while(p)
    {
        q = p->next;
        free(p);
        
        p = q;
    }
    (*L)->next = NULL;
    return OK;
}
```

## 参考资料

- [链表的创建和遍历_C 语言中文网](http://c.biancheng.net/cpp/biancheng/view/57.html)
- [C 语言链表的概念_C 语言中文网](http://c.biancheng.net/cpp/html/98.html)
- [C 语言单链表实现 19 个功能完全详解 - Lee.Kevin - 博客园](http://www.cnblogs.com/lifuqing/archive/2011/08/20/List.html)
