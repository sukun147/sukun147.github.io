# 数据结构学习(三)


<!--more-->

[TOC]

# 本系列学习笔记链接

- 第一章：[数据结构学习(一)](/data_structure-1)
- 第二章：[数据结构学习(二)](/data_structure-2)
- 第三章：[数据结构学习(三)](/data_structure-3)

# 数据结构——用C语言描述(三)

## 线性结构

### 限定性线性表——栈和队列

限定性：限制线性表插入和删除等运算的位置（只允许在端点位置操作）

#### 栈

##### 栈的定义与实现

- 栈的定义：把运算位置限制在表尾端
- 栈顶：允许运算端
- 栈底：不允许运算端
- 栈顶指示器：用来指示动态变化的栈顶位置
- 空栈：表中无任何元素
- 满栈：无法申请到栈区可用空间
- 栈的常见运算：进栈（入栈）、退栈（出栈）
- 上溢：栈已满还入栈
- 下溢：栈已空还出栈
- 栈的特性：后进先出（Last In First Out, LIFO）

对于栈 ADT：

数据元素：同一个数据对象的任意类型数据

关系：栈中数据元素之间是线性关系

有 7 种基本操作运算：

1. 初始化`InitStack(S)`
2. 清栈`ClearStack(S)`
3. 判空`IsEmpty(S)`
4. 判满`IsFull(S)`
5. 进栈`Push(S, x)`
6. 出栈`Pop(S, x)`
7. 读栈顶`GetTop(S, x)`

{{< admonition question >}}

按1，2，3的顺序进栈，则出栈顺序有哪些?

{{< /admonition >}}

{{< admonition success 答案 >}}

元素在进栈过程中也可以出栈，也就是并非所有元素全部进栈后再出栈。

因此，出栈顺序有：123，132，213，231，321

{{< /admonition >}}

###### 栈的顺序实现

- 用一组连续的存储单元依次存放自栈底到栈顶的数据元素
- 设一个位置指针`top`（栈顶指针）动态指示栈顶元素在顺序栈中的位置
- `top = -1` 表示空栈

```c
#define Stack_Size 50

typedef struct
{
    int elem[Stack_Size];
    int top;
} SeqStack;

void InitStack(SeqStack *S) { S->top = -1; }

bool IsEmpty(SeqStack S) { return S.top == -1; }

bool IsFull(SeqStack S) { return S.top == Stack_Size - 1; }

bool push(SeqStack *S, int x)
{
    if (IsFull(*S))
        return false;
    S->top++;
    S->elem[S->top] = x;
    return true;
}

bool pop(SeqStack *S, int *x)
{
    if (IsEmpty(*S))
        return false;
    *x = S->elem[S->top];
    S->top--;
    return true;
}

bool GetTop(SeqStack S, int *x)
{
    if (IsEmpty(S))
        return false;
    *x = S.elem[S.top];
    return true;
}
```

###### 顺序实现的两栈共享技术

为两个栈申请一个共享的一维数组空间`S[M]`，将两个栈的栈底分别为一维数组的两端 0 和 M-1。

