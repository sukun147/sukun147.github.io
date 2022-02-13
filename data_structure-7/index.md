# 数据结构学习(七)


<!--more-->

[本系列学习笔记链接](/categories/数据结构/)

# 数据结构——用C语言描述(四)

## 非线性结构

### 图

#### 图的定义与基本术语

##### 图的定义

图（Graph）是一种网状数据结构，其形式化定义如下：
$$
Graph=(V,R)\\\
V=\{x|\ x∈DataObject\}\\\
R=\{VR\}\\\
VR=\{<x,y>|\ P(x,y)∧(x,y∈V)\}
$$
其中，

- DataObject：具有相同特性的元素的集合
- V：顶点（vertex）集合
- VR：顶点间关系的集合
- P(x,y)：表示 x 和 y 之间有特定的关联属性 P

##### 图的抽象类型定义

数据对象 V：具有相同特性的元素的集合

结构关系：多对多
$$
R=\{VR\}\\\
VR=\{<x,y>|\ P(x,y)\ (xy∈V)\}
$$
操作集合：

1. 创建`GreateGraph(G)`
2. 销毁`DestoryGraph(G)`
3. 确定顶点位置`LocateVertex (G, v)`
4. 取第 i 个顶点`GetVertex(G, i)`
5. 找顶点 v 的第一个邻接点`FirstAdjVertex(G, v)`
6. 找顶点 v 的下一个邻接点`NextAdjVertex(G, v, w)`
7. 插入顶点`InsertVertex(G, u)`
8. 删除顶点及弧`DeleteVertex(G, v)`
9. 插入弧`lnsertArc (G, v, w)`
10. 删除弧`DeleteArc (G, v, w)`
11. 遍历`TraverseGraph(G)`

##### 基本术语

- 图

  - 无向图：图中边无方向
  - 有向图：图中边有方向
  - 有向完全图：图中每个顶点和其余 n-1 个顶点都有弧相连（总共$n×(n-1)$条边）
  - 无向完全图：图中每个顶点和其余 n-1 个顶点都有边相连（总共$\frac{n×(n-1)}{2}$条边）
  - 子图：若 G' 的顶点包含于 G 的顶点，则称图 G' 为 G 的子图
  - 连通图：对于图中任意两个顶点都是连通的
  - 连通分量：无向图中的极大连通子图
  - 强连通图：任意两个顶点之间互相可达
  - 强连通分量：有向图的极大强连通子图

- 邻接点：两顶点之间存在边，则称其互为邻接点

- 路径和回路

  - 路径：顶点序列
  - 路径长度：顶点序列上经过的边的个数
  - 回路或环：起点和终点相同
  - 简单路径：顶点序列中的顶点各不相同的路径
  - 简单回路：除了第一个和最后一个顶点外，其余各顶点均不重复出现的回路

- 度

  - 无向图的度：顶点 V 的度 TD(V)——和 V 相连的边的个数

  - 有向图的度：入度+出度

    入度 ID(V)：进来的弧

    出度 OD(V)：出去的弧

  - 度的计算公式：n 个顶点，e 条边或弧，则有
    $$
    2e=\sum_{i=1}TD(V_i)
    $$

- 权和网

  - 权：图的边或弧上与它相关的数（可以表示从一个顶点到另一个顶点的距离或耗费等信息）
  - 赋权图（网）：带权的图

- 生成树：极小连通子图，含有连通图全部顶点 n 并有 n-1 条边

- 顶点在图中的位置：人为排列中的位置序号，可将任一顶点看作图的第一个顶点，对任一顶点其邻接点间不存在顺序关系

#### 图的存储结构

显然，我们需要存储顶点和顶点间关系两部分信息，对此我们有如下四种办法

##### 邻接矩阵表示法（数组表示法）

###### 定义

用两个数组来表示图：存储顶点信息的一维数组、存储顶点关联关系的二维数组（邻接矩阵）

###### 表示

G 是一具有 n 个顶点的**无权图**，G 的邻接矩阵是如下性质的 n×n 矩阵 A：
$$
A[i,j]=
\begin{cases}
1 &若<v_i,v_j>或(v_i,v_j)∈VR\\\\
0 &反之
\end{cases}
$$
若图 G 是一个有 n 个顶点的**网**，则它的邻接矩阵是具有如下性质的 n×n 矩阵 A：
$$
A[i,j]=
\begin{cases}
W_{ij} &若<v_i,v_j>或(v_i,v_j)∈VR\\\\
0或∞ &反之
\end{cases}
$$
例如：

