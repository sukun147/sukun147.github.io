# MySQL学习(二)


<!--more-->

## MySQL学习(二)

### 数据库的设计

#### 多表之间的关系

分类：

1. 一对一

   eg，人与身份证号码

2. 一对多（多对一）

   eg，员工与部门

3. 多对多

   学生与课程

##### 一对多（多对一）

`emp员工表`——1

| eid  | name | age  | emp_dept_fk |
| ---- | ---- | ---- | ----------- |
| 1    | 张三 | 24   | 1           |
| 2    | 李四 | 24   | 1           |
| 3    | 王五 | 35   | 2           |

`dept部门表`——n

| id   | name   |
| ---- | ------ |
| 1    | 财务部 |
| 2    | 销售部 |

在多的一方建立外键，指向一的一方的主键。

##### 多对多

`stu学生表`——m

| sid  | name | age  |
| ---- | ---- | ---- |
| 1    | 张三 | 18   |
| 2    | 李四 | 18   |
| 3    | 王五 | 19   |

`class课程表`——n

| cid  | name |
| ---- | ---- |
| 1    | 语文 |
| 2    | 数学 |

t_stu_class`中间表`

| sid  | cid  |
| ---- | ---- |
| 1    | 1    |
| 1    | 2    |
| 3    | 2    |

多对多关系实现需要借助第三张中间表。中间表至少包含两个字段，这两个字段作为第三张表的外键，分别指向两张表的主键。

##### 一对一

`stu学生表`

| id   | name | age  | cid  |
| ---- | ---- | ---- | ---- |
| 1    | 张三 | 18   | 1    |
| 2    | 李四 | 19   | 2    |

`card身份证表`

| id   | number  |
| ---- | ------- |
| 1    | 5105242 |
| 2    | 5105241 |

一对一关系实现，可以在任意―方添加唯―外键指向另一方的主键。

当然，对于一对一关系我们还可以直接合为一张表。

##### 注意事项

在实际开发中，我们应该尽量**避免使用外键**，而应该利用程序逻辑保证数据的正确性。

#### 数据库范式

设计关系数据库时，遵从不同的规范要求，设计出合理的关系型数据库，这些不同的规范要求被称为不同的范式，各种范式呈递次规范，越高的范式数据库冗余越小。

关系数据库有六种范式：第一范式（1NF）、第二范式（2NF）、第三范式（3NF）、巴斯-科德范式（BCNF）、第四范式(4NF）和第五范式（5NF，又称完美范式）。

在日常工作生活中，仅学习前三种范式已经足以应付大多数情况。

> 本处数据库范式与离散数学中的范式并无直接联系。

![image-20220512214319230](https://s2.loli.net/2022/05/12/ToOtHrE91b6JSC8.png)

##### 第一范式（1NF）

每一列都是不可分割的原子数据项

> **规范化：** 一个低一级的关系模式通过模式分解可以转化为若干个高一级范式的关系模式的集合，这个过程叫做规范化。

![初始表](https://s2.loli.net/2022/05/13/DPKsjT3yAlVnq4o.png)

而上表显然不满足不可分割的要求，因此不满足第一范式，可改为

![1NF](https://s2.loli.net/2022/05/13/Oo5HtPQgEK4Lxyd.png)

##### 第二范式（2NF）

在 1NF 的基础上，非码属性必须完全依赖于候选码（在 1NF 基础上消除非主属性对主码的部分函数依赖）

>**函数依赖**： 设 R(U) 是属性集 U 上的关系模式，X、Y 是 U 的子集。若对于 R(U) 的任意一个可能的关系 r，r 中不可能存在两个元组在 X 上的属性值相等，而在 Y 上的属性值不等，则称 Y 函数依赖于 X 或 X 函数确定 Y。
>
>简单地说，通过 X 属性（属性组）的值，可以**确定唯一 Y 属性**的值，则称 Y 函数依赖于 X。
>
>例如：
>
>1. 学号 --> 姓名，称姓名函数依赖于学号。
>2. (学号，课程名称) --> 分数，称分数函数依赖于 (学号、课程名称) 。
>
>**完全函数依赖**： 设 R(U) 是属性集 U 上的关系模式，X、Y 是 U 的子集。如果 Y 函数依赖于 X，且对于 X 的任何一个真子集 X’，都有Y 不函数依赖于 X’，则称 Y 对 X **完全函数依赖**。如果 Y 函数依赖于 X，但 Y 不完全函数依赖于 X，则称 Y 对 X **部分函数依赖**。
>
>简单地说，X --> Y，如果 X 是一个属性组，则 Y 属性值的确定需要依赖于 X 属性组中所有的属性值。
>
>例如：
>
>1. 学号 -/-> 分数，称分数不函数依赖于学号。
>2. (学号，课程名称) --> 姓名，学号 --> 姓名，称姓名对 (学号、课程名称) 部分函数依赖。
>3. (学号，课程名称) --> 分数，称分数对 (学号、课程名称) 完全函数依赖。
>
>![候选码，主属性，函数依赖，完全函数依赖](https://img-blog.csdnimg.cn/20190414144744240.png)
>
>**候选码：** 若关系中的某一属性组的值能**唯一地标识**一个元组，而其**子集不能**，则称该属性组为**候选码**。若一个关系中有多个候选码，则选定其中一个为**主码**。
>
>**主属性**： 所有**候选码的属性**称为主属性。不包含在任何候选码中的属性称为非主属性或非码属性。

显然，对于上表，(学号，课程名称) 为候选码，分数对其完全函数依赖，但姓名、系名、系主任对其部分函数依赖，因此可分为课程表+学生表：

![选课表](https://s2.loli.net/2022/05/13/CQ2fx6vMUHWcYDn.png)

![学生表](https://s2.loli.net/2022/05/13/nM8hzPjfJuWqFCv.png)

简单地说，第二范式是指每个表有且仅有一个数据项作为关键字或主键，其他数据项与关键字或者主键一一对应，即其他数据项完全依赖于关键字或主键。由此可知单主属性的关系均属于第二范式。

##### 第三范式（3NF）

在 2NF 基础上，任何非主属性不依赖于其它非主属性（在 2NF 基础上消除传递依赖）。

> **传递函数依赖**：
>
> ![传递依赖](https://img-blog.csdnimg.cn/20190414151055453.png)
>
> 例如：学号 --> 系名，系名 --> 系主任，称系主任对学号传递函数依赖。

显然，对于上表，系主任依赖于系名，可将学生表进一步分表为学生表+系表，而选课表不变：

![选课表](https://s2.loli.net/2022/05/13/CQ2fx6vMUHWcYDn.png)

![学生表](https://s2.loli.net/2022/05/13/Edx5hmcpMJ1ZbW9.png)

![系表](https://s2.loli.net/2022/05/13/XpLvxt3BV5ceHGK.png)

对于初始表，其存在以下问题：

1. 存在非常严重的数据冗余：姓名、系名、系主任
2. 数据添加存在问题：添加新开设的系和系主任时，数据不合法
3. 数据删除存在问题：同学毕业了，删除数据，会将系的数据一起删除。

现在，已全部解决！
