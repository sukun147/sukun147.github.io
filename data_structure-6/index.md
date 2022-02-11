# 数据结构学习(六)


<!--more-->

[本系列学习笔记链接](/categories/数据结构/)

# 数据结构——用C语言描述(四)

## 非线性结构

### 树

#### 树的定义与基本术语

##### 树的基本概念

树（Tree）是n (n≥0）个结点的有限集合T。当n=0时，称为空树；当n>0时，该集合满足如下条件:

- 必有一个称为根（root）的特定结点，它没有直接前驱，但有零个或多个直接后继
- 其余 n-1 个结点可以划分成 m (m≥0）个互不相交的有限集 T1，T2，T3,...，Tm，其中 Ti 又是一棵树，称为根 root 的子树。每棵子树的根结点有且仅有一个直接前驱，但有零个或多个直接后继。

![image-20220202235214918](https://s2.loli.net/2022/02/02/ox4ML3wP6fgavuc.png)

##### 树的图解表示

- 倒置树结构法（树形表示法）
- 文氏图表示法（嵌套集合形式）
- 广义表形式（嵌套括号表示法）
- 凹入表示法

##### 树的相关术语

- 结点：包括一个数据元素及若干指向其它结点的分支信息。
- 结点的度：结点的子树个数
- 叶结点：度为 0 的结点，即无后继的结点，也称为终端结点
- 分支结点：度不为 0 的结点称为非终端结点
- 树的度：树中所有结点的度的最大值
- 结点的层次：从根结点开始定义根结点的层次为 1，根的直接后继的层次为 2，依此类推
- 树的高度：树中所有结点的层次的最大值
- 有序树：各子树之间有先后次序
- 森林：多颗互不相交的树的集合
- 孩子结点：直接后继结点
- 双亲结点：直接前驱结点
- 兄弟结点：同一双亲结点的孩子结点之间互称兄弟结点
- 祖先结点：从根到该结点路径上的所有结点
- 孙子结点：直接后继和间接后继

##### 树的抽象数据类型

数据对象 D：该集合中的所有元素具有相同的特性

结构关系 R：若 D 为空集，则为空树；若 D 中仅含有一个数据元素，则 R 为空集，否则 R={H}，H 是如下的二元关系:

- D中存在唯一的根元素 root，在关系 H 中无前驱
- 除 root 以外，D 中每个结点在关系 H 下都有且仅有一个前驱

操作集合：

1. 初始化树`lnitTree(Tree)`
2. 销毁树`DestoryTree(Tree)`
3. 创建树`CreateTree(Tree)`
4. 判空`TreeEmpty(Tree)`
5. 求根`Root(Tree)`
6. 求双亲`Parent(Tree，x)`
7. 找 x 结点的第一个孩子`FirstChild(Tree，x)`
8. 找 x 结点的下一个兄弟`NextSibling(Tree，x)`
9. 插入孩子结点`InsertChild(Tree，p, Child)`
10. 删除孩子结点`DeleteChild(Tree，p, i)`
11. 遍历树`TraverseTree(Tree，Visit())`

#### 二叉树

##### 二叉树的定义与基本操作

二叉树（Binary Tree）：

- 每个结点的度都不大于 2
- 每个结点的孩子结点次序不能任意颠倒

10 种操作：

1. 初始化`lnitiate(bt)`
2. 创建`Create(bt)`
3. 销毁`Destory(bt）`
4. 判空`Empty(bt)`
5. 求根`Root(bt)`
6. 求双亲结点`Parent(bt, x)`
7. 求左孩子`LeftChild(bt，x)`
8. 求右孩子`RightChild(bt，x)`
9. 遍历`Traverse(bt)`
10. 清空`Clear(bt)`

##### 二叉树性质

1. 在二叉树的第 i 层的最大结点数：
   $$
   2^{i-1}
   $$

2. 深度为 k 的二叉树的最大结点数：
   $$
   \sum_{i=1}^{k}2^{i-1}=2^{k-1}
   $$

3. 对任意一棵二叉树，若终端结点数为 n0，而其度数为 2 的结点数为 n2，则 n0= n2+1

4. 满二叉树：有最大结点数，即每层结点都是满的

5. 完全二叉树：深度为 k，结点数为 n 的二叉树，结点 1~n 的位置序号分别与满二叉树的结点 1~n 的位置序号一一对应

   {{< admonition >}}

   满二叉树必为完全二叉树，而完全二叉树不一定是满二叉树

   {{< /admonition >}}

6. 具有 n 个结点的完全二叉树的深度为
   $$
   [\log_2n]+1\\\
   eg:[\log_27]+1=3
   $$

7. 对于有 n 个结点的完全二叉树，按照从上到下和从左到右的顺序编号结点，则序号 i 结点有以下关系：

   - 若 i =1，则 i 无双亲结点；

     若 i>1，则 i 的双亲结点为
     $$
     [\frac{i}{2}]
     $$

   - 若 2*i > n，则 i 无左孩子；

     若 2\*i ≤ n，则 i 结点的左孩子结点为 **2\*i** 

   - 若 2*i+1 > n，则 i 无右孩子；

     若 2\*i+1 ≤ n，则 i 的右孩子结点为 **2\*i+1**

##### 二叉树存储结构

###### 顺序存储

以完全二叉树的形式来存储数据元素，但对于一般二叉树，这将造成极大的空间浪费。

{{< admonition success 举个栗子>}}

单支树是其极端情况，例如一颗深度 4 的单支树，我们需要用 15 个结点空间来存储这 4 个结点。

{{< /admonition >}}

###### 链式存储

我们采用二叉链表来存储二叉树，其中每个结点需要有三个域：
$$
\begin{array}{|c|c|c|}
        \hline
        LChild & Data & RChild\\\\\hline
\end{array}
$$

```c
typedef struct Node
{
    char data;
    struct Node *LChild, *RChild;
} BiTNode, *BiTree;
```

有时，为了寻找其双亲结点，我们还会增设`Parent`域形成三叉链表
$$
\begin{array}{|c|c|c|c|}
        \hline
        LChild & Data & Parent & RChild\\\\\hline
\end{array}
$$

#### 二叉树的遍历与线索化

##### 二叉树的遍历

含义：指按一定规律对二叉树中的每个结点访问且仅访问一次。

目的：将非线性结构经过遍历得到结点访问序列，也就是线性化过程

按照先左后右，相对于根的顺序，有以下三种方式：

1. DLR 先序遍历
2. LDR 中序遍历
3. LRD 后序遍历

对于下面这个二叉树，三种遍历方法有不同的结果

![image-20220204124302112](https://s2.loli.net/2022/02/04/1BwfeO2dWqu6Upr.png)

###### DLR的递归定义

```c
void PreOrder(BiTree root)
{
    if (root)
    {
        Visit(root->data);
        PreOrder(root->LChild);
        PreOrder(root->RChild);
    }
}
```

结果：ABDFGCEH

###### LDR的递归定义

```c
void InOrder(BiTree root)
{
    if (root)
    {
        InOrder(root->LChild);
        Visit(root->data);
        InOrder(root->RChild);
    }
}
```

结果：BFDGACEH

###### LRD的递归定义

```c
void PostOrder(BiTree root)
{
    if (root)
    {
        PostOrder(root->LChild);
        PostOrder(root->RChild);
        Visit(root->data);
    }
}
```

结果：FGDBHECA

##### 遍历算法应用

###### 输出二叉树中的结点（先序遍历）

```c
void PreOrder(BiTree root)
{
    if (root)
    {
        printf("%c",root->data);
        PreOrder(root->LChild);
        PreOrder(root->RChild);
    }
}
```

###### 输出二叉树中的叶子结点（先序遍历）

```c
void PreOrder(BiTree root)
{
    if (root)
    {
        if (root->LChild == NULL && root->RChild == NULL)
            printf("%c", root->data);
        PreOrder(root->LChild);
        PreOrder(root->RChild);
    }
}
```

###### 统计叶子结点数目（后序遍历）

```c
//方法一
int LeafCount = 0;
void leaf_1(BiTree root)
{
    if (root)
    {
        leaf_1(root->LChild);
        leaf_1(root->RChild);
        if (root->LChild == NULL && root->RChild == NULL)
            LeafCount++;
    }
}
//方法二
int leaf_2(BiTree root)
{
    int LeafCount;
    if (root)
        LeafCount = 0;
    else if (root->LChild == NULL && root->RChild == NULL)
        LeafCount = 1;
    else
        LeafCount = leaf_2(root->LChild) + leaf_2(root->RChild);
    return LeafCount;
}
```

###### 建立二叉链表存储的二叉树

```c
void CreateBiTree(BiTree *bt)
{
    char ch;
    ch = getchar();
    if (ch == '.')
        *bt = NULL;
    else
    {
        *bt = (BiTree)malloc(sizeof(BiTNode));
        (*bt)->data = ch;
        CreateBiTree(&((*bt)->LChild));
        CreateBiTree(&((*bt)->RChild));
    }
}
```

###### 按树状横向打印二叉树（中序遍历）

```c
void PrintTree(BiTree root, int nLayer)
{
    if (!root)
        return;
    PrintTree(root->RChild, nLayer + 1);
    for (int i = 0; i < nLayer; i++)
        printf(" ");
    printf(" %c\n", root->data);
    PrintTree(root->LChild, nLayer + 1);
}
```

##### 基于栈的递归消除

在[数据结构学习(三)#栈的应用与递归](/data_structure-3/#栈的应用与递归)中已经简单地介绍了递归的一系列问题，但并没有介绍基于栈的递归消除方法。本节将讲解此部分内容。

###### 中序遍历的非递归算法（直接实现栈操作）

```c
void Inorder(BiTree root)
{
    int top = 0;
    BiTNode *p;
    p = root;
    /**************
    L1: //遍历左子树
        if (p)
        {
            top += 2;
            if (top > m) //溢出处理
                return;
            s[top - 1] = p; //本层参数进栈
            s[top] = L2;    //返回地址进栈
            p = p->LChild;  //给下层参数赋值
            goto L1;
        }
    L2: //访问根
        Visit(p->data);
        top += 2;
        if (top < m)
            return;       //溢出处理
        s[top - 1] = p; //遍历右子树
        s[top] = L3;
        p = p->RChild;
        goto L2;
    L3: //退层
        if (top)
        {
            addr = s[top];
            p = s[top - 1]; //取出返回地址
            top -= 2;       //退出本层参数
            goto addr;      //恢复上层断点
        }
    ***************/
    do
    {
        while (p)
        {
            if (top > m)
                return;
            top++;
            s[top] = p;
            p = p->LChild;
        }
        if (top)
        {
            p = s[top];
            top--;
            Visit(p->data);
            p = p->RChild;
        }
    } while (p || top);
}
```

###### 中序遍历的非递归算法（调用栈操作）

```c
void InOrder(BiTree root)
{
    SeqStack S;
    InitStack(&S);
    BiTNode *p;
    p = root;
    while (p || !IsEmpty(S))
    {
        if (p) //进栈，遍历左子树
        {
            push(&S, p);
            p = p->LChild;
        }
        else //退栈，访问根结点，遍历右子树
        {
            pop(&S, &p);
            Visit(p->data);
            p = p->RChild;
        }
    }
}
```

##### 线索二叉树

通过前面介绍的遍历运算我们将一颗二叉树进行了线性化，但无法直接得到结点在遍历序列中的前驱、后继结点的信息，由此，我们想到两种办法解决这个问题：

1. 将二叉树遍历一遍，在遍历过程中便可得结点的前驱和后继，但这种动态访问浪费时间。
2. 充分利用二叉链表中的空链域，将遍历过程中结点的前驱、后继信息保存下来。

{{< admonition question>}}

n 个结点的二叉树中有多少个空链域？

{{< /admonition >}}

{{< admonition success 答案>}}

共有 2n 个链域，其中 n-1 个非空，故有 n+1 个空链域

{{< /admonition >}}

为了区分孩子结点、前驱结点、后继结点，我们应新增设两个标志域（`Ltag`、`Rtag`）：
$$
\begin{array}{|c|c|c|c|c|}
        \hline
        Ltag & LChild & Data & Rtag & RChild\\\\\hline
\end{array}
$$
Ltag：

1. Ltag=0——LChild域指示结点的左孩子
2. Ltag=1——LChild域指示结点的遍历前驱

Rtag：

1. Rtag=0——RChild域指示结点的右孩子
2. Rtag=1——RChild域指示结点的遍历后继

相关术语：

- 线索：指向前驱和后继结点的指针
- 线索链表：具有线索的二叉链表
- 线索化：对二叉树遍历并加上线索（修改空链域）的过程
- 线索二叉树：线索化了的二叉树

###### 中序遍历线索化

```c
void Inthread(BiTree root) //当前结点root，前驱结点pre
{
    BiTNode *pre = NULL;
    if (root)
    {
        Inthread(root->LChild); //线索化左子树
        if (!root->LChild)
        {
            root->Ltag = 1;
            root->LChild = pre; //前驱线索
        }
        if (pre && !pre->RChild)
        {
            pre->Rtag = 1;
            pre->RChild = root; //后继线索
        }
        pre = root;
        Inthread(root->RChild); //线索化右子树
    }
}
```

###### 在**中序**线索二叉树中找**前驱**结点

```c
void InPre(BiTNode *p, BiTNode *pre) //找p的中序前驱pre
{
    BiTNode *q;
    if (p->Ltag)
        pre = p->LChild;
    else
    {
        for (q = p->LChild; !q->Rtag; q = q->RChild) //在p的左子树中查找“最右下端”结点
            pre = q;
    }
}
```

###### 在**中序**线索二叉树中找**后继**结点

```c
void InSucc(BiTNode *p, BiTNode *succ) //找p的中序后继succ
{
    BiTNode *q;
    if (p->Rtag)
        succ = p->RChild;
    else
    {
        for (q = p->RChild; q->Ltag == 0; q = q->LChild) //在p的右子树中查找“最左下端”结点
            succ = q;
    }
}
```

###### 在**先序**线索二叉树中找**后继**结点

![image-20220207101351220](https://s2.loli.net/2022/02/07/nM8GZFOY5RiWklz.png)

```c
if (!p->Ltag)
    succ = p->LChild;
else
    succ = p->RChild;
```

###### 在**后序**线索二叉树中找**前驱**结点

![image-20220207101417881](https://s2.loli.net/2022/02/07/PdyqnkNKwziGLe9.png)

```c
if(!p->Rtag)
    pre=p->RChild;
else
    pre=p->LChild;
```

###### 在**先序**线索二叉树中找**前驱**结点

1. P 为 F 的左子，则 P 的先序前驱为 F
2. P 为 F 的右子，且 F 无左子，则 P 的先序前驱为 F
3. P 为 F 的右子，且 F 有左子，则 P 的先序前驱为 F 的左子树的“最右下端”

###### 在**后序**线索二叉树中找**后驱**结点

1. P 为 F 的左子，则 P 的后序后继为 F 的右子树的“最左下端”
2. P 为 F 的左子，且 F 无右子，则 P 的后序后继为 F
3. P 为 F 的右子，则 P 的后序后继为 F 

##### 由遍历序列确定二叉树

| 两种遍历序列的组合 | 是否能唯一确定二叉树 |
| ------------------ | -------------------- |
| 先序+中序          | 是                   |
| 后序+中序          | 是                   |
| 先序+后序          | 否                   |

我们观察前序遍历和中序遍历可知，在前序序列中第一个结点为根结点，而根结点把中序序列分为了左子树和右子树两部分。

{{< admonition success 举个栗子>}}

已知一棵二叉树的前序序列为`18 14 7 3 11 22 35 27`，中序序列为`3 7 11 14 18 22 27 35`。

则该二叉树为

![image-20220207104840438](https://s2.loli.net/2022/02/07/xY2b3fJOrGdpqcR.png)

{{< /admonition >}}

同理，后序序列中的最后一个结点为根结点，而根结点把中序序列分为了左子树和右子树两部分。

这里给出由先序和中序序列还原二叉树并横向树状打印的代码

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#define m 25

typedef struct Node
{
    char data;
    struct Node *LChild, *RChild;
} BiTNode, *BiTree;

void PrintTree(BiTree root, int nLayer) //横向打印
{
    if (!root)
        return;
    PrintTree(root->RChild, nLayer + 1);
    for (int i = 0; i < nLayer; i++)
        printf(" ");
    printf(" %c\n", root->data);
    PrintTree(root->LChild, nLayer + 1);
}

BiTree Create(char *pre, char *in, int cnt)
{
    BiTree root;
    int inPtr, LeftCnt, RightCnt;
    inPtr = LeftCnt = RightCnt = 0;
    root = (BiTree)malloc(sizeof(BiTNode));
    if (!root)
        return;
    root->data = pre[0];
    root->LChild = NULL;
    root->RChild = NULL;
    if (cnt == 1)
        return root;
    while (in[++inPtr] != pre[0]) //找根结点
        ;
    LeftCnt = inPtr;
    RightCnt = cnt - LeftCnt - 1;
    root->LChild = Create(&pre[1], in, LeftCnt);
    root->RChild = Create(&pre[LeftCnt + 1], &in[LeftCnt + 1], RightCnt);
    return root;
}

int main()
{
    BiTree root;
    char pre[m], in[m];
    printf("请输入前序序列:");
    gets(pre);
    printf("请输入中序序列:");
    gets(in);
    root = Create(pre, in, strlen(pre));
    PrintTree(root, 0);
    return 0;
}
/*
输入：
请输入前序序列:abdecfg
请输入中序序列:dbeafcg
输出：
   g
  c
   f
 a
   e
  b
   d
*/
```

##### 总结

遍历运算是基础，线索是遍历应用，递归机制支持遍历，递归是技术重点。

#### 树、森林与二叉树的关系

##### 树的存储结构

###### 双亲表示法

用一组连续的空间来存储树中的结点，在保存每个结点的同时附设一个指示器来指示其双亲结点在表中的位置

![image-20220207132214199](https://s2.loli.net/2022/02/07/U6kPYraeNfq4y1J.png)
$$
\begin{array}{|c|c|}
        \hline
        Data & Parent\\\\\hline
        1 & -1\\\\\hline
        2 & 0\\\\\hline
        3 & 0\\\\\hline
        4 & 1\\\\\hline
        5 & 1\\\\\hline
        6 & 1\\\\\hline
        7 & 2\\\\\hline
\end{array}
$$

```c
#define MAX 100
typedef struct TNode
{
    int data;
    int parent;
} TNode;

typedef struct
{
    TNode terr[MAX];
    int nodenum; //结点数
} parentTree;
```

- 优点：

  容易查找双亲结点

- 缺点：

  求孩子结点需要遍历整个数组

###### 孩子表示法

每个结点的孩子结点构成一个单链表，即 n 个结点共有 n 个孩子链表。n 个结点的数据和 n 个孩子链表的头指针组成一个顺序表。

```c
typedef struct ChildNode //孩子链表
{
    int Child; //孩子结点在线性表中的位置
    struct ChildNode *next;
} ChildNode;

typedef struct //顺序表
{
    int data;
    ChildNode *FirstChild; //头指针
} DataNode;

typedef struct //树
{
    DataNode nodes[MAX]; //顺序表
    int root, num;       //根节点位置，结点数
} ChildTree;
```

###### 孩子兄弟表示法（最重要）

链表中每个结点设有两个链域，分别指向该结点的第一个孩子结点和下一个兄弟（右兄弟）结点。事实上，这是一种二叉链表。
$$
\begin{array}{|c|c|c|}
        \hline
        Data & FirstChild & NextSibling\\\\\hline
\end{array}
$$

```c
typedef struct CSNode
{
    int data;
    struct CSNode *FirstChild, *NextSibling;
} CSNode, *CSTree;
```

![image-20220207151501704](https://s2.loli.net/2022/02/07/3BKf968hdW1Mzg2.png)

##### 树、森林与二叉树的相互转换

以下步骤我们不必记忆，我们需要做的是使用**孩子兄弟表示法**来转换二叉树，也就是用存储来对应

###### 树转换为二叉树

步骤：

1. 树中所有相邻兄弟之间架一条连线
2. 对树中的每个结点，只保留其与第一个孩子结点之间的连线，删去其与其他孩子结点之间的连线
3. 以树的根结点为轴心，将整棵树顺时针旋转一定的角度，使之结构层次分明

![image-20220207151844726](https://s2.loli.net/2022/02/07/dXrU5WzQAp8jV6n.png)

易看出，在二叉树中，其左孩子为原来的孩子结点，而右孩子为原来的兄弟结点。

###### 森林转换为二叉树

步骤：

1. 将森林中的每棵树转换成相应的二叉树
2. 从第二棵二叉树开始，依次把后一棵二叉树的根结点作为前一棵二叉树根结点的右孩子

###### 二叉树还原为树或森林

步骤：

1. 若某结点是其双亲的左孩子，则把该结点的右孩子、右孩子的右孩子...都与该结点的双亲结点用线连起来
2. 删掉原二叉树中所有双亲结点与右孩子结点的连线
3. 整理所得到的树或森林

![image-20220207153005922](https://s2.loli.net/2022/02/07/k3vA1ONCHKjIYzr.png)

##### 树与森林的遍历

###### 树的先根遍历

若树非空，则遍历方法为

1. 访问根结点
2. 从左到右，依次先根遍历根结点的每一棵子树

###### 树的后根遍历

若树非空，则遍历方法为

1. 从左到右依次后根遍历根结点的每一棵子树
2. 访问根节点

###### 森林的先序遍历

若森林非空，则遍历方法为：

1. 访问森林中第一棵树的根结点
2. 先序遍历第一棵树的根结点的子树森林
3. 先序遍历除去第一棵树之后剩余的树构成的森林

###### 森林的中序遍历

若森林非空，则遍历方法为：

1. 中序遍历森林中第一棵树的根结点的子树森林
2. 访问第一棵树的根结点
3. 中序遍历除去第一棵树之后剩余的树构成的森林

###### 森林的后序遍历

若森林非空，则遍历方法为：

1. 后序遍历森林中第一棵树的根结点的子树森林
2. 后序遍历除去第一棵树之后剩余的树构成的森林
3. 访问第一棵树的根结点

#### 哈夫曼树及其应用

##### 哈夫曼树

相关概念：

- 路径：从一个结点到另一个结点之间的分支序列

- 路径长度：从一个结点到另一个结点所经过的分支数目

- 树的路径长度：树中所有结点的路径长度之和为树的路径长度 PL

- 结点的权：给树的每个结点赋予一个具有某种实际意义的实数，我们称该实数为这个结点的权

- 带权路径长度：从树根到某一结点的路径长度与该结点的权的乘积

- 树的带权路径长度：树的带权路径长度为树中所有叶子结点的带权路径长度之和
  $$
  WPL=\sum_{i=1}^{n}w_i×l_i
  $$

易看出，二叉树中路径长度为 K 的结点至多有 2^k 个，故而
$$
结点 n 的路径长度=[\log_2n]\\\
前n项和=\sum_{i=1}^n[\log_2k]
$$
由此，我们可得推论：**完全二叉树具有最小路径长度的性质**，然而**哈夫曼树才具有最小带权路径长度的性质**。

###### 哈夫曼树定义

哈夫曼树又叫最优二叉树，它是由 n 个带权叶子结点构成的所有二叉树中带权路径长度 WPL 最短的二叉树。

###### 构建哈夫曼树

步骤：

1. 给定 n 个权值`{w1, w2, ..., wn}`,构造森林`F={T1,T2, ...., Tn}`，其中 Ti 为单根树，根的权值为 wi
2. 在森林 F 中选择两棵根结点权值最小的二叉树，作为一棵新二叉树的左、右子树，标记新二叉树的根结点权值为其左右子树的根结点权值之和
3. 从F中删除被选中的那两棵二叉树，同时把新构成的二叉树加人到森林F中
4. 重复2、3操作，直到森林中只含有一棵二叉树为止

![构建哈夫曼树](https://tse1-mm.cn.bing.net/th/id/R-C.fac99f3b2105c62f916ef1aa4abb66fc?rik=aQlZo%2b5YqASIqw&riu=http%3a%2f%2fimages.cnitblog.com%2fblog%2f311549%2f201309%2f19165659-26d3652273854bf38204fa06d552ce69.jpg&ehk=OLk%2bOZtE5C%2bYiiQdT7f23rjqBkOfGeOjwnSbCrtR2nE%3d&risl=&pid=ImgRaw&r=0)

###### 哈夫曼树类型定义

显然，在哈夫曼树中没有度为 1 的结点，并且一棵叶子树 n 的哈夫曼有 2×(n-1) 个结点，也就是说我们可以用此大小的一维数组来存储哈夫曼树。

而两两合并我们还需要存储其双亲信息和孩子结点信息，故而需要静态三叉链表
$$
\begin{array}{|c|c|c|c|}
        \hline
        Weight & Parent & LChild & RChild\\\\\hline
\end{array}
$$

```c
typedef struct
{
    int weight, parent, LChild, RChild;
} HTNode, HuffmanTree[M + 1];
```

###### 哈夫曼树算法实现

```c
void crt_huffmantree(HuffmanTree ht, int w[], int n) // w存放n个字符和指令的权值
{
    int m, i, s1, s2;
    m = 2 * n - 1;
    HuffmanTree *p;
    for (p = ht, i = 1; i <= n; i++, p++, w++)
    {
        (*p)->weight = *w;
        (*p)->parent = (*p)->LChild = (*p)->RChild = 0;
    }
    for (; i <= m; i++, p++)
        (*p)->weight = (*p)->parent = (*p)->LChild = (*p)->RChild = 0;
    for (i = n + 1; i <= m; i++)
    {
        select(ht, i - 1, &s1, &s2); //选择parent为0且weight最小的两个结点, 其序号分别赋值给s1,s2
        ht[s1].parent = i;
        ht[s2].parent = i;
        ht[i].LChild = s1;
        ht[i].RChild = s2;
        ht[i].weight = ht[s1].weight + ht[s2].weight;
    }
}
```

##### 哈夫曼编码

###### 哈夫曼编码概念

- 前缀码：如果在一个编码系统中，任一个编码都不是其他任何编码的前缀（最左子串），则称该编码系统中的编码是前缀码
- 哈夫曼编码：对一棵有 n 个叶子的哈夫曼树，若规定左分支赋 0，右分支赋 1，可得到对应叶子节点的哈夫曼编码

哈夫曼编码特性：

- 哈夫曼编码是前缀码
- 哈夫曼编码是最优前缀码——能使各种报文（由这 n 种字符构成的文本）对应的二进制串的平均长度最短

###### 哈夫曼编码作用

$$
\begin{array}{|c|c|c|c|c|c|c|c|}
        \hline
        指令 & I_1 & I_2 & I_3 & I_4 & I_5 & I_6 & I_7\\\\\hline
        使用频率p_i & 0.40 & 0.30 & 0.15 & 0.05 & 0.04 & 0.03 & 0.03\\\\\hline
\end{array}
$$

对于上表所示指令频率的模型机，如果我们采用定长编码，我们每条指令都需要 3 位编码（2^3=8）。

为了减少程序的总位数，我们可以采用变长编码，而一个变长编码应该是前缀码，否则容易产生二义性。

而利用哈夫曼编码，我们可以设计出最优的前缀编码。

我们以每条指令的使用频率作为权值构造哈夫曼树，又规定左分支记为 1，右分支记为 0。

![image-20220211112154314](https://s2.loli.net/2022/02/11/WjDBysVOG3HALtx.png)

由此可得其哈夫曼编码为
$$
\begin{array}{|c|c|c|c|c|c|c|c|}
        \hline
        指令 & I_1 & I_2 & I_3 & I_4 & I_5 & I_6 & I_7\\\\\hline
        编码 & 0 & 10 & 110 & 11100 & 11101 & 11110 & 11111\\\\\hline
\end{array}
$$
{{< admonition success 举个栗子>}}

我们假设该模型机有1000条指令，其频率按前表所述，若采用定长编码，程序总位数=3×1000=3000。但若采用哈夫曼编码，程序总位数=1×400+2×300+3×150+5×150=2200。且采用哈夫曼编码的平均码长为
$$
\sum_{i=1}^nl_i=0.4×1+0.3×2+0.15×3+0.15×5=2.20
$$
由此可见哈夫曼编码的优越性。

{{< /admonition >}}

###### 哈夫曼编码算法实现

```c
...//已构造哈夫曼树，现从叶子结点到根逆向求每个字符或者指令的哈夫曼编码
    int start, c, f, j;
    hc = (HuffmanCode)malloc(n * sizeof(char *)); //分配n个字符编码的头指针
    char *cd = (char *)malloc(n * sizeof(char));
    cd[n - 1] = '\0';
    for (j = 0; j < n; j++)
    {
        start = n - 1;
        for (c = j; f = ht[j].parent; f != 0, c = f, f = ht[f].parent) //从叶子到根结点求编码
            if (ht[f].LChild == c)
                cd[--start] = '0';
            else
                cd[--start] = '1';
        hc[j] = (char *)malloc((n - start) * sizeof(char)); //为第i个指令编码分配空间
        strcpy(hc[j], &cd[start]);
    }
    free(cd);
}
```