![image-20220211212142562](https://s2.loli.net/2022/02/11/7cTfGbN14ZIJzwl.png)
$$
A=\left(
  \begin{array}{cccc}
    0 & 1 & 1 & 1\\\
    1 & 0 & 0 & 0\\\
    1 & 0 & 0 & 1\\\
    1 & 0 & 1 & 0\\\
  \end{array}
\right)
$$

###### C语言类型描述

```c
#define MAX_VERTEX_NUM 20
#define INFINITY 32678
#define OtherInfo int
typedef enum
{
    DG,  //有向
    DN,  //有向网
    UDG, //无向
    UDN  //无向网
} GraphKind;
typedef char VertexData;
typedef struct ArcNode //弧的结点类型
{
    AdjType adj;    //关联信息
    OtherInfo info; //边权信息
} ArcNode;
typedef struct
{
    VertexData vexs[MAX_VERTEX_NUM];               //顶点数组
    ArcNode arcs[MAX_VERTEX_NUM][MAX_VERTEX_NUM]; //关联关系数组
    int vexnum, arcnum;                           //顶点信息，弧的信息
    GraphKind kind;                               //弧的种类
} AdjMatrix;          
```

###### 特点

1. 存储空间：

- 无向图

  邻接矩阵是对称矩阵，可采用下三角压缩存储只需$\frac{n×(n-1)}{2}$空间

- 有向图

  邻接矩阵不一定是对称矩阵，所以需要$n^2$个存储空间。

2. 便于运算：

- 根据$A[i,j]=0或1$来判定图中任意两个顶点之间是否有边相连

- 便于求各个顶点的度

  - 无向图：其邻接矩阵第 i 行元素之和就是图中第i 个顶点的度
    $$
    TD(v_i)=\sum_{j=1}^nA[i,j]
    $$

  - 有向图：第 i **行**元素之和就是图中第 i 个顶点的**出度**；第 i **列**元素之和就是图中第 i 个顶点的**入度**。

  $$
  OD(v_i)=\sum_{j=1}^nA[i,j]\\\
  ID(v_i)=\sum_{j=1}^nA[j,i]
  $$

3. 便于实现一些基本操作

###### 创建有向网的算法

```c
bool CreateDN(AdjMatrix *G)
{
    int i, j, k, weight;
    VertexData v1, v2;
    scanf("%d,%d", &G->arcnum, &G->vexnum);
    for (i = 0; i < G->vexnum; i++)
        for (j = 0; j < G->vexnum; j++)
            G->arcs[i][j].adj = INFINITY; //初始化
    for (i = 0; i < G->vexnum; i++)
        scanf("%c", &G->vexs[i]); //读取顶点的一维数组
    for (k = 0; k < G->arcnum; k++)
    {
        scanf("%c,%c,%d", &v1, &v2, &weight);
        i = LocateVex_M(G, v1);
        j = LocateVex_M(G, v2);
        G->arcs[i][j].adj = weight; //生成顶点
    }
    return true;
}
```

##### 邻接表表示法

###### 基本思想

采用链式结构存储图，只存储图中有关联的边的信息：

- 对图中 n 个顶点均建有关联的边链表
- 每个顶点信息与其边链表的头指针构成表头结点表

###### 结构构成

- 表头结点表：

  由所有表头结点以顺序结构的形式存储，以便可以随机访问任一顶点的邻接点单链表。
  $$
  \begin{array}{|c|c|}
          \hline
          vexdata(数据域) & firstarc(链域)\\\\\hline
  \end{array}
  $$

- 边表：

  由表示图中顶点间邻接关系的 n 个边链表组成。
  $$
  \begin{array}{|c|c|c|}
          \hline
          adjvex(邻接点域) & info(数据域) & nextarc(链域)\\\\\hline
  \end{array}
  $$

###### 图例



![image-20220211212142562](https://s2.loli.net/2022/02/11/7cTfGbN14ZIJzwl.png)

![image-20220212163629214](https://s2.loli.net/2022/02/12/nCYdf147Uq3wJ9M.png)

###### 结构类型定义

```c
#define MAX_VERTEX_NUM 10
#define OtherInfo int
typedef enum
{
    DG,
    DN,
    UDG,
    UDN
} GraphKind;
typedef int VertexData;
typedef struct ArcNode
{
    int adjvex;
    struct ArcNode *nextarc;
    OtherInfo info;
} ArcNode;
typedef struct VertexNode
{
    VertexData data;
    ArcNode *firstarc;
} VertexNode;
typedef struct
{
    VertexNode vertex[MAX_VERTEX_NUM];
    int vexnum, arcnum;
    GraphKind kind;
} AdjList;
```

###### 存储空间

n 个顶点，e 条边的无向图：

采取邻接表作为存储结构，需要 n 个表头结点和 2e 个表结点。

###### 无向图的度

在无向图的邻接表中，顶点$V_i$的度恰好就是第 i 个边链表上结点的个数

###### 有向图的度

有向图中，第 i 个边链表上顶点的个数是顶点$V_i$的出度。

求该顶点的入度，须遍历整个邻接表。在所有单链表中，查找邻接点域的值为 i 的结点并计数求和。

{{< admonition success 解决方案>}}

逆邻接表法：

对每一顶点$V_i$再建立一个所有以顶点$V_i$为弧头的弧的表（逆邻接表)。求顶点$V_i$的入度即是计算逆邻接表中第 i 个顶点的边链表中节点个数

正向邻接表求出度，逆向邻接表求入度。

{{< /admonition >}}

##### 十字链表

###### 定义

十字链表是有向图的另一种链式存储结构。它是有向图的邻接表和逆邻接表结合的一种链表。

有向图中的每一条弧对应十字链表中的一个弧结点，有向图中的每个顶点对应有一个结点，叫做顶点结点。

###### 结构

用于存储顶点的首元结点结构：
$$
\begin{array}{|c|c|c|}
        \hline
        data & firstin & firstout\\\\\hline
\end{array}
$$

- `data`数据域
- `firstin`以当前顶点为弧头的其他顶点构成的链表
- `fitstout`以当前顶点为弧尾的其他顶点构成的链表

普通结点的结构：
$$
\begin{array}{|c|c|c|c|c|}
        \hline
        tailvex & headvex & hlink & tlink & info\\\\\hline
\end{array}
$$

- `tailvex`弧尾顶点的位置下标
- `headvex`弧头顶点的位置下标
- `hlink`下一个以首元节点为弧头的顶点
- `tlink`下一个以首元节点为弧尾的顶点
- `info`数据域

###### 结构类型定义

```c
#define OtherInfo int
#define MAX_VERTEX_NUM 20
typedef VertexData int;
typedef struct ArcBox
{
    int tailvex, headvex;         //弧尾、弧头对应顶点在数组中的位置下标
    struct ArcBox *hlink, *tlink; //指向弧头相同和弧尾相同的下一个弧
    OtherInfo info;
} ArcBox;
typedef struct VertexNode
{
    VertexData data;
    ArcBox *firstin, *firstout; //分别指向以该顶点为弧头和弧尾的链表首个结点
} VertexNode;
typedef struct
{
    VertexNode xlist[MAX_VERTEX_NUM];
    int vexnum, arcnum;
} OLGraph;
```

###### 图例

![image-20220212181409288](https://s2.loli.net/2022/02/12/GVw9OYtJZLNTshH.png)

###### 建立算法

```c
void CreateDG(OLGraph *G)
{
    int v1, v2;
    scanf("%d,%d", &G->vexnum, &G->arcnum);
    for (int i = 0; i < G->vexnum; i++) //初始化
    {
        scanf("%d", &G->xlist[i].data);
        G->xlist[i].firstin = NULL;
        G->xlist[i].firstout = NULL;
    }
    for (int k = 0; k < G->arcnum; k++)
    {
        scanf("%d,%d", &v1, &v2);
        int i = LocateVex(G, v1);
        int j = LocateVex(G, v2);
        ArcBox *p = (ArcBox *)malloc(sizeof(ArcBox)); //建立弧的结点
        p->tailvex = i;
        p->headvex = j;
        //采用头插法插入新的p结点
        p->hlink = G->xlist[j].firstin;
        p->tlink = G->xlist[i].firstout;
        G->xlist[j].firstin = G->xlist[i].firstout = p;
    }
}
```

###### 优点

在十字链表中能容易地找到以$V_i$为尾的弧，也能容易地找到以$V_i$为头的弧，有向图若采用十字链表作为存储结构，容易求出顶点$V_i$的度。

##### 邻接多重表

###### 定义

邻接多重表是无向图的另外一种存储结构，它将图中关于一条边的信息用一个结点来表示，能够提供更为方便的边处理信息。

###### 结构

顶点结构：
$$
\begin{array}{|c|c|}
        \hline
        data & firstedge\\\\\hline
\end{array}
$$
边结点结构：
$$
\begin{array}{|c|c|c|c|c|c|}
        \hline
        mark(标志域) & ivex & ilink & jvex & jlink & info\\\\\hline
\end{array}
$$

###### 结构类型定义

```c

```

###### 图例

![](https://s2.loli.net/2022/02/12/7lT2NBfQwkeipHZ.png)

#### 图的遍历



#### 图的应用

