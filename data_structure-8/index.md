# 数据结构学习(八)


<!--more-->

# 数据结构——用C语言描述(八)

## 技术

### 查找

#### 查找的基本概念

- 列表：由同一类型的数据元素（或记录）构成的集合，可利用任意数据结构实现

- 关键字：数据元素的某个数据项的值可标识列表中的一个数据元素

  **主关键字**可**唯一标识**列表中的一个数据元素，否则为次关键字

- 查找：根据给定的关键字值，在特定的列表中确定一个其关键字与给定值相同的数据元素，并返回该数据元素在列表中的位置

  1. 查找对象 K（找什么）

  2. 查找范围 L（在哪找）

  3. K 在 L 中的位置（查找的结果）

- 平均查找长度（ASL）：查找过程中对关键码的比较次数的平均值
  $$
  ASL=P_1C_1+P_2C_2+…+P_nC_n=\sum_{i=1}^nP_iC_i\\\
  P_i为查找概率，C_i为查找次数
  $$

- 查找基本方法：

  1. 线性表查找法
  2. 树表式查找法
  3. 计算式（哈希）查找法

#### 基于线性表的查找法

##### 顺序查找法

###### 数据类型定义

```c
#define KeyType int
#define OtherType int
#define LIST_SIZE 20
typedef struct
{
    KeyType key;
    OtherType other_data;
} RecordType;
typedef struct
{
    RecordType r[LIST_SIZE + 1]; // r[0]为工作单元
    int length;
} RecordList;
```

###### 顺序结构算法

此处为从后往前的顺序：

```c
//不设置监视哨，在顺序表中查找关键字等于k的元素
int SeqSearch_1(RecordList L, KeyType k)
{
    int i;
    for (i = L.length; i >= 1 && L.r[i].key != k; i--)
        ;
    if (i >= 1)
        return i;
    else
        return 0;
}
//设置监视哨，顺序表L由后往前查找关键字k，查到返回k在L中的位置，否则返0
int SeqSearch_2(RecordList L, KeyType k)
{
    int i;
    L.r[0].key = k;
    for (i = L.length; L.r[i].key != k; i--)
        ;
    return i;
}
```

###### 性能分析

$$
n为表长，c_i为比较次数\\\
c_1=n-1+1\\\
c_2=n-2+1\\\
…\\\
c_n=n-n+1\\\
即c_i=n-i+1\\\
查找每个元素的概率相等，即P_i=\frac{1}{n}\\\
查找成功时平均查找长度为\\\
ASL_{succ}=\sum_{i=1}^nP_iC_i=\frac{1}{n}\sum_{i=1}^nC_i=\frac{1}{n}\sum_{i=1}^n(n-i+1)=\frac{n+1}{2}
$$

{{< admonition >}}

ASL为$O(n)$

{{< /admonition >}}

##### 折半查找法

###### 定义

折半查找（二分查找），要求：

1. 采用顺序存储
2. 关键字有序排列

###### 基本过程

中间记录关键字与查找关键字比较：

如果两者相等，则查找成功；

否则如查找关键字小于中间位置关键字，则查前部子表，否则查找后部子表。

###### 算法

```c
int BinSrchZ(SqList L, KeyType k)
{
    int low, high, mid;
    low = 1;
    high = L.length; //置区间初值
    while (low <= high)
    {
        mid = (low + high) / 2;
        if (k == L.r[mid].key)
            return mid; //查找成功
        else if (k < L.r[mid].key)
            high = mid - 1; //继续在前半区间查找
        else
            low = mid + 1; //继续在后半区间查找
    }
    return 0;
}
```

###### 性能分析

折半查找过程可用判定树描述判定树结构：

- 树中每一结点表示表中一记录结点值记录在表中的位置
- 从根到被查结点路径关键字比较次数为被查结点层数
- 成功进行最多比较次数不超过树深度$[\log_2n] +1$

假定表的长度$n=2h-1$，则相应判定树必为深度是 h 的满二叉树，$h=\log_2(n+1)$

