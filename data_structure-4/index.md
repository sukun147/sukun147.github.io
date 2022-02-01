# 数据结构学习(四)


<!--more-->

[本系列学习笔记链接](/categories/数据结构/)

# 数据结构——用C语言描述(四)

## 线性结构

### 串

#### 串的定义

- 字符串（string）：零个或多个字符组成的有限序列。记为
  $$
  S='a_1a_2…a_n'\ (n≥0),a_i∈V字符集合
  $$
  组成线性表的每个元素就是一个单字符串是特定的线性表

- 串的名字：**S**

- 串的值：单引号括起来的字符序列，可以是字母，数字或其他字符

- 串的长度：n，串中字符的个数

- 空串（Null String）：n=0 时的串

{{< admonition success 举个栗子>}}

`ch='hello world'`就是字符串长度为 11 ，串名为 ch 的字符串

{{< /admonition >}}

- 子串：串中任意个连续字符组成的子序列。
- 主串：包含子串的串。
- 求子串：`sub(主串，起始位置，长度)`

{{< admonition success 举个栗子>}}

`sub('china',2,2)='in'`

{{< /admonition >}}

模式串在主串中的位置（串的模式匹配）：从主串的起始位置起，模式串在主串中首次出现的位置序号

{{< admonition success 举个栗子>}}

主串`'Beijing,China'`模式串`in`：

若起始位置为 0，则模式串第一个字符在主串中的位置为 4；

若起始位置为 5，则模式串第一个字符在主串中的位置为 10。

{{< /admonition >}}

串相等：当且仅当两个串长度相等且对应位置的字符都相等。

空串与空格串的区别：

- 空格串：一个或多个空格组成的串，其长度为空格个数。
- 空串：无任何字符组成的串，其长度为零。

对于串 ADT：

数据对象：字符集
$$
D=\{a_i|\ a_i∈CharacterSet,\ i=1,2,…,n;\ n≥0\}
$$
数据关系：线性关系
$$
R=\{<a_{i-1},a_i>|\ a_{i-1},a_i∈D,\ i=1,2,…,n;\ n≥0\}
$$
有 13 种基本操作运算：

1. 赋值`StrAsign(S, str)`
2. 插入`StrInsert(S, pos, T)`
3. 删除`StrDelete(S, pos, len)`
4. 拷贝`StrCopy(S, T)`
5. 判空`StrEmpty(S)`
6. 比较`StrCompare(S, T)`
7. 求串长`StrLength(S)`
8. 清空`StrClear(S)`
9. 连接`StrCat(S, T)`
10. 求子串`SubString(Sub, S, pos, len)`
11. 串的模式匹配`Strlndex(S, T, pos)`
12. 置换`StrReplace(S, T, V)`
13. 消除`StrDestroy(s)`

#### 串的顺序存储

##### 定长顺序串

```c
#define MAXLEN 20
typedef struct 
{
    char ch[MAXLEN];
    int len;
} SString;
```

###### 串插入

在进行串的插人时，插人位置 pos 将串分为两部分（假设为 A、B，长度为 LA、LB），及待插人部分（假设为 C，长度为 LC），则串由插人前的 AB 变为 ACB ，可能有三种情况：

1. 插入后串长 (LA+LB+LC) ≤ MAXLEN：

   将 B 后移 LC 个元素位置，再将 C 插人

2. 插入后串长 > MAXLEN 且 pos + LC < MAXLEN：

   B 后移时会有部分字符被舍弃

3. 插入后串长 > MAXLEN 且 pos + LC > MAXLEN：

   B 的全部字符被舍弃（不需后移），并且在插人时也有部分字符被舍弃

```c
bool StrInsert(SString *s, int pos, SString t) //在串s中序号为pos的字符之前插入串t
{
    int i;
    if (pos < 0 || pos > s->len) //插入位置不合法
        return false;
    if (s->len + t.len <= MAXLEN) //插入后串长≤MAXLEN
    {
        for (i = s->len + t.len - 1; i >= t.len + pos; i--)
            s->ch[i] = s->ch[i - t.len];
        for (i = 0; i < t.len; i++)
            s->ch[i + pos] = t.ch[i];
        s->len += t.len;
    }
    else if (pos + t.len <= MAXLEN) //插入后串长>MAXLEN,但串t可以全部插入
    {
        for (i = MAXLEN - 1; i > t.len + pos - 1; i--)
            s->ch[i] = s->ch[i - t.len];
        for (i = 0; i < t.len; i++)
            s->ch[i + pos] = t.ch[i];
        s->len = MAXLEN;
    }
    else //串t的部分要舍弃
    {
        for (i = 0; i < MAXLEN - pos; i++)
            s->ch[i + pos] = t.ch[i];
        s->len = MAXLEN;
    }
    return true;
}
```

