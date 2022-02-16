# 数据结构学习(五)


<!--more-->

[本系列学习笔记链接](/categories/数据结构/)

# 数据结构——用C语言描述(五)

## 线性结构

### 数组

#### 数组的定义与运算

数组：是一组有固定个数的数据元素集合，是一般线性表的推广，组成线性表的元素为带有结构信息的元素。

—维数组为向量形式的线性表;

二维数组是由一维数组组成的线性表，依次类推可得到多维数组定义。

对于数组 ADT

数据对象：高维数组组成的线性表
$$
D=\{a_{j1j2…jn}|\ n>0,1≤j_i≤b_i,a_{j1j2…jn}∈ElementSet\}
$$
其中，n 称为数组的维数，j 是数组的第 i 维下标，b 是数组第 i 维的长度。

结构关系：线性序列
$$
R=\{R_1,R_2,…,R_n\}
$$
有 4 种基本操作：

1. 初始化`InitArray(A, n, bound1,…, boundn)`
2. 销毁`DestroyArray(A)`
3. 取值`GetValue(A, e, index1,…, indexn)`
4. 修改`SetValue(A, e, index1,…, indexn)`

#### 数组的顺序存储与实现

数组适合顺序存储：

1. 给定数组维数及各维长度，数组元素个数固定
2. 元素存取运算不改变数组大小

同时，计算机内存储器是一维的，可直接顺序存储一维数组，但高维数组则是按照某种次序把高维映射到一维。

1. 按行序存储，如 BASIC、C 语言
2. 按列序存储，如 FORTRAN 语言（世界上第一个被正式推广使用的高级语言）

对于数组中，给定下标计算其在一维空间的存储位置有以下公式：
$$
数组元素地址=基址+(该变量线性序号-1)*size
$$
{{< admonition success 举个栗子>}}

对于一维数组`A=(a1,a2,…,an)`，每个元素占据`size`个存储单元。

则元素 ai 的存储地址为
$$
Loc(a[i])=Loc(a[1])+(i-1)×size
$$
{{< /admonition >}}

{{< admonition question >}}

二维数组`a[5][4]`按行序存放，每个元素占 4 单元,首元素地址是 2000 ，求`a[3][2]`的内存地址。

{{< /admonition >}}

{{< admonition success 答案 >}}

类似与举个栗子中一维数组的公式，二维数组（下标从 1 开始）中有
$$
Loc(a[i][j])=Loc(a[1][1])+(n×(i-1)+(j-1))×size
$$
故本题答案为
$$
Loc(a[3][2]) = 2000+(4×(3-1) +(2-1))×4=2036
$$
{{< /admonition >}}

我们还可以对此推广至一般三维数组，其下限为c1、c2、c3，上限为d1、d2、d3，则
$$
Loc(A[j_1][j_2][j_3])=Loc(A[c_1][c_2][c_3])+((d_2-c_2+1)×(d_3-c_3+1)×(j_1-c_1)+(d3-c3+1)×(j_2-c_2)+(j_3-c_3))×size
$$
{{< admonition info >}}

上述数组可记作`A[c1..d1,c2..d2,c3..d3]`

{{< /admonition >}}

再做进一步的推广至 n 维，有
$$
Loc[j_1,j_2,…,j_n]=Loc[c_1,c_2,…,c_n]+\sum_{i=1}^Na_i×(j_i-c_i)
$$
其中
$$
a_i=size×\prod_{k=i+1}^n(d_k-c_k+1)\ (1≤i≤n)
$$
{{< admonition >}}

地址计算的题目容易出现在各种考试中，但这些公式不应该去死记硬背，而是重在理解！

{{< /admonition >}}

#### 规律分布特殊矩阵的压缩存储

原则：对有规律的元素和值相同的元素只分配一个存储单元，对于零元素不分配空间。

思想：按规律找出地址计算公式

##### 三角矩阵

$$
A=
\begin{pmatrix}
    a_{11} &  & \\\\
    a_{21} & a_{22} & \\\\
    a_{31} & a_{32} & a_{33}\\\\
\end{pmatrix}
\rightarrow
\begin{array}{|c|}
        \hline
        a_{11}\\\\\hline
        a_{21}\\\\\hline
        a_{22}\\\\\hline
        ···   \\\\\hline
        a_{nn}\\\\\hline
\end{array}
$$

