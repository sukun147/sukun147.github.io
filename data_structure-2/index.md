# 数据结构学习(二)


<!--more-->

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
    int elem[MAXSIZE]; //线性表占用的数组空间
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
int Locate(SeqList L, int e)
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
int GetData(SeqList L, int i)
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
int InsList(SeqList *L, int i, int e)
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
int DelList(SeqList *L, int i, int *e)
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
    int data;
    struct Node *next;
} Node, *LinkList;
LinkList L;
```

###### 求长度

```C
int ListLength(LinkList L)
{
    Node *p;
    int j;
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
    *L = (LinkList)malloc(sizeof(Node));
    (*L)->next = NULL;
}
```

###### 建表

```c
/*
头插法建表eg:
输入：123$
链表：321
*/
LinkList CreateFromHead(LinkList L)
{
    Node *s;
    char c;
    int flag = 1;
    while (flag)
    {
        c = getchar();
        if (c != '$')
        {
            s = (Node *)malloc(sizeof(Node));
            s->data = c;
            s->next = L->next;
            L->next = s;
        }
        else
            flag = 0;
    }
}
/*
尾插法建表eg:
输入：123$
链表：123
*/
LinkList CreateFromTail(LinkList L)
{
    Node *r, *s;
    char c;
    int flag = 1;
    r = L;
    while (flag)
    {
        c = getchar();
        if (c != '$')
        {
            s = (Node *)malloc(sizeof(Node));
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
//按序号查找（第i个）
Node *Get(LinkList L, int i)
{
    int j;
    Node *p;
    p = L;
    j = 0;
    while ((p->next != NULL) && (j < i))
    {
        p = p->next;
        j++;
    }
    if (i == j)
        return p;
    else
        return NULL;
}

//按值查找
Node *Locate(LinkList L, int key)
{
    Node *p;
    p = L->next;
    while (p != NULL)
        if (p->data != key)
            p = p->next;
        else
            break;
    return p;
}
```

###### 前插

```c
int InsList(LinkList L, int i, int e)
{
    Node *pre, *s;
    int k;
    pre = L;
    k = 0;
    while (pre != NULL && k < i - 1)
    {
        pre = pre->next;
        k++;
    }
    if (!pre)
    {
        printf("插入位置不合理！");
        return 0;
    }
    s = (Node *)malloc(sizeof(Node));
    s->data = e;
    s->next = pre->next;
    pre->next = s;
    return 1;
}
```

###### 删除

```c
void DelList(LinkList L, int i, int *e)
{
    Node *p, *r;
    int k;
    p = L;
    k = 0;
    while (p->next != NULL && k < i - 1)
    {
        p = p->next;
        k++;
    }
    if (k != i - 1)
    {
        printf("删除结点的位置i不合理！");
        return;
    }
    r = p->next;
    p->next = r->next;
    *e = r->data;
    free(r);
}
```

###### 示例

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct
{
    int data;
    struct Node *next;
} Node, *LinkList;

int ListLength(LinkList L)
{
    Node *p;
    int j;
    p = L->next;
    j = 0;
    while (p != NULL)
    {
        p = p->next;
        j++;
    }
    return j;
}

void InitList(LinkList *L)
{
    *L = (LinkList)malloc(sizeof(Node));
    (*L)->next = NULL;
}

LinkList CreateFromHead(LinkList L)
{
    Node *s;
    char c;
    int flag = 1;
    while (flag)
    {
        c = getchar();
        if (c != '$')
        {
            s = (Node *)malloc(sizeof(Node));
            s->data = c;
            s->next = L->next;
            L->next = s;
        }
        else
            flag = 0;
    }
}

LinkList CreateFromTail(LinkList L)
{
    Node *r, *s;
    char c;
    int flag = 1;
    r = L;
    while (flag)
    {
        c = getchar();
        if (c != '$')
        {
            s = (Node *)malloc(sizeof(Node));
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

Node *Get(LinkList L, int i)
{
    int j;
    Node *p;
    p = L;
    j = 0;
    while ((p->next != NULL) && (j < i))
    {
        p = p->next;
        j++;
    }
    if (i == j)
        return p;
    else
        return NULL;
}

Node *Locate(LinkList L, int key)
{
    Node *p;
    p = L->next;
    while (p != NULL)
        if (p->data != key)
            p = p->next;
        else
            break;
    return p;
}

int InsList(LinkList L, int i, int e)
{
    Node *pre, *s;
    int k;
    pre = L;
    k = 0;
    while (pre != NULL && k < i - 1)
    {
        pre = pre->next;
        k++;
    }
    if (!pre)
    {
        printf("插入位置不合理！");
        return 0;
    }
    s = (Node *)malloc(sizeof(Node));
    s->data = e;
    s->next = pre->next;
    pre->next = s;
    return 1;
}

void DelList(LinkList L, int i, int *e)
{
    Node *p, *r;
    int k;
    p = L;
    k = 0;
    while (p->next != NULL && k < i - 1)
    {
        p = p->next;
        k++;
    }
    if (k != i - 1)
    {
        printf("删除结点的位置i不合理！");
        return;
    }
    r = p->next;
    p->next = r->next;
    *e = r->data;
    free(r);
}

void Difference(LinkList LA, LinkList LB)
{
    Node *pre, *p, *q, *r;
    pre = LA;
    p = LA->next;
    while (p != NULL)
    {
        q = LB->next;
        while (q != NULL && q->data != p->data)
            q = q->next;
        if (q != NULL)
        {
            r = p;
            pre->next = p->next;
            p = p->next;
            free(r);
        }
        else
        {
            pre = p;
            p = p->next;
        }
    }
}

int main()
{
    LinkList L, p;
    int del;
    int length;
    InitList(&L);
    CreateFromHead(L);
    length = ListLength(L);
    printf("第一次的长度:%d\t", length);
    p = Get(L, 1);
    printf("第一个元素值:%c\n", p->data);
    if (InsList(L, 1, '0'))
    {
        length = ListLength(L);
        printf("第二次的长度:%d\t", length);
        p = Get(L, 1);
        printf("第一个元素值:%c\n", p->data);
    }
    DelList(L, 1, &del);
    length = ListLength(L);
    printf("第三次的长度:%d\t", length);
    printf("被删除元素值:%c\n", del);
    return 0;
}

/*
输入：
54321$
输出：
第一次的长度:5	第一个元素值:1
第二次的长度:6	第一个元素值:0
第三次的长度:5	被删除元素值:0
*/
```

###### 就地逆置算法

```c
void ReverseList(LinkList L)
{
    LinkList p, q;
    p = L->next;
    L->next = NULL;
    while (p != NULL)
    {
        q = p->next;
        p->next = L->next;
        L->next = p;
        p = q;
    }
}
```

###### 应用实例——求集合差

```c
void Difference(LinkList LA, LinkList LB)
{
    Node *pre, *p, *q, *r;
    pre = LA;
    p = LA->next;
    while (p != NULL)
    {
        q = LB->next;
        while (q != NULL && q->data != p->data)
            q = q->next;
        if (q != NULL)
        {
            r = p;
            pre->next = p->next;
            p = p->next;
            free(r);
        }
        else
        {
            pre = p;
            p = p->next;
        }
    }
}
```

###### 单链表总结

- 访问单链表 L 中某结点第 i 结点或值为 e 结点时，必须从头开始。

- 表尾控制条件：当前结点 p->next == NULL

- 在处理过程中始终需要维持当前指针 p 与前驱指针 pre 的关系。


##### 循环链表

![image-20220122231030920](https://s2.loli.net/2022/01/22/1xpHjmXsLAkQcao.png)

###### 示例

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct
{
    int data;
    struct Node *next;
} Node, *LinkList;

void InitList(LinkList *L)
{
    *L = (LinkList *)malloc(sizeof(Node));
    (*L)->next = *L;
}

int main()
{
    LinkList L, p;
    int i;
    InitList(&L);
    for (i = 1; i <= 3; i++)
    {
        p = (Node *)malloc(sizeof(Node));
        p->data = i;
        p->next = L->next;
        L->next = p;
    }
    p = L->next;
    for (i = 1; i <= 6; i++)
    {
        printf("%d\t", p->data);
        if (p->next == L)
        {
            printf("已到达表尾\n");
            p = p->next;
        }
        p = p->next;
    }
    return 0;
}

/*
输出：
3	2	1	已到达表尾
3	2	1	已到达表尾
*/
```

###### 循环单链表的合并算法

```c
//采用头指针
LinkList merge_1(LinkList LA, LinkList LB)
{
    Node *p, *q;
    p = LA;
    q = LB;
    while (p->next != LA)
        p = p->next;
    while (q->next != LB)
        q = q->next;
    q->next = LA;
    p->next = LB->next;
    free(LB);
    return LA;
}
//采用尾指针
LinkList merge_2(LinkList RA, LinkList RB)
{
    Node *p, *tmp;
    p = RA->next;
    tmp = RB->next;
    RA->next = tmp->next;
    free(RB->next);
    RB->next = p;
    return RB;
}
```

###### 循环单链表总结

- 特点：首尾相接，可从当前结点遍历所有结点
- 空表判断条件：L->next == L
- 表尾控制条件：当前结点 p->next == L

##### 双向链表

![image-20220123134739873](https://s2.loli.net/2022/01/23/Es3XMVmqWD92hfv.png)

```c
typedef struct DNode
{
    int data;
    struct DNode *prior, *next;
} DNode, *DoubleList;
```

###### 前插操作

```c
int DlinkIns(DoubleList L, int i, int e)
{
    DNode *s, *p;
    int k;
    p = L;
    k = 0;
    while (p != NULL && k < i - 1)
    {
        p = p->next;
        k++;
    }
    if (!p)
    {
        printf("插入位置不合理！");
        return 0;
    }
    s = (DNode *)malloc(sizeof(DNode));
    if (s)
    {
        s->data = e;
        s->prior = p->prior;
        p->prior->next = s;
        s->next = p;
        p->prior = s;
        return 1;
    }
    else
        return 0;
}
```

###### 删除操作

```c
int DlinkDel(DoubleList L, int i, int *e)
{
    DNode *p;
    int k;
    p = L;
    k = 0;
    while (p->next != NULL && k < i - 1)
    {
        p = p->next;
        k++;
    }
    if (k != i - 1)
    {
        printf("删除结点的位置i不合理！");
        return 0;
    }
    *e = p->data;
    p->prior->next = p->next;
    p->next->prior = p->prior;
    free(p);
    return 1;
}
```

###### 循环双向链表

![image-20220123140536543](https://s2.loli.net/2022/01/23/eREGaBlKvLCqZMs.png)

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct DNode
{
    int data;
    struct DNode *prior, *next;
} DNode, *DoubleList;

void DlinkInit(DoubleList *L)
{
    *L = (DoubleList *)malloc(sizeof(DNode));
    (*L)->prior = (*L)->next = *L;
}

int main()
{
    DoubleList L, p, r;
    int i;
    DlinkInit(&L);
    r = L;
    for (i = 1; i <= 3; i++)
    {
        p = (DNode *)malloc(sizeof(DNode));
        p->data = i;
        r->next->prior = p;
        p->next = r->next;
        r->next = p;
        p->prior = r;
    }
    while (p->next != L)
    {
        printf("%d\t", p->data);
        p = p->next;
    }
    printf("%d\t已到表尾\n", p->data);
    while (p->prior != L)
    {
        printf("%d\t", p->data);
        p = p->prior;
    }
    printf("%d\t已到表头\n", p->data);
    return 0;
}

/*
输出：
3	2	1	已到表尾
1	2	3	已到表头
*/
```

###### 双向链表总结

- 双向链表可从两个方向访问任一结点，快速，方便。
- 可以推广至多重链表，其本质上是用空间换时间。

##### 静态链表

由于有些语言并未提供指针类型，我们可以采用数组模拟实现链表，静态模拟动态。

![image-20220123172702494](https://s2.loli.net/2022/01/23/cDiN4Thm5kOXR1f.png)

```c
#define MAXSIZE 100
typedef struct 
{
    int data;
    int cursor;
}Component,StaticList[MAXSIZE];
```

###### 初始化

```c
void initial(StaticList space, int *av)
{
    space[0].cursor = -1;
    for (int k = 1; k < MAXSIZE - 1; k++)
        space[k].cursor = k + 1;
    space[MAXSIZE].cursor = -1;
    *av = 1;
}
```

###### 分配结点

```c
//从备用链表中分配结点给使用者
int getnode(StaticList space, int *av)
{
    int i;
    i = *av;
    *av = space[*av].cursor;
    return i;
}
```

###### 回收结点

```c
//备用链表回收空闲结点
void freenode(StaticList space, int *av, int k)
{
    space[k].cursor = *av;
    *av = k;
}
```

##### 线性表的链式存储总结

- 存储结构：用一组任意配置的单元存储，用指针维持结点线性关系

- 类型：

  - 动态链表

    单链表，双向链表，多重链表，循环链表

  - 静态链表

- 特点：适合动态变化长度操作，需有存储指针附加代价

#### 线性表应用——一元多项式表示及相加

一元多项式数学表示：
$$
P_n(x)=p_0+p_1x^{e_1}+p2x^{e_2}+…+p_nx^{e_n}
$$
其用线性表表示：
$$
P=(p_0,p_1,p_2,…,p_n)
$$
两个一元多项式的相加：(设m<n)
$$
P_n(x)+Q_m(x)=R_n(x)
$$

$$
R=(p_0+q_0,p_1+q_1,p_2+q_2,…,p_m+q_m,p_{m+1},…,p_n)
$$

##### 链式存储

只存储非零系数项和指数项。

所有指数相同的项的对应系数相加，若和不为零则构成“和多项式”中的一项，所有指数不相同的项均按升幂复抄到“和多项式”中。

###### 完整代码

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Polynode
{
    int coef; //系数
    int exp;  //指数
    struct Polynode *next;
} Polynode, *Polylist;

Polylist PolyCreate();
Polylist AscSort(Polylist L);
Polylist PolyAdd(Polylist LA, Polylist LB);
void PolyPrint(Polylist L);

int main()
{
    Polylist LA, LB, LC;
    LA = AscSort(PolyCreate());
    printf("f(x)_A=");
    PolyPrint(LA);
    LB = AscSort(PolyCreate());
    printf("f(x)_B=");
    PolyPrint(LB);
    LC = PolyAdd(LA, LB);
    printf("f(x)_C=f(x)_A+f(x)_B=");
    PolyPrint(LC);
    return 0;
}

Polylist PolyCreate()
{
    Polynode *head, *rear, *s;
    int c, e;
    head = (Polynode *)malloc(sizeof(Polynode));
    rear = head;
    scanf("%d %d", &c, &e);
    while (c != 0)
    {
        s = (Polynode *)malloc(sizeof(Polynode));
        s->coef = c;
        s->exp = e;
        rear->next = s;
        rear = s;
        scanf("%d %d", &c, &e);
    }
    rear->next = NULL;
    return head;
}

Polylist AscSort(Polylist L)
{
    Polynode *p, *q;
    int ctmp, etmp;
    for (p = L->next; p != NULL; p = p->next)
        for (q = p->next; q != NULL; q = q->next)
            if (p->exp > q->exp)
            {
                etmp = p->exp;
                p->exp = q->exp;
                q->exp = etmp;
                ctmp = p->coef;
                p->coef = q->coef;
                q->coef = ctmp;
            }
            else if (p->exp == q->exp)
            {
                p->coef = p->coef + q->coef;
                p->next = q->next;
            }
    return L;
}

Polylist PolyAdd(Polylist LA, Polylist LB)
{
    Polynode *a, *b, *p;
    a = LA->next;
    b = LB->next;
    p = LA;
    while (a != NULL && b != NULL)
    {
        if (a->exp == b->exp)
        {
            a->coef = a->coef + b->coef;
            p->next = a;
            p = p->next;
            a = a->next;
            b = b->next;
        }
        else if (a->exp < b->exp)
        {
            p->next = a;
            p = p->next;
            a = a->next;
        }
        else
        {
            p->next = b;
            p = p->next;
            b = b->next;
        }
    }
    if (a != NULL)
        p->next = a;
    else
        p->next = b;
    return LA;
}

void PolyPrint(Polylist L)
{
    Polynode *p;
    p = L->next;
    while (p != NULL)
    {
        if (p->coef > 0 && p->exp == 0 && p == L->next) //系数大于0，指数0，首项
            printf("%d", p->coef);
        else if (p->coef > 0 && p->exp == 0 && p != L->next) //系数大于0，指数0，非首项
            printf("+%d", p->coef);
        else if (p->coef < 0 && p->exp == 0) //系数小于0，指数0
            printf("%d", p->coef);
        else if (p->coef == 1 && p->exp == 1 && p == L->next) //系数1,指数为1，首项
            printf("x");
        else if (p->coef == 1 && p->exp == 1 && p != L->next) //系数1,指数为1，非首项
            printf("+x");
        else if (p->coef == 1 && p->exp != 0 && p->exp != 1 && p == L->next) //系数1，指数非0非1，首项
            printf("x^%d", p->exp);
        else if (p->coef == 1 && p->exp != 0 && p->exp != 1 && p != L->next) //系数1，指数非0非1，非首项
            printf("+x^%d", p->exp);
        else if (p->coef == -1 && p->exp == 1) //系数-1,指数1
            printf("-x");
        else if (p->coef == -1 && p->exp != 0 && p->exp != 1) //系数-1，指数非0非1
            printf("-x^%d", p->exp);
        else if (p->coef > 0 && p->exp == 1 && p == L->next) //系数大于0，指数1，首项
            printf("%dx", p->coef);
        else if (p->coef > 0 && p->exp == 1 && p != L->next) //系数大于0，指数1，非首项
            printf("+%dx", p->coef);
        else if (p->coef < 0 && p->exp == 1) //系数小于0，指数1
            printf("%dx", p->coef);
        else if (p->coef > 0 && p->exp != 0 && p->exp != 1 && p == L->next) //系数大于0,指数非0非1，首项
            printf("%dx^%d", p->coef, p->exp);
        else if (p->coef > 0 && p->exp != 0 && p->exp != 1 && p != L->next) //系数大于0,指数非0非1，非首项
            printf("+%dx^%d", p->coef, p->exp);
        else if (p->coef < 0 && p->exp != 0 && p->exp != 1) //系数小于0,指数非0非1
            printf("%dx^%d", p->coef, p->exp);
        p = p->next;
    }
    printf("\n");
}
```

运行截图：

![image-20220124120646691](https://s2.loli.net/2022/01/24/GDJ9I5xcQrAbelV.png)

#### 顺序表与链表的综合比较

##### 基于空间的考虑

- **顺序表**存储是静态分配，程序执行前须定义存储规模。
- **静态链表**是静态分配，同时存在若干个结点类型相同的链表，可共享空间。
- **动态链表**存储是动态分配，只要内存空间尚有空闲就不会产生溢出。

顺序表虽然存储密度更高，但灵活度不够。

{{< admonition info >}}

存储密度，指结点数据本身所占的存储量和整个结点结构所占的存储量之比。顺序表存储密度为1，而链表小于1

{{< /admonition>}}

当线性表的长度变化较大，难以估计其存储规模时，采用动态链表作为存储结构较好。

##### 基于时间的考虑

**顺序表**是一种随机存储结构，适合大量元素的**查找**。

**链表**的插入删除只用修改挂链，不需要移动元素，所以适合**动态变化**。

若表的插入和删除主要发生在表的**首尾两端**，则宜采用**尾指针**表示的**单循环链表**。

##### 基于语言的考虑

有些语言提供指针类型，因此可以用链表形式。有些语言**不提供指针类型**，只能用**静态链表**来模拟，静态链表在存储分配上有所不足，但也能像动态链表一样灵活插入和删除。

##### 存储方式比较

![image-20220124161044353](https://s2.loli.net/2022/01/24/gUAenr7zIkqEpJx.png)

#### 线性表例题

##### 顺序表分奇偶

已知顺序表 L 中的数据元素类型为 int。设计算法将其调整为左右两部分，左边的元素（即排在前面的）均为奇数，右边所有元素（即排在后面的）均为偶数，并要求算法的时间复杂度为 O(n)，空间复杂度为 O(1)

```c
#define MAXSIZE 100
typedef struct
{
    int elem[MAXSIZE];
    int last;
} SeqList;
```

WP：

```c
AdjustSeqlist(SeqList *L)
{
    int i, j, tmp;
    i = 0;
    j = L->last;
    while (i < j)
    {
        while (L->elem[i] % 2 != 0)
            i++;
        while (L->elem[j] % 2 == 0)
            j--;
        if (i < j)
        {
            tmp = L->elem[i];
            L->elem[i] = L->elem[j];
            L->elem[j] = tmp;
        }
    }
}
```

##### 二进制数加一运算

建立了一个带头结点的线性链表，用以存放输入的二进制数，链表中每个结点的data域存放一个二进制位。并在此链表上实现对二进制数加1的运算。

```c
typedef struct
{
    int data;
    struct Node *next;
} Node, *LinkList;
```

WP：

```c
void BinAdd(LinkList L)
{
    Node *q, *r, *tmp, *s;
    q = L->next;
    r = L;
    while (q != NULL) //找到最后一个值域0的结点，将值赋为1
    {
        if (q->data == 0)
            r = q;
        q = q->next;
    }
    if (r != L)
        r->data = 1;
    else
    {
        tmp = r->next;
        s = (Node *)malloc(sizeof(Node));
        s->data = 1; //头插
        s->next = tmp;
        r->next = s;
        r = s;
    }
    r = r->next;
    while (r != NULL) //进位结点后的结点值域置0
    {
        r->data = 0;
        r = r->next;
    }
}
```