###### 串的模式匹配

```c
int index(SString s, int pos, SString t) //从主串s的pos位置起,与模式串t逐位匹配
{
    int i, j;
    if (!t.len)
        return 0; //空串是任意串的子串
    i = pos;
    j = 0;
    while (i < s.len && j < t.len)
        if (s.ch[i] == t.ch[j])
        {
            i++;
            j++;
        }
        else
        {
            i = i - j + 1; //对应字符不等，主串从起始位置下一位起
            j = 0;
        }
    if (j >= t.len)
        return i - j;
    else
        return -1; //匹配不成功，返回-1
}
```

注意到时间主要耗费在了 while 循环即回溯中，这种算法也称为 BF 算法（或暴力匹配算法）其时间复杂度为
$$
O(s.len*t.len)
$$
对于该算法的改进，重点在于实现非回溯，其中的佼佼者 KMP 算法能在时间复杂度上达到
$$
O(n+m)
$$


这里仅给出其代码，详解：[KMP算法（快速模式匹配算法）C语言详解](http://data.biancheng.net/view/180.html)

```c
#include <stdio.h>
#include <string.h>

void Next(char *T, int *next)
{
    int i = 1;
    next[1] = 0;
    int j = 0;
    while (i < strlen(T))
    {
        if (j == 0 || T[i - 1] == T[j - 1])
        {
            i++;
            j++;
            next[i] = j;
        }
        else
        {
            j = next[j];
        }
    }
}

int KMP(char *S, char *T)
{
    int next[10];
    Next(T, next); //根据模式串T,初始化next数组
    int i = 1;
    int j = 1;
    while (i <= strlen(S) && j <= strlen(T))
    {
        // j==0:代表模式串的第一个字符就和当前测试的字符不相等；S[i-1]==T[j-1],如果对应位置字符相等，两种情况下，指向当前测试的两个指针下标i和j都向后移
        if (j == 0 || S[i - 1] == T[j - 1])
        {
            i++;
            j++;
        }
        else
        {
            j = next[j]; //如果测试的两个字符不相等，i不动，j变为当前测试字符串的next值
        }
    }
    if (j > strlen(T))
    { //如果条件为真，说明匹配成功
        return i - (int)strlen(T);
    }
    return -1;
}
int main()
{
    int i = KMP("ababcabcacbab", "abcac");
    printf("%d", i);
    return 0;
}
```

##### 堆串

堆：系统将一个地址连续、容量很大的存储空间作为字符串的可用空间。

每建立新串时，需提供串值的起始位置指针和串长度,示统从堆串区分配空间。

#### 串的链式存储

![image-20220128183940085](https://s2.loli.net/2022/01/28/TLSxFDA2lnosCWK.png)

块链串是有头尾指针的链表，其中单个结点称为块。

- 结点大小：data 域存放字符个数
  - 结点大小为 1 时，存储密度低，处理简单，是单链表
  - 结点大小大于 1 时，存储密度高，管理复杂
- 链域大小：next 域占用字符个数

![链串](https://cdn.timsrc.com/%E4%B8%B2%E5%80%BC%E7%9A%84%E9%93%BE%E8%A1%A8%E5%AD%98%E5%82%A8%E7%A4%BA%E6%84%8F%E5%9B%BE-1562211880222)

```c
#define BLOCK_SIZE 4

typedef struct 
{
    char ch[BLOCK_SIZE];
    struct Block *next;
} Block;

typedef struct 
{
    Block *head;
    Block *tail;
    int len;
} BLString;
```

这里仅给出普通模式匹配算法（BF算法）用链式存储的实现，其他内容的具体操作与单链表类似，请看[数据结构学习(二)](/data_structure-2/#单链表)

```c
Block *StrIndex(BLString *s, BLString *t)
{
    Block *sp, *tp, *start;
    if (!t->len)
        return s->head->next;
    start = s->head->next;
    sp = start;
    tp = t->head->next;
    while (sp != NULL && tp != NULL)
    {
        if (sp->ch == tp->ch)
        {
            sp = sp->next;
            tp = tp->next;
        }
        else
        {
            start = start->next;
            sp = start;
            tp = t->head->next;
        }
    }
    if (tp == NULL)
        return start;
    else
        return NULL;
}
```

{{< admonition question >}}

假设主串`S='aaabbbababaabb'` ,模式串`P='abaa'` ，用 BF 算法从主串的第 6 个字符开始进行模式匹配，需要做多少趟匹配，第2趟匹配做多少次比较？

{{< /admonition >}}

{{< admonition success 答案>}}

4趟，4次

{{< /admonition >}}

