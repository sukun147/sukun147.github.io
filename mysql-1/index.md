# MySQL学习(一)


<!--more-->

## MySQL学习(一)

### 什么是SQL

**Structured Query Language**：结构化查询语言——定义了操作所有关系型数据库的规则。

以下是当前数据库 **TOP10** 所对应的数据库类型。

| Rank | DBMS                 | Database Model | Score   |
| :--- | -------------------- | -------------- | ------- |
| 1    | Oracle               | Relational     | 1254.82 |
| 2    | MySQL                | Relational     | 1204.16 |
| 3    | Microsoft SQL Server | Relational     | 938.46  |
| 4    | PostgreSQL           | Relational     | 614.46  |
| 5    | MongoDB              | Document       | 483.38  |
| 6    | Redis                | Key-value      | 177.61  |
| 7    | Elasticsearch        | Search engine  | 160.83  |
| 8    | IBM Db2              | Relational     | 160.46  |
| 9    | Microsoft Access     | Relational     | 142.78  |
| 10   | SQLite               | Relational     | 132.80  |

> 以上数据来自来自于 DB-Engines 发布的 [DB-Engines Ranking of database management systems, April 2022](https://db-engines.com/en/ranking)

我们发现其中大部分都是关系型数据库，也就是说，我们既可以用 SQL 操作 MySQL，也可以用其操作 Oracle。当然，对于不同的数据库，SQL 语句有细微差异，我们称其为**方言**。

### 什么是MySQL

MySQL 是一个关系型数据库管理系统，由瑞典 MySQL AB 公司开发，目前属于 Oracle 旗下产品。MySQL 是最流行的关系型数据库管理系统之一，在 WEB 应用方面，MySQL 是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件。MySQL 软件采用了双授权政策，分为社区版和商业版，由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发都选择 MySQL 作为网站数据库。

MySQL 所使用的 SQL 语言是用于访问数据库的最常用标准化语言。

MySQL 将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。

### MySQL安装

我这里采用最为便捷的方法——**docker** 容器启动。

依然是给出 shell 和 docker-compose 两种启动，注意修改密码等内容：

1. `shell`

   ```shell
   docker run -itd --name mysql -h mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=<your passwd> mysql
   ```

2. `docker-compose.yml`

   ```yaml
   version: "3"
   services:
   
     db:
       image: mysql
       container_name: mysql
       stdin_open: true
       tty: true
       environment:
         MYSQL_ROOT_PASSWORD: "root"
       ports:
         - "3306:3306"
       restart: always
       hostname: mysql
   ```

这就安装好了，输入命令`docker exec -it mysql bash`即可进入容器，按下`{ctrl}+{p}+{q}`即可退出容器但保持容器活跃，输入命令`mysql -u root -p`即可进入 mysql 命令行。

### MySQL使用

#### SQL通用语法

- SQL 语句以`;`或`\g`结尾。

- SQL 语句应用空格或缩进以代码格式化，便于阅读。

- SQL 语句中关键词尽量大写，但 SQL 语句对大小写不敏感。

- SQL 语句中注释有三种：

  ```mysql
  # 单行注释，MySQL特有
  -- 单行注释
  /*
  多行注释
  */
  ```

- SQL 分类：

  1. DDL（Data Definition Language）数据定义语言：
     用来定义数据库对象︰数据库，表，列等。关键字： `create`，`drop`，`alter`等。

  2. DML（Data Manipulation Language）数据操作语言：
     用来对数据库中表的数据进行增删改。关键字：`insert`，`delete`，`update`等。

  3. DQL（Data Query Language）数据查询语言：
     用来查询数据库中表的记录（数据）。关键字：`select`，`where`等。

  4. DCL（Data Control Language）数据控制语言（了解即可）：

     用来定义数据库的访问权限和安全级别，及创建用户。关键字：`GRANT`，`REVOKE`等。

##### DDL：操作数据库、表

###### 操作数据库

1. **C**（Create）：创建

   创建数据库`CREATE DATABASE 数据库名称;`

   判断不存在才创建`CREATE DATABASE IF NOT EXISTS test;`

   指定字符集创建`CREATE DATABASE test CHARACTER SET gbk;`

   综合：`CREATE DATABASE IF NOT EXISTS test CHARACTER SET gbk`——如果该数据库不存在则创建之，并指定字符集为 gbk。

2. **R**（Retrieve）：查询

   查询数据库名称`SHOW DATABASES;`

   我们会看到有四个数据库`information_schema`、`mysql`、`performance_schema`、`sys`，其用途如下：

   - `information_schema`：信息数据库，在其中的是视图而非基本表，这个库提供访问数据库元数据的方式。
   - `mysql`：核心数据库，存储数据库的用户、权限设置、关键字等控制管理信息。比如说，我们可以在`user`表中修改用户密码。
   - `performance_schema`：收集数据库服务器性能参数
   - `sys`：其中数据来自：`performance_schema`。目的是将`performance_schema`复杂度降低，让 DBA 能更好的阅读这个库里的内容，让 DBA 更快的了解 DB 的运行情况。

   - 查询数据库创建语句`SHOW CREATE DATABASE 数据库名称;`

     ![image-20220510173606574](https://s2.loli.net/2022/05/10/n2fxMb1qUXJT5YD.png)

3. **U**（Update）：修改

   修改数据库字符集`ALTER DATABASE 数据库名称 CHARACTER SET 字符集;`

   值得注意的是，该操作时修改库，不会对其中已有的表的字符集做出修改！

4. **D**（Delete）：删除

   删除数据库`DROP DATABASE 数据库名称;`

   判断存在才删除`DROP DATABASE IF EXISTS 数据库名称;`

5. 使用数据库

   查询当前正在使用的数据库名称`SELECT DATABASE();`

###### 操作表

1. **C**（Create）：创建

   - 语法

     ```mysql
     CREATE TABLE 表名(
         列名1 数据类型1,
         列名2 数据类型2
     );
     ```

   - 数据类型：详细见[MySQL 数据类型 | 菜鸟教程 (runoob.com)](https://www.runoob.com/mysql/mysql-data-types.html)

     1. `int`整数类型——`age INT`
     2. `double`小数类型——`score DOUBLE(5,2)`，小数最多五位，小数点后保留两位
     3. `date`日期类型，`yyyy-MM-dd`格式
     4. `datetime`日期类型，`yyyy-MM-dd HH:mm:ss`格式
     5. `timestamp`时间戳，`yyyy-MM-dd HH:mm:ss`格式，若不赋值则获取当前系统时间
     6. `varchar`字符串类型——`name VARCHAR(20)`，最大 20 个**字符**。

   - 示例：

     ```mysql
     CREATE TABLE student(
         id INT,
         name VARCHAR(20),
         age INT,
         score DOUBLE(5,2),
         birthday DATE,
         insert_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
     );
     ```
     
   - 复制表`CREATE TABLE 表名 LIKE 源表名;`

2. **R**（Retrieve）：查询

   - 查询库中所有表名称`SHOW TABLES;`
   - 查询表结构`DESC 表名;`
   - 查询表创建语句`SHOW CREATE TABLE 表名`

3. **U**（Update）：修改

   - 修改表名`ALTER TABLE 原表名 RENAME TO 新表名;`
   - 修改表字符集`ALTER TABLE 表名 CHARACTER SET 字符集;`
   - 添加列`ALTER TABLE 表名 ADD 列名 数据类型;`
   - 修改列`ALTER TABLE 表名 CHANGE 原列名 新列名 新数据类型;` 或`ALTER TABLE 表名 MODIFY 原列名 新数据类型`
   - 删除列

4. **D**（Delete）：删除

   - `DROP TABLE 表名`
   
     `DROP TABLE IF EXISTS 表名`

##### DML：增删改表中数据

###### 添加数据

语法：`INSERT INTO 表名(列名1,列名2,...,列名n) VALUES(值1,值2,...,值n);`

注意事项：

1. 列名与值一一对应。
2. 如果表名后不定义列名，则默认给所有列添加值
3. 除了数字类型，其他数据类型均需要用单引号（或双引号）括起来

###### 删除数据

语法：`DELETE FROM 表名 [WHERE 条件];`

注意事项:

如果不添加条件，则删除**所有**记录！

不推荐使用`DELETE FROM 表名;`来删除表中所有记录，推荐使用`TRUNCATE TABLE 表名;`

> `TRUNCATE TABLE 表名;`与`DROP TABLE 表名;`以及`DELETE FROM 表名;`区别：
>
> `TRUNCATE TABLE 表名`直接删除所有记录再创建相同表
>
> `DROP TABLE 表名`直接删除表
>
> `DELETE FROM 表名`重复执行`DELETE`以删除表中每一条记录，但不删除表结构。

###### 修改数据

语法：`UPDATE 表名 SET 列1=值1,列2=值2,...,列n=值n [WHERE 条件]`

注意事项：如果不添加条件，则修改**所有**记录！

##### DQL：查询表中记录

###### 语法

```mysql
SELECT 
	字段列表 
FROM 
	表名列表 
WHERE 
	条件列表 
GROUP BY 
	分组字段 
HAVING 
	分组之后的条件 
ORDER 
	排序 
LIMIT 
	分页限定;
```

###### 基础查询

1. 多字段查询`SELECT 列名1,列名2,...,列名n FROM 表名;`

   如果是查询所有字段，可以用`SELECT * FROM 表名;`，但这样写的可读性不高。

2. 去除重复`DISTINCT`

3. 计算列`SELECT 四则运算式 FROM 表名;`

   ```mysql
   SELECT math+english FROM stu;
   ```

   值得注意的是，如果有`NULL`参与，则计算结果也为`NULL`，解决方法：使用函数`IFNULL(字段,替换值)`

   ```mysql
   SELECT math+IFNULL(english,0) FROM stu;
   ```

4. 起别名`AS`（该关键字可省略）

   ```mysql
   SELECT math 数学,english 英语,math+IFNULL(english,0) AS 总分 FROM stu;
   ```

###### 条件查询

`WHERE`子句后跟条件

运算符：

- `>, <, >=, <=, =, <>`

  `<>`在 SQL 中表示不等于，在 MySQL 中也可以用`!=`表示不等于，但请注意，**没有**`==`！

- `BETWEEN...AND`

- `IN(集合)`

- `LIKE`模糊查询

  - 占位符：`_`单个字符，`%`多个任意字符

    eg，查询`name`字段包含'马'的人：`SELECT name FROM stu WHERE name LIKE '%马%'`

- `IS NULL`

  NULL 值不可以做大小比较，只能用`IS`或`IS NOT`来判断

- `AND 或 &&`

  在 SQL 中建议使用前者，后者不通用

- `OR 或 ||`

- `NOT 或 !`

###### 排序查询

语法：`ORDER BY 排序字段1 排序方式1, 排序字段2 排序方式2...;`

默认排序方式为`ASC`即升序，而`DESC`为降序，当且仅当第一条件无法完全判断时才执行后续条件

```mysql
SELECT id,name,score FROM stu ORDER BY score desc,id; -- 按照 score 降序排列，相同成绩则按 id 升序排列
```

###### 聚合函数

将—列数据作为一个整体，进行纵向的计算。

- COUNT：计算个数
- MAX：计算最大值
- MIN：计算最小值
- SUM：求和
- AVG：求平均值

```mysql
SELECT COUNT(id) FROM stu; 
```

注意，所有聚合函数的计算会排除 NULL 值，解决方案：

- 选择不包含 NULL 值的列进行计算
- 使用`IFNULL`函数

###### 分组查询

语法：`GROUP BY 分组字段 HAVING 分组后查询条件`

注意事项：

1. 分组之后查询的字段：分组字段、聚合函数
2. `WHERE`和`HAVING`的区别：
   1. `WHERE`在分组之前进行限定，如果不满足条件，则不参与分组。`HAVING`在分组之后进行限定，如果不满足结果，则不会被查询出来
   2. `WHERE`后不可跟聚合函数，`HAVING`可以进行聚合函数的判断。

```mysql
SELECT gender, AVG(score) FROM stu WHERE score >= 70 GROUP BY gender HAVING COUNT(id) > 2; 
-- 按 gender 分组，查询 score 平均数且 score 低于 70 的参与，分组之后，人数要大于二：
```

###### 分页查询

语法：`LIMIT 开始的索引，每页显示的条数`

公式：`$$开始的索引 = (当前的页码 - 1) * 每页显示的条数$$`

eg，从0开始，显示3条 `SELECT * FROM stu LIMIT 0,3`

注意，`LIMIT`是 MySQL 的方言，对于其他 SQL 可能不适用！

#### 约束

概念：对表中的数据进行限定，保证数据的正确性、有效性和完整性。

分类：

- 主键约束：primary key
- 非空约束：not null
- 唯一约束：unique
- 外键约束：foreign key

##### 非空约束：not null

1. 创建表时添加非空约束：

   ```mysql
   CREATE TABLE test(
       id INT,
       name VARCHAR(20) NOT NULL -- name非空
   );
   ```

2. 创建表后添加非空约束：

   ```mysql
   ALTER TABLE test MODIFY name VARCHAR(20) NOT NULL;
   ```

3. 删除非空约束：

   ```mysql
   ALTER TABLE test MODIFY name VARCHAR(20);
   ```

##### 唯一约束：unique

1. 创建表时添加唯一约束：

   ```mysql
   CREATE TABLE test(
       id INT,
       phone_number VARCHAR(20) UNIQUE -- phone_number唯一
   );
   ```

2. 创建表后添加唯一约束：

   ```mysql
   ALTER TABLE test MODIFY name VARCHAR(20) UNIQUE;
   ```

3. 删除唯一约束：

   ```mysql
   ALTER TABLE test DROP INDEX phone_number;
   ```

##### 主键约束：primary key

- 非空且唯一

- 主键是表中记录的唯一标识

  > 比如身份证号

1. 创建表时添加主键：

   ```mysql
   CREATE TABLE test(
       id INT PRIMARY KEY, -- id主键
       name VARCHAR(20)
   );
   ```

2. 创建表后添加主键：

   ```mysql
   ALTER TABLE test MODIFY id INT PRIMARY KEY;
   ```

3. 删除主键：

   ```mysql
   ALTER TABLE test DROP PRIMARY KEY; -- 删除主键
   ALTER TABLE test MODIFY id INT; -- 删除非空约束
   ```

4. 自动增长

   概念：如果某一列是数值类型的，使用 auto_increment 可以来完成值的自动增长

   1. 创建表时添加主键约束，并完成主键自增长：

      ```mysql
      CREATE TABLE test(
          id INT PRIMARY KEY AUTO_INCREMENT, -- id主键
          name VARCHAR(20)
      );
      ```

      之后再添加记录时可以省略主键，其值为上一条记录的主键值加一

      ```mysql
      insert into test(name) values('a');
      ```

   2. 添加自动增长

      ```mysql
      ALTER TABLE test MODIFY id INT AUTO_INCREMENT;
      ```

   3. 删除自动增长

      ```mysql
      ALTER TABLE test MODIFY id INT;
      ```

##### 外键约束：foreign key

> 如果公共关键字在一个关系中是主关键字，那么这个公共关键字被称为另一个关系的外键。由此可见，外键表示了两个关系之间的相关联系。以另一个关系的外键作主关键字的表被称为主表，具有此外键的表被称为主表的从表。

1. 语法：

   ```mysql
   CREATE TABLE 表名(
       ...
       外键列
       CONSTRAINT 外键名称 FOREIGN KEY (外键列名称) REFERENCES 主表名称(主表列名称)
   );
   ```

   eg，

   ```mysql
   CREATE TABLE department(
       id INT PRIMARY KEY AUTO_INCREMENT,
       dep_name VARCHAR(20),
       dep_location VARCHAR(20)
   ); -- 主表
   CREATE TABLE employee(
       id INT PRIMARY KEY AUTO_INCREMENT,
       name VARCHAR(20),
       age INT,
       dep_id INT, -- 外键对应主表的主键
       CONSTRAINT emp_dept_fk FOREIGN KEY (dep_id) REFERENCES department(id)
   ); -- 从表
   ```

2. 创建表后添加外键

   ```mysql
   ALTER TABLE employee ADD CONSTRAINT emp_dept_fk FOREIGN KEY (dep_id) REFERENCES department(id);
   ```

3. 删除外键

   ```mysql
   ALTER TABLE employee DROP FOREIGN KEY emp_dept_fk;
   ```

4. 级联操作

   > 级联，指当主动方对象执行操作时，被关联对象（被动方）是否同步执行同一操作。

   分类：

   - 级联更新
   - 级联删除

   添加外键，设置级联更新、级联删除

   ```mysql
   ALTER TABLE employee ADD CONSTRAINT emp_dept_fk FOREIGN KEY (dep_id) REFERENCES department(id) ON UPDATE CASCADE ON DELETE CASCADE;
   ```

