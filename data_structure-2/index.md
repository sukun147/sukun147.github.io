# 数据结构学习(二)


<!--more-->

[TOC]

# 本系列学习笔记链接

- 第一章：[数据结构学习(一)](/data_structure-1)
- 第二章：[数据结构学习(二)](/data_structure-2)

# 数据结构——用C语言描述(二)

## 线性结构

### 线性表

#### 线性表定义

线性表（LinearList）：由 n 个类型相同数据元素的有限序列，记作
$$
(a_1,a_2,···,a_i,a_{i+1},···,a_n)
$$
称 n 为线性表长度，特别的，当 n = 0 时，称为空表。

每个数据元素只有一个直接前驱和一个直接后继（首尾元素除外），即为一对一的线性关系。

线性表特点：

- 同一性
- 有穷性
- 有序性

对于线性表 ADT，有 9 种基本操作运算：

1. 初始化 InitList(L)
2. 销毁 DestroyList(L)
3. 置空 ClearList(L)
4. 判空 EmptyList(L)
5. 求长度 ListLength(L)
6. 查找 Locate(L, e)
7. 存取 GetData(L, i)
8. 插入 InsList(L, i, e)
9. 删除 DelList(L, i, &e)

#### 线性表的顺序存储

```c
#define MAXSIZE 100
//线性表数据类型定义
typedef struct
{
    ElemType elem[MAXSIZE]; //线性表占用的数组空间
    int last;               //线性表中最后一个元素在数组中的位置下标
} SeqList;
/*
两种变量定义方式
SeqList L1 使用L1.elem[i-1]访问
SeqList *L2 使用L2->elem[i-1]访问
*/
```

##### 查找操作

```C
//按内容查找
int Locate(SeqList L, ElemType e)
{
    int i = 0;
    while ((i <= L.last) && (L.elem[i] != 0))
        i++;
    if (i <= L.last)
        retrun i + 1; //若找到值为e的元素，则返回其序号
    else
        return -1; //若没找到，返回空序号
}

//按序号查找
ElemType GetData(SeqList L, int i)
{
    if (i <= L.last)
        return L.elem[i - 1];
    else
    {
        printf("序号超过上限");
        return 0;
    }
}
```

##### 插入操作

```c
//在表的第i个位置之前插入一个新元素e
int InsList(SeqList *L, int i, ElemType e)
{
    int k;
    if ((i < 1) || (i > L->last + 2))
    {
        printf("插入位置i值不合法");
        return 0;
    }
    if (L->last >= MAXSIZE - 1)
    {
        printf("表已满无法插入");
        return 0;
    }
    for (k = L->last; k >= i - 1; k--)
        L->elem[k + 1] = L->elem[k];
    L->elem[i - 1] = e;
    L->last++;
    return 1;
}
```

##### 删除操作

```C
//将表的第i个元素删去
int DelList(SeqList *L, int i, ElemType *e)
{
    int k;
    if ((i < 1) || (i > L->last + 1))
    {
        printf("删除位置不合法！");
        return 0;
    }
    *e = L->elem[i - 1];
    for (k = i; k <= L->last; k++)
        L->elem[k - 1] = L->elem[k];
    L->last--;
    return 1;
}
```

##### 顺序表合并算法

有两个顺序表 LA 和 LB，其元素均为递增有序排列，编写算法，将两个有序表合并成一个递增有序的顺序表 LC。

```c
void merg(SeqList *LA, SeqList *LB, SeqList *LC)
{
    int i, j, k;
    i = 0;
    j = 0;
    k = 0;
    while (i <= LA->last && j <= LB->last)
        if (LA->elem[i] <= LB->elem[j])
        {
            LC->elem[k] = LA->elem[i];
            i++;
            k++;
        }
        else
        {
            LC->elem[k] = LB->elem[j];
            j++;
            k++;
        }
    while (i <= LA->last) //表A长于表B
    {
        LC->elem[k] = LA->elem[i];
        i++;
        k++;
    }
    while (j <= LB->last) //表B长于表A
    {
        LC->elem[k] = LB->elem[j];
        j++;
        k++;
    }
    LC->last = LA->last + LB->last + 1;
}
```

比较循环的时间复杂度为
$$
O(LA->last+LB->last)
$$
复制循环时间复杂度为
$$
O(max(LA->last,LB->last))
$$

##### 顺序表优缺点总结

- 优点
  1. 无需为表示结点间的逻辑关系而增加额外的存储空间
  2. 可方便地随机存取查找表中的任一元素
- 缺点
  1. 大量插入删除时效率低，除表尾位置外，在其它位置插入删除都必须移动大量元素（平均移动次数为表长度一半）
  2. 由于顺序表要求占用连续的存储空间,存储分配只能预先进行静态分配。因此当表长变化较大时，难以确定合适的存储规模。

总结：便于随机存取，不适合动态变化

#### 线性表的链式存储

##### 单链表

![image-20220122153019138](https://s2.loli.net/2022/01/22/Bnj81JZgApR237O.png)

```C
typedef struct
{
    ElemType data;
    struct Node *next;
} Node, *LinkList;
LinkList L;
```

###### 求长度

```C
int ListLength(LinkList L)
{
    Node *p;
    p = L->next;
    j = 0;
    while (p != NULL)
    {
        p = p->next;
        j++;
    }
    return j;
}
```

###### 建立空表

```c
void InitList(LinkList *L)
{
    L = (LinkList *)malloc(sizeof(Node));
    L->next = NULL;
}
```

###### 头插法建表

```c
LinkList CreateFromHead(LinkList L)
{
    Node *s;
    char c;
    int flag = 1;
    while (flag)
    {
        c = getchar();
        if (c!='$')
        {
            s = (LinkList *)malloc(sizeof(Node));
            s->data = c;
            s->next = L->next;
            L->next = s;
        }
        else
            flag = 0;
    }
}
```

###### 尾插法建表

```c
LinkList CreateFromTail(LinkList L)
{
    LinkList L;
    Node *r,
        *s;
    char c;
    int flag = 1;
    r = L;
    while (flag)
    {
        c = getchar();
        if (c != '$')
        {
            s = (LinkList *)malloc(sizeof(Node));
            s->data = c;
            r->next = s;
            r = s;
        }
        else
        {
            flag = 0;
            r->next = NULL;
        }
    }
}
```

###### 查找

```c

```



##### 循环链表



##### 双向链表



##### 静态链表