折半查找成功的平均查找长度：
$$
ASL_{bs}=\sum_{i=1}^nP_iC_i=\frac{1}{n}\sum_{j=1}^hj×2^{j-1}=\frac{n+1}{n}\log_2(n+1)-1
$$
每个记录的查找概率相等，j 为每层的比较次数，$2^{j-1}$为每层的元素个数

{{< admonition >}}

ASL为$O(n\log_2n)$

{{< /admonition >}}

- 优点：
  - 比较次数少
  - 查找速度快
  - 平均性能好
  - 适用于固定长度频繁查找的有序表
- 缺点：
  - 要求待查表为有序表
  - 插入删除时需再排序

##### 分块查找法

###### 对所查表要求

- 等长分块，最后一块可不满
- 块内无序
- 块间有序

###### 基本思想

- 分块构造索引表
- 定块可用顺序或折半查找关键字 k 与索引表关键字比较，确定该查记录所在块
- 块内顺序查找

###### 平均查找长度

查找索引表的平均查找长度$L_B$，相应块内顺序查找的平均查找长度$L_W$

平均查找长度：$ASL_{bs}=L_B+L_W$

假定将长度为 n 的表分成 b 块，每块含 s 个元素，则$b=\frac{n}{s}$，

又假定表中每个元素的查找概率相等，则每个索引项的查找概率为$\frac{1}{b}$，块中每个元素的查找概率为$\frac{1}{s}$。则有

- 顺序法平均查找长度：
  $$
  L_B=\frac{1}{b}\sum_{j=1}^bj=\frac{b+1}{2}\\\
  L_W=\frac{1}{s}\sum_{i=1}^si=\frac{s+1}{2}\\\
  $$
  将$b=\frac{n}{s}$代入得
  $$
  ASL_{bs}=\frac{\frac{n}{s}+1}{2}+1\\\
  =\frac{b+1+s+1}{2}\\\
  =\frac{b+s}{2}+1
  $$

- 折半法平均查找长度：
  $$
  L_B=\log_2(b+1)-1\\\
  ASL_{bs}=\log_2(b+1)-1+\frac{s+1}{2}\\\
  =\log_2(\frac{s}{n}+1)+\frac{s-1}{2}
  $$

#### 基于树的查找法

步骤：

1. 将待查表组织成特定树
2. 在树结构上实现查找

有三种树：

1. 二叉排序树
2. 平衡二叉树
3. B 树

此处重点讲述二叉排序树

##### 二叉排序树

###### 定义

二叉树排序树或者是一棵空树，或者是具有如下性质的二叉树：

1. 若它的左子树非空，则左子树上所有结点的值均小于根结点的值
2. 若它的右子树非空，则右子树上所有结点的值均大于根结点的值
3. 它的左右子树也分别为二叉排序树