显然，此一维存储空间的大小以及地址计算公式为：
$$
\sum_{i=1}^ni=\frac{n(n+1)}{2}\\\
Loc[i,j]=Loc[1,1]+(\frac{i(i-1)}{2}+j-1)*size\ (i>j)
$$

##### 带状矩阵

$$
A_{n×n}=
\begin{pmatrix}
    a_{11} & a_{12} &\\\\
    a_{21} & a_{22} & a_{23}\\\\
     & a_{32} & a_{33} & a_{34}\\\\
     &  & a_{43} & a_{44} & a_{45}\\\\
     &  &  &  … & … & …
\end{pmatrix}
\rightarrow
\begin{array}{|c|}
        \hline
        a_{11}\\\\\hline
        a_{12}\\\\\hline
        a_{21}\\\\\hline
        a_{22}\\\\\hline
        a_{23}\\\\\hline
        ···   \\\\\hline
        a_{nn}\\\\\hline
\end{array}
$$

显然，此一维存储空间的大小为
$$
3n-2
$$
又注意到元素下标 i、j 有以下关系：
$$
j-i=
\begin{cases}
-1 &,j<i对角线下 \\\\
0 &,j=i对角线 \\\\
1 &,j>i对角线上
\end{cases}
$$
故而
$$
前i-1行元素个数=3(i-1)-1\\\
当前行元素序号=j-i+1
$$
由此，地址计算公式为
$$
Loc(A[i][j])=Loc(A[1][1])+(3(i-1)-1+j-i+1)*size\\\
=Loc(A[1][1])+(2(i-1)+j-1)*size
$$
{{< admonition success 举个栗子>}}

基地址为 2000，size 为1，求元素 a23 的地址？

WP：前 1 行有 3(2-1)-1=2 个元素，第 2 行元素序号为 3-2+1=2，故
$$
Loc(A[2][3])=2000+(2+2)×1=2004
$$
{{< /admonition >}}

#### 稀疏矩阵的压缩存储

稀疏矩阵：指矩阵中大多数元素为零的矩阵。一般地，当非零元素个数只占矩阵元素总数的25%~30%，或低于这个百分数时，我们称这样的矩阵为稀疏矩阵。

##### 三元组表表示法

由于稀疏矩阵中非零元的毫无规律，我们需要在存储值的同时也存储其行号和列号，这样的方法称为三元组表表示法。

```c
#define MAXSIZE 1000

typedef struct
{
    int row, col;
    int e;
} Triple;

typedef struct
{
    Triple data[MAXSIZE + 1];
    int m, n, len; // m行n列，非零元有len个
} TSmatrix;
```

###### 经典转置法

```c
void TransMatrix(int source[n][m], int dest[m][n])
{
    int i, j;
    for (i = 0; i < m; i++)
        for (j = 0; j < n; j++)
            dest[i][j] = source[j][i];
}
```

###### 列序递增转置法

$$
\begin{pmatrix}
    1 &  & 2\\\\
     & 3 & \\\\
    4 &  & 5\\\\
     & 6 & \\\\
\end{pmatrix}
$$

此稀疏矩阵的三元组表为

|      | row  | col  |  e   |
| ---- | :--- | ---- | :--: |
| 1    | 1    | 1    |  1   |
| 2    | 1    | 3    |  2   |
| 3    | 2    | 2    |  3   |
| 4    | 3    | 1    |  4   |
| 5    | 3    | 3    |  5   |
| 6    | 4    | 2    |  6   |

易于发现，仅做行列互换（交换`row`和`col`）并没有完成转置操作，因为行列互换后没有做到以行序为主序，此时还需要对行下标做排序。

对此，我们有两种方法，其一是依次扫描`col`为 1 到 n，即列序递增转置法。

```c
void TransposeTSMatrix(TSmatrix A, TSmatrix *B)
{
    int i, j, k;
    B->m = A.n;
    B->n = A.m;
    B->len = A.len;
    if (B->len)
    {
        j = 1;
        for (k = 1; k <= A.n; k++)
            for (i = 1; i <= A.len; i++)
                if (A.data[i].col == k)
                {
                    B->data[j].row = A.data[i].col;
                    B->data[j].col = A.data[i].row;
                    B->data[j].e = A.data[i].e;
                    j++;
                }
    }
}
```

易得出，该算法时间复杂度为
$$
O(A.n×A.len)
$$
此算法大大降低了存储空间的开销，但时间耗费并未降低，原因在于要多次扫描稀疏矩阵三元组。

###### 一次定位法

