# 数据结构学习(七)


<!--more-->

[本系列学习笔记链接](/categories/数据结构/)

# 数据结构——用C语言描述(七)

## 非线性结构

### 图

#### 图的定义与基本术语

##### 图的定义

图（Graph）是一种网状数据结构，其形式化定义如下：

Graph=(V,R)

V={x|x∈DataObject}

R={VR}

VR={<x,y>|P(x,y)∧(x,y∈V)}

其中，

- DataObject：具有相同特性的元素的集合
- V：顶点（vertex）集合
- VR：顶点间关系的集合
- P(x,y)：表示 x 和 y 之间有特定的关联属性 P

##### 图的抽象类型定义

数据对象 V：具有相同特性的元素的集合

结构关系：多对多

R={VR}

VR={<x,y>|P(x,y) (xy∈V)}

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
#define VRType int
#define InfoType char
#define VertexType int
#define MAX_VERTEX_NUM 20
#define INFINITY 32678
typedef enum
{
    DG,  //有向
    DN,  //有向网
    UDG, //无向
    UDN  //无向网
} GraphKind;
typedef struct
{
    VRType adj;     //对于无权图，用 1 或 0 表示是否相邻；对于带权图，直接为权值
    InfoType *info; //弧或边额外含有的信息指针
} ArcNode;
typedef struct
{
    VertexType vexs[MAX_VERTEX_NUM];              //顶点
    ArcNode arcs[MAX_VERTEX_NUM][MAX_VERTEX_NUM]; //顶点之间的关系
    int vexnum, arcnum;                           //顶点数，弧数
    GraphKind kind;                               //图的种类
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
    VertexType v1, v2;
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
#define VertexType int
#define InfoType int
#define MAX_VERTEX_NUM 20
typedef enum
{
    DG,
    DN,
    UDG,
    UDN
} GraphKind;
typedef struct ArcNode
{
    int adjvex;              //邻接点在数组中的位置下标
    struct ArcNode *nextarc; //指向下一个邻接点的指针
    InfoType *info;          //信息域
} ArcNode;
typedef struct VNode
{
    VertexType data;   //顶点的数据域
    ArcNode *firstarc; //指向邻接点的指针
} VNode;
typedef struct
{
    VNode vertex[MAX_VERTEX_NUM]; //存储各链表头结点的数组
    int vexnum, arcnum;           //顶点数和边或弧数
    GraphKind kind;               //图的种类
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
#define InfoType int
#define VertexType int
#define MAX_VERTEX_NUM 20
typedef struct ArcBox
{
    int tailvex, headvex;         //弧尾、弧头对应顶点在数组中的位置下标
    struct ArcBox *hlink, *tlink; //分别指向弧头相同和弧尾相同的下一个弧
    InfoType *info;
} ArcBox;
typedef struct VertexNode
{
    VertexType data;
    ArcBox *firstin, *firstout; //指向以该顶点为弧头和弧尾的链表首个结点
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
        mark & ivex & ilink & jvex & jlink & info\\\\\hline
\end{array}
$$

###### 结构类型定义

```c
#define InfoType int
#define VertexType int
#define MAX_VERTEX_NUM 20
typedef enum
{
    unvisited,
    visited
} VisitIf;
typedef struct EBox
{
    VisitIf mark;               //标志域
    int ivex, jvex;             //边的两顶点在数组中的位置下标
    struct EBox *ilink, *jlink; //分别指向与ivex、jvex相关的下一个边
    InfoType *info;             //边包含的其它的信息
} EBox;
typedef struct VexBox
{
    VertexType data;
    EBox *firstedge; //顶点相关的第一条边
} VexBox;
typedef struct
{
    VexBox adjmulist[MAX_VERTEX_NUM]; //顶点数组
    int vexnum, edgenum;              //顶点数和边数
} AMLGraph;
```

###### 图例

![邻接多重表](https://img-blog.csdnimg.cn/20201108104721155.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N1bnNoaW9uX2JveQ==,size_16,color_FFFFFF,t_70#pic_center)

#### 图的遍历

##### 定义

从图中的某个顶点出发，按某种方法对图中的所有顶点访问且仅访问一次。

显然我们需要设置访问标志数组`visited[]`：

- 保证图中的各顶点均被访问
- 避免重复访问

##### 方法

```c
int visited[];
void TraverseGraph(Graph g) //Graph表示图的一种存储结构，如邻接表
{
    int vi;
    for (vi = 0; vi < g.vexnum; vi++)
        visited[vi] = 0;
    for (vi = 0; vi < g.vexnum; vi++)
        if (!visited[vi])
            search(g, vi);
}
```

此处的`search(g, vi)`函数分为深度优先搜索（DFS）和广度优先搜索（BFS）两种。

###### DFS

深度优先搜索 DFS（Depth_First Search）是指按照深度方向搜索，类似于树的先根遍历：

1. 从图中某个顶点$v_0$出发，首先访问$v_0$
2. 找出刚访问过的顶点$v_i$的第一个未被访问的邻接点，然后访问该顶点。重复此步骤，直到当前的顶点没有未被访问的邻接点为止
3. 返回前一个访问过的顶点，找出该顶点的下一个未被访问的邻接点，访问该顶点。转2

与树的先根遍历一样，DFS 算法是递归技术实现：

```c
void DepthFirstSearch(Graph g, int v0)
{
    int w;
    visit(v0);
    visited[v0] = 1;
    w = FirstAdjVertex(g, v0); //找v0的一个邻接点
    while (w != -1)            //邻接点存在
    {
        if (!visited[w])
            DepthFirstSearch(g, w);
        w = NextAdjVertex(g, v0, w); //找v0的下一个邻接点
    }
}
```

###### 邻接矩阵实现DFS

```c
void DepthFirstSearch(AdjMatrix g, int v0)
{
    int vj;
    visit(v0);
    visited[v0] = 1;
    for (vj = 0; vj < g.vexnum; vj++)
        if (!visited[vj] && g.arcs[v0][vj].adj == 1)
            DepthFirstSearch(g, vj);
}
```

###### 邻接表实现DFS

```c
void DepthFirstSearch(AdjList g, int v0)
{
    visit(v0);
    visited[v0] = 1;
    ArcNode *p = g.vertex[v0].firstarc;
    while (p)
    {
        if (!visited[p->adjvex])
            DepthFirstSearch(g, p->adjvex);
        p = p->nextarc;
    }
}
```

###### 深度优先生成树

显然，DFS 算法将生成一个覆盖 n 个点，有 n-1 条边的访问序列，也就是树，我们可以进一步修改 DFS 算法使其输出深度优先生成树：

```c
void DepthFirstSearch(Graph g, int v0)
{
    int w;
    visit(v0);
    visited[v0] = 1;
    w = FirstAdjVertex(g, v0);
    while (w != -1)
    {
        if (!visited[w])
        {
            Printf(v0, w); //修改内容
            DepthFirstSearch(g, w);
        }
        w = NextAdjVertex(g, v0, w);
    }
}
```

###### BFS

广度优先搜索 BFS（Breadth_First Search）是指按照广度方向搜索，类似于树的层次遍历：

1. 从图中某个顶点$v_0$出发，首先访问$v_0$
2. 依次访问$v_0$的各个未被访问的邻接点
3. 分别从这些邻接点（端结点）出发，依次访问它们的各个未被访问的邻接点（新的端结点）

与树的层次遍历一样，BFS 算法是队列技术实现：

```c
void BreadthFirstSearch(Graph g, int v0)
{
    int v, w;
    visit(v0);
    visited[v0] = 1;
    InitQueue(&Q);
    EnterQueue(&Q, v0);
    while (!IsEmpty(Q))
    {
        DeleteQueue(&Q, &v);
        w = FirstAdjVertex(g, v);
        while (w != -1)
        {
            if (!visited[w])
            {
                visit(w);
                visited[w] = 1;
                EnterQueue(&Q, w);
            }
            w = NextAdjVertex(g, v, w);
        }
    }
}
```

###### 广度优先生成树

类似于深度优先生成树，我们也可以进一步修改 BFS 算法使其输出广度优先生成树：

```c
void BreadthFirstSearch(Graph g, int v0)
{
    int v, w;
    visit(v0);
    visited[v0] = 1;
    InitQueue(&Q);
    EnterQueue(&Q, v0);
    while (!IsEmpty(Q))
    {
        DeleteQueue(&Q, &v);
        w = FirstAdjVertex(g, v);
        while (w != -1)
        {
            if (!visited[w])
            {
                visit(w);
                visited[w] = 1;
                Printf(v, w); //修改内容
                EnterQueue(&Q, w);
            }
            w = NextAdjVertex(g, v, w);
        }
    }
}
```

#### 图的应用

##### 图的连通性问题

###### 无向图的连通分量

调用搜索过程的次数就是该图连通分量的个数。

- 连通图仅调用一次搜索过程——连通分量为 1
- 非连通图需多次调用搜索过程——连通分量为此处次数

修改图的遍历算法即可实现统计无向图的连通分量：

```c
void TraverseGraph(Graph g)
{
    int vi, count;
    count = 0;
    for (vi = 0; vi < g.vexnum; vi++)
        visited[vi] = 0;
    for (vi = 0; vi < g.vexnum; vi++)
        if (!visited[vi])
        {
            search(g, vi);
            count++;
        }
}
```

###### 两顶点之间的简单路径



###### 生成树



###### 最小生成树

各边的代价之和最小的生成树称为最小（代价）生成树。

性质：设$N=(V,\{E\})$是一连通网，U 是顶点集 V 的一个非空子集。若$(u,v)$是一条具有最小权值的边，存在一棵包含边$(u,v)$的最小生成树。

由此性质，我们有两种算法实现：

1. 普里姆算法（加点法）

   思想：加点不构成回路

   1. 假设 N=(V,{E}) 是连通网，TE 为最小生成树中边的集合

   2. 初始 U={$u_0$}($u_0$∈V),TE=∅

   3. 在所有$u∈U, v∈V-U$的边中选一条代价最小的边$(u_0, v_0)$并入集合 TE，同时将$v_0$并入 U

   4. 重复第 2 步，n-1 直到 U=V 为止。

   此时，TE 中必含有 n-1 条边，则 T=(V,{TE}) 为N的最小生成树

   {{< admonition >}}

   普里姆算法时间复杂度为$O(n^2)$

   {{< /admonition >}}

2. 克鲁斯卡尔算法（加边法）

   思想：加边不构成回路

   1. 假设 N=(V,{E}) 是连通网，将 N 中的边按权值从小到大的顺序排列

   2. 将 n 个顶点看成 n 个集合

   3. 按权值由小到大的顺序选择边，选边应满足两个顶点不在同一个集合内，将该边放到生成树边的集合中。同时将该边的两个顶点所在的顶点集合合并

   4. 重复第 3 步，直到所有的顶点都在同一个顶点集合内

##### 有向无环图的应用1—拓扑排序

###### AOV-网

顶点表示活动的网：

- 顶点表示活动
- 弧表示活动间的优先关系

![image-20220213203039998](https://s2.loli.net/2022/02/13/byL37Zok5QEf84a.png)

###### 拓扑序列

有向图 G=(v,{E}) 中顶点的线性序列

图中任意两个顶点$v_i$、$v_j$，有一条从$v_i$到$v_j$的路径，则在拓扑序列中$v_i$必排在$v_j$之前。

###### 拓扑网的特性

1. 先行关系具有可传递性
2. 拓扑序列不唯一

###### 拓扑排序的基本思想

1. 选一个无前驱顶点输出
2. 删除此点及以它为起点的弧
3. 重复1、2，直到不存在无前驱顶点
4. 输出顶点数小于图中顶点数，则有向图中存在回路，否则输出顶点顺序即为拓扑序列

###### 拓扑排序算法

- 基于**邻接矩阵**：

  A 为有向图 G 的邻接矩阵，则有

  1. 找 G 中无前驱的顶点——在 A 中找到值全为 0 的列
  2. 删除以 i 为起点的所有弧——将矩阵中i对应的行置为全 0

  即选**全 0 列，置全 0 行**。

- 基于**邻接表**：

  附设一个存放各顶点入度的数组`indegree[]`

  1. 找 G 中无前驱的顶点——查找`indegree[i]`为零的顶点$v_i$
  2. 删除以 i 为起点的所有弧——对链在顶点 i 后面的所有邻接顶点 k，将对应的`indegree[k]`减 1

  为避免重复检测入度为零的顶点，设置一个入度为 0 点的[栈](/data_structure-3/#栈的顺序实现)，若某一顶点的入度减为0，则将它入栈。每当输出某一顶点，便将它从栈中删除。

  ```c
  bool TopoSort(AdjList G)
  {
      int i, k, count, indegree[MAX_VERTEX_NUM];
      Stack S;
      ArcNode *p;
      FindID(G, indegree); //求各顶点入度
      InitStack(&S);
      for (i = 0; i < G.vexnum; i++)
          if (!indegree[i])
              push(&S, i); //将入度0的顶点入栈
      count = 0;
      while (!IsEmpty(S))
      {
          pop(&S, &i);
          printf("%c", G.vertex[i].data);
          count++;
          p = G.vertex[i].firstarc;
          while (p)
          {
              k = p->adjvex;
              indegree[k]--;
              if (!indegree[k])
                  push(&S, k);
              p = p->nextarc;
          }
      }
      if (count < G.vexnum) //有回路
          return false;
      else
          return true;
  }
  ```

##### 有向无环图的应用2—关键路径

###### AOE-网

边表示活动的网：

- 顶点表示事件
- 弧表示活动
- 弧的权值表示活动所需时间

###### 在AOE-网中的基本概念

- 源点：唯一入度为零的顶点
- 汇点：唯一出度为零的顶点
- 关键路径：从源点到汇点的最长路径
- 关键活动：关键路径上的活动

###### 几个重要的定义

- $ve(i)$：事件$v_i$的最早发生时间——从源点到顶点$v_i$的最长路径长度

- $vl(i)$：事件$v_i$的最晚发生时间（在保证汇点按其最早发生时间发生的前提下）

  在求出$ve(i)$的基础上，可从汇点开始，按逆拓扑顺序向源点递推：

  $vl(n-1)=ve(n-1)$;

  $vl(i)=min${$vl(k)-dut(<i,k>)$}

- $e(i)$：活动$a_i$的最早开始时间

  活动$a_i$对应的弧为$<j,k>$，$e(i)$等于从源点到顶点 j 的最长路径的长度，即$e(i)=ve(j)$

-  $l(i)$：活动$a_i$的最晚开始时间

  活动$a_i$对应的弧为$<j, k>$，其持续时间为$dut(<j, k>)$则$l(i)=vl(k)-dut(<j, k>)$

- $a_i$的松弛时间（时间余量）：$a_i$的最晚开始时间与$a_i$的最早开始时间之差——$l(i)-e(i)$

  松弛时间（时间余量）为 0 的活动为关键活动

###### 求关键路径的基本步骤

1. 在对图中顶点进行拓扑排序过程中按拓扑序列求出每个事件的最早发生时间$ve(i)$
2. 按逆拓扑序列求每个事件的最晚发生时间$vl(i)$
3. 求出每个活动$a_i$的最早开始时间$e(i)$和最晚发生时间$l(i)$
4. 找出$e(i)=l(i)$的活动$a_i$，即为关键活动

###### 修改后的拓扑排序算法

通过拓扑顺序求出事件的最早发生时间

```c
int ve[MAX_VERTEX_NUM];             //每个顶点的最早发生时间
bool TopoOrder(AdjList G, Stack *T) // T为拓扑序列栈，S为0入度的顶点栈
{
    int count, i, j, k, indegree[MAX_VERTEX_NUM];
    ArcNode *p;
    Stack S;
    InitStack(T);
    InitStack(&S);
    FindID(G, indegree);
    for (i = 0; i < G.vexnum; i++)
        if (!indegree[i])
            push(&S, i); //将入度0的顶点入栈
    count = 0;
    for (i = 0; i < G.vexnum; i++)
        ve[i] = 0; //初始化最早发生时间
    while (!IsEmpty(S))
    {
        pop(&S, &j);
        push(T, j);
        count++;
        p = G.vertex[i].firstarc;
        while (p)
        {
            k = p->adjvex;
            indegree[k]--;
            if (!indegree[k])
                push(&S, k);
            if (ve[j] + p->info > ve[k]) //事件最早发生时间
                ve[k] = ve[j] + p->info;
            p = p->nextarc;
        }
    }
    if (count < G.vexnum)
        return false;
    else
        return true;
}
```

###### 求关键路径的实现算法

```c
bool CriticalPath(AdjList G)
{
    ArcNode *p;
    int i, j, k, dut, ei, li, vl[MAX_VERTEX_NUM];
    char tag;
    Stack T;
    if (!TopoOrder(G, &T)) //拓扑序列求ve
        return false;
    for (i = 0; i < G.vexnum; i++)
        vl[i] = ve[i]; //初始化
    while (IsEmpty(T))
    {
        pop(&T, &j); //逆拓扑序列求vl
        p = G.vertex[j].firstarc;
        while (p)
        {
            k = p->adjvex;
            dut = p->info;
            if (vl[k] - dut < vl[j])
                vl[j] = vl[k] - dut;
            p = p->nextarc;
        }
    }
    for (j = 0; j < G.vexnum; j++) //求ei，li和关键活动
    {
        p = G.vertex[j].firstarc;
        while (p)
        {
            k = p->adjvex;
            dut = p->info;
            ei = ve[j];
            li = vl[k] - dut;
            tag = (ei == li) ? '*' : ' ';
            printf("%c %c %d %d %d %c\n", G.vertex[j].data, G.vertex[k].data, dut, ei, li, tag); //输出关键活动
            p = p->nextarc;
        }
    }
    return true;
}
```

##### 最短路径

###### 求某一顶点到其它各顶点的最短路径

迪杰斯特拉（Dijkstra）算法

- 基本思想：

  下一条最短路径是弧$(v_0,v_x)$或是中间经过 S 的某些顶点到达$v_x$的路径——按路径长度递增顺序产生最短路径

- 算法步骤：

  1. g 为**邻接矩阵**表示的带权图，将$v_0$到其余顶点的路径长度初始化为权值

  2. 选择$v_k$，使得$v_k$为目前求得的下一条从$v_0$出发最短路径的终点

  3. 修改从$v_0$出发到集合 V-S 上任一顶点$v_i$的最短路径的长度

     如果$dist [k]+g.arcs\[k][i]<dist[i]$则修改$dist[i]=dist[k]+g.arcs\[k][i]$

  4. 重复2、3 n-1 次，即按最短路径长度的递增顺序，逐个求出$v_0$到图中其它每个顶点的最短路径

- 算法描述：

  ```c
  ShortestPath_DJS(AdjMatrix g, int vo, VRType dist[MAX_VERTEX_NUM], VertexType path[MAX_VERTEX_NUM])
  //存放i的当前最短路径长度，存放i的当前最短路径
  {
      VertexType s; //已找到最短路径的终点集合
      int i, k, t, min;
      for (i = 0; i < g.vexnum; i++) //初始化dist[i]、path[i]
      {
          InitList(&path[i]);
          dist[i] = g.arcs[v0][i];
          if (dist[i] < MAX)
          {
              AddTail(&path[i], g.vexs[v0]);
              AddTail(&path[i], g.vexs[i]);
              InitList(&s);
              AddTail(&s, g.vexs[v0]);
          }
      }
      for (t = 1; t < g.vexnum - 1; t++)
      {
          min = MAX;
          for (i = 0; i < g.vexnum; i++)
              if (!Member(g.vexs[i], s) && dist[i] < min)
              {
                  k = i;
                  min = dist[i];
              }
          if (min = INFINITY)
              return;
          AddTail(&s, g.vexs[k]); //选择最小距离vk
          for (i = 0; i < g.vexnum; i++)
              if (!Member(g.vexs[i], s) && (dist[k] + g.arcs[k][i] < dist[i])) //基于vk修正距离和路径
              {
                  dist[i] = dist[k] + g.arcs[k][i];
                  path[i] = path[k];
                  AddTail(&path[i], g.vexs[i]); // path[i]=path[k]∪{vi}
              }
      }
  }
  ```

  {{< admonition >}}

  此算法时间复杂度为$O(n^2)$

  {{< /admonition >}}

###### 求任意一对顶点间的最短路径

佛罗依德算法（Floyd）

- 基本思想

  生成中间顶点序号不大于 0 … 不大于 n一1 的路径矩阵

  图 g 用**邻接矩阵法**表示，求图 g 中任意一对顶点$v_i$、$v_j$间的最短路径

  将$v_i$到$v_j$的最短的路径长度初始化为$g.arcs\[i][j]$，然后进行如下 n 次比较和修正

  1. 在$v_i$、$v_j$间加入顶点$v_0$，比较$(v_i,v_0,v_j)$和$(v_i,v_j)$路径的长度，取其中较短路径作为$v_i$到$v_j$的且中间顶点号不大于 0 的最短路径

  2. 在$v_i$，$v_j$间加入顶点$v_1$，得$(v_i,…,v_1)$和$(v_1,…,v_j)$，其中$(v_i,…,v_1)$是$v_i$到$v_1$的且中间顶点号不大于 0 的最短路径，$(v_1,…,v_j)$是$v_1$到$v_j$且中间顶点号不大于 0 的最短路径，这两条路径在上一步中已求出。将$(v_i,…,v_1,v_j)$与$v_i$到$v_j$中间顶点号不大于 0 的最短路径比较，取较短的路径作为$v_i$到$v_j$的且中间顶点号不大于 1 的最短路径
  3. 依次类推，经过 n 次比较和修正,在第 n-1 步，将求得$v_i$到$v_j$且中间顶点号不大于 n-1 的最短路径，这必是从$v_i$到$v_j$的最短路径

- 算法描述

  ```c
  ShortestPath_Floyd(AdjMatrix g, VRType dist[MAX][MAX], VertexType path[MAX][MAX])
  {
      int i, j, k;
      for (i = 0; i < g.vexnum; i++)
          for (j = 0; j < g.vexnum; j++)
          {
              InitList(&path[i][j]);
              dist[i][j] = g.arcs[i][j]; //初始化dist[i][j]和path[i][j]
              if (dist[i][j] < MAX)
              {
                  AddTail(&path[i][j], g.vexs[i]);
                  AddTail(&path[i][j], g.vexs[j]);
              }
          }
      for (k = 0; k < g.vexnum; k++)
          for (i = 0; i < g.vexnum; i++)
              for (j = 0; j < g.vexnum; j++)
                  if (dist[i][k] + dist[k][j] < dist[i][j])
                  {
                      dist[i][j] = dist[i][k] + dist[k][j];
                      path[i][j] = JoinList(path[i][k], path[k][j]);
                  }
  }
  ```

#### 例题

##### 求距离顶点$v_0$的最短路径长度为k的所有顶点

已知无向图 g，设计算法求距离顶点$v_0$的最短路径长度为 k 的所有顶点，要求尽可能节省时间。

显然，本题应使用广度优先搜索算法（BFS），从顶点$v_0$开始，将一步可达的、两步可达的 … 直至 k 步可达的所有顶点记录下来，同时用一个队列记录每个结点的层号，输出第 k+1 层的所有结点即为所求。

```c
void BFS_layer(AdjList g, int v0, int k)
{
    Queue Q;
    int v, w, layer;
    visit(v0);
    visited[v0] = 1;
    InitQueue(&Q);
    EnterQueue(&Q, v0, 1);
    while (Q.front != Q.rear) //队列非空
    {
        DeleteQueue(&Q, &v, &layer);
        if (layer == k + 1)
            printf("%d ", v);
        w = FirstAdjVertex(g, v);
        while (w != -1)
        {
            if (!visited[w])
            {
                visit(w);
                visited[w] = 1;
                EnterQueue(&Q, w, layer + 1);
            }
            w = NextAdjVertex(g, v, w);
        }
    }
}
```