![image-20220217154039461](https://s2.loli.net/2022/02/17/slmnZP2eaRKA95z.png)

###### 插入和生成

- 若二叉排序树是空树，则`key`成为二叉排序树的根
- 若二叉排序树非空，`key`与二叉排序树的根比较：
  1. `key`的值小于根结点的值，则将`key`插入左子树
  2. `key`的值大于等于根结点的值，则将`key`插入右子树

```c
void CreateBST(BSTree *bst) //从键盘输入元素的值，创建相应的二叉排序树
{
    KeyType key;
    *bst = NULL;
    scanf("%d", &key);
    while (key != ENDKEY)
    {
        InsertBST(bst, key);
        scanf("%d", &key);
    }
}
void InsertBST(BSTree *bst, KeyType key) // key插入到bst为根的二叉排序树，从根比
{
    BiTree s;
    if (!*bst)
    {
        s = (BSTree)malloc(sizeof(BSTNode));
        s->key = key;
        s->LChild = s->RChild = NULL;
        *bst = s;
    }
    else if (key < (*bst)->key)
        InsertBST(&((*bst)->LChild), key);
    else
        InsertBST(&((*bst)->RChild), key);
}
```

{{< admonition >}}

插入一个结点`InsertBST(bst, key)`时间复杂度为$O(\log_2n)$，创建二叉排序树 n 个结点的时间复杂度为$O(n\log_2n)$。

{{< /admonition >}}

{{< admonition tip>}}

对二叉排序树进行中序遍历，一定会得到一个递增有序序列。

{{< /admonition >}}

###### 查找

方法：

根据二叉排序树的特点，关键字 k 与根 t 比较：

1. key = t：返回根结点地址
2. key < t：进一步查左子树
3. key > t：进一步查右子树

算法：

- 递归

  ```c
  BSTree SearchBST(BSTree bst, KeyType key)
  {
      if (!bst)
          return NULL;
      else if (bst->key == key)
          return bst;
      else if (bst->key > key)
          return SearchBST(bst->LChild, key);
      else
          return SearchBST(bst->RChild, key);
  }
  ```

- 非递归

  ```c
  BSTree SearchBST_2(BSTree bst, KeyType key) //非递归查找
  {
      BSTree q;
      q = bst;
      while (q)
      {
          if (q->key == key)
              return q; //查找成功
          else if (q->key > key)
              q = q->LChild;
          else
              q = q->RChild;
      }
      return NULL; //查找失败
  }
  ```

###### 查找性能

平均查找长度和二叉排序树的形态有关：

就平均性能而言，二叉排序树上的查找和二分查找相差不大，并且二叉排序树上的插入和删除结点十分方便，无需移动大量结点

- 最好情况：

  二叉排序树接近二分平衡树，平均查找长度大约是$\log_2n$

- 最坏情况：

  得到一棵深度为 n 的单支树，平均查找长度和单链表上的顺序查找相同，是$\frac{n+1}{2} $。

##### 平衡二叉树

###### 定义

平衡二叉排序树又称为 AVL 树。

一棵平衡二叉排序树或者是空树，或者是具有下列性质的二叉排序树：

1. 左子树与右子树的高度之差的绝对值小于等于 1
2. 左子树和右子树也是平衡二叉排序树

###### 平衡因子

平衡因子（balance factor）即结点的左子树深度与右子树深度之差。

对一棵平衡二叉排序树而言，其所有结点的平衡因子只能是-1、0或1。

![image-20220217161551386](https://s2.loli.net/2022/02/17/OgEZ8dYelJku3Mt.png)

#### 计算式查找法——哈希法

###### 哈希的基本思想

比较式查找通过对关键字的一系列查找比较实现，效率与查找表长有关，当关键字可能取值的集合远远大于实际表长时，比较式查找效率低。

而计算式查找对关键字计算得到对应元素的地址$H(key)=Addr$

哈希法（散列法、杂凑法）基本思想：

1. 哈希函数：

   建立哈希函数$p=H(key)$，计算关键字 key 在哈希表中的存储位置 p。

2. 冲突：

   冲突问题:若关键字$key_1 key_2$，哈希函数值$H(key_1)= H(key_2)$，则发生冲突。

冲突原因：

1. 必然性：由于关键字可能的取值空间远远大于哈希表的地址空间，冲突不可避免
2. 可能性：哈希函数$H(key)$的散列性能不好，可能加剧冲突发生

###### 哈希函数的构造方法

原则：

1. 函数本身便于计算
2. 计算出来的地址分布均匀

方法：

1. 数字分析法：

   从关键字中选出分布较均匀的若干位，构成哈希地址

2. 平方取中法：

   先求出关键字的平方值，然后按需要取平方值的中间几位作为哈希地址

3. 分段叠加法：

   按哈希表地址位数将关键字分成位数相等的几部分(最后一部分可以较短），然后将这几部分相加，舍弃最高进位后的结果就是该关键字的哈希地址。

4. 除留余数法：

   哈希函数为$H(K)=K\\%P$，其中%为模 P 取余运算，表长为 m，P 应为小于等于 m 的最大素数

5. 伪随机数法：

   采用一个伪随机函数做哈希函数，即$h(key)=random(key)$。

###### 处理冲突的方法

1. **开放定址法**

   当关键字 key 的哈希地址$p=H (key)$出现冲突时，以 p 为基础，产生另一个哈希地址$p_1$，如果$p_1$仍然冲突，再以 p 为基础，产生另一个哈希地址$p_2$，…，直到找出一个不冲突的哈希地址$p_i$ ，将相应元素存入其中。通用的再散列函数形式：$H_i=(H(key)+d_i)\\%m$

   形成$d_i$的方法：

   1. 线性探测再散列

      $d_i=1,2,3,…,m-1$

      冲突发生时，顺序查看表中下一单元，直到找出一个空单元或查遍全表

   2. 二次探测再散列

      $d_i=1^2,-1^2,2^2,-2^2,…,k^2,-k^2$

      特点:冲突发生时，在表的左右进行跳跃式探测，比较灵活

   3. 伪随机探测再散列

      $d_i=伪随机数序列$

2. 再哈希法

   同时构造多个不同的哈希函数：$H_i=RH_1(key)\ i=1,2,…,k$

   当哈希地址$H_i=RH_1(key)$发生冲突时，再计算$H_i=RH_2(key)$……直到冲突不再产生。

   这种方法不易产生聚集，但增加了计算时间。

3. **链地址法**

   ![链地址法](https://s2.loli.net/2022/02/18/WIV1uPTQ9kUgD5j.png)

4. 建立公共溢出区

   将哈希表分为基本表和溢出表两部分，凡是和基本表发生冲突的元素，一律填入溢出表

###### 哈希表的查找过程

哈希表的查找过程与哈希表的创建过程规则一致

注意：查到空单元时表示找不到

查找过程：

1. 计算$p_0=hash(K)$

2. 如果单元$p_0$为空，则所查元素不存在

3. 如果单元$p_0$中元素的关键字为K，则找到所查元素

4. 否则重复下述解决冲突的过程：

   按解决冲突的方法，找出下一个哈希地址$p_i$

   1. 如果单元$p_i$为空，则所查元素不存在
   2. 如果单元$p_i$中元素的关键字为K，则找到所查元素

查找算法：

```c
int HashSearch(HashTable ht, KeyType K)
{
    int i, p0, pi;
    p0 = hash(K);
    if (ht[p0].key == NULLKEY)
        return -1;
    else if (ht[p0].key == K)
        return 0;
    else //线性探测再散列
    {
        for (i = 1; i <= m - 1; i++)
        {
            pi = (p0 + i) % m;
            if (ht[pi].key == NULLKEY)
                return -1;
            else if (ht[pi].key == K)
                return pi;
        }
        return -1;
    }
}
```

###### 哈希法性能分析

影响比较次数的因素：

1. 哈希函数

2. 处理冲突的方法

3. 装填因子

哈希表的装填因子 α 的定义：
$$
α=\frac{哈希表中元素个数}{哈希表的长度}
$$
α 可描述哈希表的装满程度。显然，α 越小，发生冲突的可能性越小；而 α 越大，发生冲突的可能性也越大。

等概率情况下查找成功的平均查找长度：
$$
ASL_{succ}=\frac{1}{表中置入元素个数n}\sum_{i=1}^nC_i
$$
等概率情况下查找不成功的平均查找长度公式
$$
ASL_{unsucc}=\frac{1}{哈希函数取值个数r}\sum_{i=0}^{r-1}C_i
$$

1. 线性探测再散列：
   $$
   ASL_{succ}≈\frac{1+\frac{1}{1-α}}{2}\\\
   ASL_{unsucc}=\frac{1+\frac{1}{(1-α)^2}}{2}
   $$

2. 伪随机探测再散列、二次探测再散列、再哈希：
   $$
   ASL_{succ}≈-\frac{\ln(1-α)}{α}\\\
   ASL_{unsucc}=\frac{1}{1-α}
   $$

3. 链址法：
   $$
   ASL_{succ}≈1+\frac{α}{2}\\\
   ASL_{unsucc}=α+e^{-α}
   $$