为了提高算法性能，我们想通过一次循环完成转置，即对 A 中非零元“一次定位”直接放到 B 三元表中。

需设置两个数组存放预先计算的值，实现一次定位：

- `num[col]`存放 A 三元组 col 列非零元个数
- `position[col]`存放 A 三元组 col 列第一个非零元位置

由此，我们可得出一条递归关系`position[col] = position[col-1] + num[col-1]`其中 2<=col<=A.n。

```c
void FastTransposeTSMatrix(TSmatrix A, TSmatrix *B)
{
    int col, t, p, q, num[MAXSIZE], position[MAXSIZE];
    B->len = A.len;
    B->m = A.n;
    B->n = A.m;
    if (B->len)
    {
        for (col = 1; col <= A.n; col++)
            num[col] = 0;
        for (t = 1; t <= A.len; t++)
            num[A.data[t].col]++;
        position[1] = 1;
        for (col = 2; col <= A.n; col++)
            position[col] = position[col - 1] + num[col - 1];
        for (p = 1; p < A.len; p++)
        {
            col = A.data[p].col;
            q = position[col];
            B->data[q].row = A.data[p].col;
            B->data[q].col = A.data[p].row;
            B->data[q].e = A.data[p].e;
            position[col]++;
        }
    }
}
```

由于此算法只扫描一次三元组，故时间复杂度为
$$
O(A.n+A.len)
$$
{{< admonition success 举个栗子>}}

现有一稀疏矩阵A，A.m=100，A.n=500，A.len=100

- 使用列序递增法：时间耗费为 A.n×A.len=50000 次，存储耗费为 A.len×3=300
- 使用一次定位法：时间耗费为 A.n+A.len+A.n+A.len=1200 次，存储耗费为 A.len×3+A.n×2=1300

可以看出，一次定位法大大降低了时间开销。

{{< /admonition >}}

##### 链式存储结构：十字链表

使用链表存储稀疏矩阵可以实现矩阵的动态存储，即灵活地进行矩阵运算操作。

结点结构示意图：

![image-20220202174235148](https://s2.loli.net/2022/02/02/WA19hsxNynMGPu8.png)

```c
typedef struct
{
    int row, col; //非零元的行列下标
    int value;
    struct OLNode *right, *down; //非零元的后继链域
} OLNode, *OLlik;

typedef struct
{
    OLlik *row_head, *col_head; //行列链表头指针
    int m, n, len;              // m行n列，有len个非零元的稀疏矩阵
} CrossList;
```

$$
\begin{pmatrix}
    -3 & 0 & 0 & 5\\\\
    0 & -1 & 0 & 0\\\\
    8 & 0 & 0 & 7\\\\
\end{pmatrix}
$$

上述矩阵十字链表描的示意图：

![image-20220202180335191](https://s2.loli.net/2022/02/02/XCJ5RhV9QMAbmIp.png)

容易看出，该矩阵用十字链表表示需要 4+3+5=12 个结点。

### 广义表

广义表是线性表的推广，是 n 个数据元素的有限序列。记作：
$$
GL=(d_1,d_2,…,d_n)
$$
称 n 为广义表长度，GL 为广义表名。

若广义表的元素也是广义表，则称其为子表，也就是说广义表是递归定义的。

规定：广义表元素大写，单元素小写，d1 为表头，其余元素构成的表 (d2, d3, …, dn) 是表尾。

#### 广义表的存储结构

##### 广义表的头尾链表存储结构

由于广义表的特性，我们需要有两类结点：单元素结点、表结点。

对于表结点，需要有三个域：标志域、指向表头的指针域、指向表尾的指针域。

而对于但元素结点，只需要两个域：标志域、值域。

```c
typedef enum
{
    ATOM,
    LIST
} ElemTag; // ATOM表示原子默认值0，LIST表示子表默认值1

typedef struct GLNode
{
    ElemTag tag; //标志域
    union
    {
        int atom; //原子结点值域
        struct
        {
            struct GLNode *hp, tp;
        } htp;  //表结点
    } atom_htp; //联合体域
} * GList;
```

##### 广义表的同层结点链存储结构

单元素结点和子表结点均由三个域组成。

```c
typedef enum
{
    ATOM,
    LIST
} ElemTag;

typedef struct GLNode
{
    ElemTag tag; //标志域
    union
    {
        int atom; //原子结点值域
        struct GLNode *hp;
    } atom_hp; //联合体域
    struct GLNode *tp;
} * GList;
```