![image-20220124181603654](https://s2.loli.net/2022/01/24/oxlm4nX1U9AvIba.png)

值得注意的是，判满条件应为`S->top[0] + 1 == S->top[1]`即栈顶指示器相邻，并且一个栈空并不会影响另一个栈是否为空。

```c
#define M 100

typedef struct
{
    int Stack[M]; //栈区
    int top[2];   // top[0]、top[1]为两个栈顶指示器
} DqStack;

void InitStack(DqStack *S)
{
    S->top[0] = -1;
    S->top[1] = M;
}

bool push(DqStack *S, int x, int i)
{
    if (S->top[0] + 1 == S->top[1])
        return false;
    switch (i)
    {
    case 0:
        S->top[0]++;
        S->Stack[S->top[0]] = x;
        break;
    case 1:
        S->top[1]--;
        S->Stack[S->top[1]] = x;
        break;
    default:
        return false;
    }
    return true;
}

bool pop(DqStack *S, int *x, int i)
{
    switch (i)
    {
    case 0:
        if (S->top[0] == -1)
            return false;
        *x = S->Stack[S->top[0]];
        S->top[0]--;
        break;
    case 1:
        if (S->top[1] == M)
            return false;
        *x = S->Stack[S->top[1]];
        S->top[1]++;
        break;
    default:
        return false;
    }
    return true;
}
```

###### 栈的链式实现

- 采用带头结点的单链表实现链栈
- 头指针就作为栈顶指针
- 使用完毕时应释放其空间

![image-20220124211310248](https://s2.loli.net/2022/01/24/Mp4St69WwKU1THo.png)

```c
typedef struct node
{
    int data;
    struct node *next;
} LinkStackNode;
typedef LinkStackNode *LinkStack;

bool push(LinkStack top, int x)
{
    LinkStackNode *tmp;
    tmp = (LinkStackNode *)malloc(sizeof(LinkStackNode));
    if (tmp == NULL)
        return false;
    tmp->data = x;
    tmp->next = top->next; //头插
    top->next = tmp;
    return true;
}

bool pop(LinkStack top, int *x)
{
    LinkStackNode *tmp;
    tmp = top->next;
    if (tmp == NULL) //栈空
        return false;
    top->next = tmp->next;
    *x = tmp->data;
    free(tmp);
    return true;
}
```

###### 链式实现的多栈

`top[0]`、`top[1]`、……、`top[M-1]`分别为 M 个栈的栈顶指针。

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#define M 5

typedef struct node
{
    int data;
    struct node *next;
} LinkStackNode;
typedef LinkStackNode *LinkStack;

void InitStack(LinkStack *top)
{
    *top = (LinkStack *)malloc(sizeof(LinkStackNode));
    (*top)->next = NULL;
}

bool push(LinkStack top, int x)
{
    LinkStackNode *tmp;
    tmp = (LinkStackNode *)malloc(sizeof(LinkStackNode));
    if (tmp == NULL)
        return false;
    tmp->data = x;
    tmp->next = top->next; //头插
    top->next = tmp;
    return true;
}

bool pop(LinkStack top, int *x)
{
    LinkStackNode *tmp;
    tmp = top->next;
    if (tmp == NULL) //栈空
        return false;
    top->next = tmp->next;
    *x = tmp->data;
    free(tmp);
    return true;
}

int main()
{
    LinkStack top[M];
    int tmp;
    for (int i = 0; i < 5; i++)
    {
        InitStack(&top[i]);
        push(top[i], i + 1);
        pop(top[i], &tmp);
        printf("栈%d顶元素值%d\n", i + 1, tmp);
    }
    return 0;
}

/*
输出：
栈1顶元素值1
栈2顶元素值2
栈3顶元素值3
栈4顶元素值4
栈5顶元素值5
*/
```

###### 两种存储结构的栈满

- 顺序栈判满与数组定义长度有关
- 链栈判满与可否申请系统空间有关

##### 栈的应用与递归

###### 括号匹配

```c
#include <stdio.h>
#include <stdbool.h>
#include <ctype.h>
#define Stack_Size 50

typedef struct
{
    char elem[Stack_Size];
    int top;
} Stack;

void InitStack(Stack *S) { S->top = -1; }

bool IsEmpty(Stack S) { return S.top == -1; }

bool IsFull(Stack S) { return S.top == Stack_Size - 1; }

bool push(Stack *S, char x)
{
    if (IsFull(*S))
        return false;
    S->top++;
    S->elem[S->top] = x;
    return true;
}

bool pop(Stack *S, char *x)
{
    if (IsEmpty(*S))
        return false;
    *x = S->elem[S->top];
    S->top--;
    return true;
}

bool GetTop(Stack S, char *x)
{
    if (IsEmpty(S))
        return false;
    *x = S.elem[S.top];
    return true;
}

int read_line(char str[], int n)
{
    int ch, i = 0;
    while (isspace(ch = getchar()))
        ;
    while (ch != '\n' && ch != EOF)
    {
        if (i < n)
        {
            str[i++] = ch;
            ch = getchar();
        }
    }
    str[i] = '\0';
    return i;
}

void BracketMatch(char *str)
{
    Stack S;
    int i;
    char ch;
    InitStack(&S);
    for (i = 0; str[i] != 0; i++)
    {
        switch (str[i])
        {
        case '(':
        case '[':
        case '{':
            push(&S, str[i]);
            break;
        case ')':
        case ']':
        case '}':
            if (IsEmpty(S))
            {
                printf("右括号多余\n");
                return;
            }
            else
            {
                GetTop(S, &ch);
                if ((ch == '(' && str[i] == ')') || (ch == '[' && str[i] == ']') || (ch == '{' && str[i] == '}'))
                    pop(&S, &ch);
                else
                {
                    printf("对应的左右括号不同类\n");
                    return;
                }
            }
        }
    }
    if (IsEmpty(S))
        printf("括号匹配\n");
    else
        printf("左括号多余\n");
}

int main()
{
    char str[50];
    read_line(str, 50);
    BracketMatch(str);
    return 0;
}
```

运行截图：

![image-20220124230543265](https://s2.loli.net/2022/01/24/7O5B4YUHILhfoyA.png)

###### 栈与递归

递归：在定义自身的同时又出现了对自身的调用

递归定义的数学函数：斐波那契数列

![image-20220126115711930](https://s2.loli.net/2022/01/26/AgVly9BNMsJZdfu.png)

阿克曼函数

![image-20220126115654249](https://s2.loli.net/2022/01/26/kjBAGRDvMTWfoI8.png)

例如递归求解汉诺塔：

```c
//将A塔座上1到n编号的由小到大圆盘按规则搬到C塔座上，B柱作为辅助塔座
void Hanoi(int n, char A, char B, char C) 
{
    if (n == 1)
        move(A, C);
    else if (n > 1)
    {
        Hanoi(n - 1, A, C, B);
        move(A, C);
        Hanoi(n - 1, B, A, C);
    }
}
```

使用递归的前提：

- 原问题可以层层分解为类似的子问题，且子问题比原问题的规模更小
- 规模最小的子问题具有直接解

递归过程的实现：

- 递归进程（i->i+1 层）
  - 保留本层参数与返回地址
  - 给下层参数赋值
  - 将程序转移到被调函数的人口
- 递归退程（i+1->i 层）
  - 保存被调函数的计算结果
  - 恢复上层参数〈释放被调函数的数据区)
  - 依照被调函数保存的返回地址，将控制转移回调用函数

而上述过程实质上是利用栈机制实现的。

但递归算法有缺陷：

- 时间效率低
- 可能栈溢出。由于递归是用栈实现的，而每个进程的栈容量有限，当调用层次过多，可能发生栈溢出
- 部分语言不支持递归（如 basic 语言）

基此原因，我们需要消除递归：

- 第一类，简单递归问题的转换。对于尾递归、单向递归的算法，使用循环代替
- 第二类，基于栈的方式。将隐性栈转化为受用户控制的显性栈

{{< admonition info >}}

尾递归：递归调用语句只有一个且处于算法的最后，尾递归是单向递归的特例。

{{< /admonition >}}

斐波那契数列的非递归实现：

```c
int fib(int n)
{
    int x = 0, y = 1, z;
    if (n == 0 || n == 1)
        return n;
    else
        for (int i = 2; i <= n; i++)
        {
            z = y;
            y = x + y;
            x = z;
        }
    return y;
}
```

循环结构表示阶乘问题的尾递归算法：

```c
long fact(int n)
{
    int fac = 1;
    for (int i = 1; i <= n; i++)
        fac *= i;
    return fac;
}
```

#### 队列

##### 队列的定义与实现

- 队列（Queue）定义：只允许在表的一端插人元素，而在另一端删除元素的一种限定性线性表。
- 队头：允许删除的一端
- 队尾：允许插入的一端
- 特性：先进先出（ Fist In Fist Out, FIFO）

对于队列 ADT：

数据元素：同一个数据对象的任意类型数据

关系：队列中数据元素之间是线性关系

有 8 种基本操作运算：

1. 初始化`InitQueue(&Q)`
2. 判空`IsEmpty(Q)`
3. 判满`IsFull(Q)`
4. 进队`EnterQueue(&Q, x)`
5. 出队`DeleteQueue(&Q, &x)`
6. 取队头`GetHead(Q, &x)`
7. 清队`ClearQueue(&Q)`
8. 删除队列`DestroyQueue(&Q)`

###### 队列的链式实现

![image-20220125184405163](https://s2.loli.net/2022/01/25/b8v4G79tQ5YZMkT.png)

- 队首指针`front`、队尾指针`rear`为队首、队尾指示器
- 队尾进，队首出

```c
typedef struct Node
{
    int data;
    struct Node *next;
} LinkQueueNode;

typedef struct
{
    LinkQueueNode *front;
    LinkQueueNode *rear;
} LinkQueue;

bool InitQueue(LinkQueue *Q)
{
    Q->front = (LinkQueueNode *)malloc(sizeof(LinkQueueNode));
    if (Q->front != NULL)
    {
        Q->rear = Q->front;
        Q->front->next = NULL;
        return true;
    }
    else
        return false;
}

bool EnterQueue(LinkQueue *Q, int x)
{
    LinkQueueNode *NewNode;
    NewNode = (LinkQueueNode *)malloc(sizeof(LinkQueueNode));
    if (NewNode != NULL)
    {
        NewNode->data = x;
        NewNode->next = NULL;
        Q->rear->next = NewNode;
        Q->rear = NewNode;
        return true;
    }
    return false;
}

bool DeleteQueue(LinkQueue *Q, int *x)
{
    LinkQueueNode *p;
    if (Q->front == Q->rear) //空队
        return false;
    p = Q->front->next;
    Q->front->next = p->next;
    if (Q->rear == p)
        Q->rear = Q->front;
    *x = p->data;
    free(p);
    return true;
}
```

###### 队列的顺序存储（循环队列）

- 用一维数组`Queue[MAXSIZE]`存放从队头到队尾的元素
- 附设两个指针`front`和`rear`，分别指示队头元素和队尾元素在数组中的位置
- 由于只能在队头出队,在队尾入队，所以会产生**假溢出**的现象

{{< admonition info >}}

假溢出是指已经队满，但实际在队列的另一端还是有存储空间的情况。

例如，当你想乘坐公交时，从上车车门处看见人满了，但从车窗看，发现里面还有空位，但车上的人不往里走，这就产生了所谓的假溢出现象

{{< /admonition >}}

为解决假溢出，将顺序队列的数组`Queue[MAXSIZE]`看成一个环状的空间，即规定最后一个单元的后继为第一个单元，我们形象地称之为**循环队列**。

可通过数学中的取模（求余）运算来实现循环队列`rear = (rear+1) mod MAXSIZE`。

当`rear+1 = MAXSIZE`时，`rear = O`。也就是说最后一个`Queue[MAXSIZE-1]`的后继为`Queue[0]`

- 进队操作时，队尾指针的变化是`rear = (rear+1) mod MAXSIZE`
- 出队操作时，队头指针的变化是`front = (front+1) mod MAXSIZE`

在顺序队列中我们依靠`front == rear`来判断队列空，但不难看出，仅凭这一点难以判断循环队列的空满。在此，有两种解决办法：

1. 损失一个元素空间，当队尾指针所指向的空单元的后继单元是队头元素所在的单元时，则停止入队。
   - 队列满的条件为`(rear+1) mod MAXSIZE == front`
   - 队列空的条件为`rear == front`
2. 增设一个标志量，以区别队列是空，还是满

![image-20220126121643083](https://s2.loli.net/2022/01/26/QEKiH6zdJoOebFx.png)

```c
#define MAXSIZE 100

typedef struct
{
    int front;
    int rear;
    int elem[MAXSIZE];
} SeqQueue;

void InitQueue(SeqQueue *Q)
{
    Q->front = Q->rear = 0;
}

bool EnterQueue(SeqQueue *Q,int x)
{
    if ((Q->rear + 1) % MAXSIZE == Q->front)
        return false;
    Q->elem[Q->rear] = x;
    Q->rear = (Q->rear + 1) % MAXSIZE;
    return true;
}

bool DeleteQueue(SeqQueue *Q,int *x)
{
    if(Q->front==Q->rear)
        return false;
    *x = Q->elem[Q->front];
    Q->front = (Q->front + 1) % MAXSIZE;
    return true;
}
```

{{< admonition question >}}

大小为 MAXSIZE 的循环队列中，f 为当前队头元素位置，为队尾元素的后一个位置，则任意时刻，队列中的元素个数为？

{{< /admonition >}}

{{< admonition success 答案 >}}

要考虑到 r 和 f 谁更大的问题，因此要分类讨论。综合可得元素个数为 (r - f + MAXSIZE) % MAXSIZE

{{< /admonition >}}

##### 队列的应用

###### 杨辉三角

```c
void YangHuiTriangle(int N)
{
    SeqQueue Q;
    int i, x, n, tmp;
    InitQueue(&Q);
    EnterQueue(&Q, 1);
    for (n = 2; n <= N; n++)
    {
        EnterQueue(&Q, 1);
        for (i = 1; i <= n - 2; i++)
        {
            DeleteQueue(&Q, &tmp);
            printf(" %d", tmp);
            GetHead(Q, &x);
            tmp += x;
            EnterQueue(&Q, tmp);
        }
        DeleteQueue(&Q, &x);
        printf(" %d", x);
        printf("\n");
        EnterQueue(&Q, 1);
    }
    while (Q.front != Q.rear)
    {
        DeleteQueue(&Q, &x);
        printf(" %d", x);
    }
}
```

#### 总结

