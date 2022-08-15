# Redis学习(一)


<!--more-->

## Redis学习(一)——Redis入门

### Redis概述

> Redis是什么？来自[CRUG网站 (redis.cn)](http://www.redis.cn/)
>
> Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作==数据库==、==缓存==和==消息中间件MQ==。它支持多种类型的数据结构，如字符串（strings），散列（hashes），列表（lists），集合（sets），有序集合（sortedsets）与范围查询，bitmaps，hyperloglogs和地理空间（geospatial）索引半径查询。Redis 内置了复制（replication），LUA 脚本（Luascripting），LRU驱动事件（LRUeviction），事务（transactions）和不同级别的磁盘持久化（persistence），并通过 Redis 哨兵（Sentinel）和自动分区（Cluster）提供高可用性（highavailability）。

- Redis（**Re**mote **Di**ctionary **S**erver），即远程字典服务。
- 是一个免费开源的使用 ANSl **C语言**编写、支持网络、可基于内存亦可持久化的日志型、**Key-value** 数据库，并提供多种语言的 API。
- 是当下最热门的 NoSQL 技术之一。

> Redis能干什么？

1. 内存存储、持久化，内存中是断电即失、所以说持久化很重要（rdb、aof）效率高，可以用于高速缓存
2. 发布订阅系统
3. 地图信息分析
4. 计时器、计数器（浏览量）

> 特效

1. 多样的数据类型
2. 持久化
3. 集群事务

### Redis安装

我这里采用最为便捷的方法——**docker** 容器启动。

我们可以前往 Redis 中文官网下载压缩包，解压取得配置文件：[redis 6.0.6 下载 -- Redis中国用户组（CRUG）](http://www.redis.cn/download.html)

然后你需要修改配置文件的部分内容：

1.  `bind 127.0.0.1` 注释掉这部分，使 redis 可以从外部访问 
2. `daemonize yes`守护线程的方式启动 
3. `requirepass 密码`给 redis 设置密码 
4. `appendonly yes`redis持久化，默认是 no
5. `tcp-keepalive 300` 防止出现远程主机强迫关闭了一个现有的连接的错误，默认是 300

依然是给出 shell 和 docker-compose 两种启动，注意修改配置：

1. `shell`

   ```shell
   docker run -itd --name redis -h redis -p 6379:6379 -v /root/redis/redis.conf:/etc/redis/redis.conf  -v /root/redis/data:/data redis redis-server /etc/redis/redis.conf --appendonly yes
   ```

2. `docker-compose.yml`

   ```yaml
   version: "3"
   services:
   
     db:
       image: redis
       container_name: redis
       stdin_open: true
       tty: true
       ports:
         - "6379:6379"
       volumes:
         - /root/redis/redis.conf:/etc/redis/redis.conf
         - /root/redis/data:/data
       command: redis-server /etc/redis/redis.conf --appendonly yes
       restart: always
       hostname: redis
   ```

### 基础知识

执行以下命令进入 Redis 命令行工具：

```shell
docker exec -it redis bash
cd /usr/local/bin
redis-cli
```

Redis 默认有 16 个数据库（0-15），默认使用数据库 0 即第一个数据库，`select`切换数据库。

`flushdb`清空当前数据库，`flushall` 清空所有数据库。

#### Redis6.0多线程

Redis 6.0 开始支持多线程，Redis 分主线程和 IO 线程，IO 线程只用于读取客户端命令和发送回复数据给客户端，客户端命令依旧是由**主线程**来执行。

> 为什么要使用多线程呢？

Redis 将所有数据放在内存中，内存的响应速度在纳秒级别，对于小数据包，Redis服务器可以处理 80,000到 100,000 QPS，所以大多数情况下，单线程的 Redis 足够了。并且多线程的 CPU 上下文会切换是一个耗时的操作，对于内存系统来说，如果没有上下文切换效率就是最高的！多次读写都是在一个 CPU 上的，在内存情况下，这个就是最佳的方案。

但是 Redis 从自身的角度来说，因为读写网络的 read/write 系统调用占用了 Redis 执行期间大部分 CPU 时间，瓶颈主要在于网络 IO 消耗，网络 IO 主要延时由服务器响应延时+带宽限制+网络延时+跳转路由延时组成，其一般在毫秒级别。

**多线程 Redis 主要为了利用多核 CPU，目前主线程只能利用一个核，多线程任务可以分摊 Redis 同步 IO 读写的负荷。**

**Redis6.0多线程开启方法**
 要开启 Redis 的IO线程功能，需要修改 redis.conf 配置文件：

```conf
io-threads-do-reads yes     # 开启IO线程
io-threads 6                # 设置IO线程数
```

线程数的设置，官方建议：4 核的设置 2 或 3 个线程，8 核的建议设置 6 个线程，线程数一定要小于机器核数，线程数并不是越大越好，官方认为超过 8 个基本没什么意义了。

#### 五大数据类型

##### Redis-Key

所有命令都可以在官网查询：[Redis命令中心（Redis commands） -- Redis中国用户组（CRUG）](http://www.redis.cn/commands.html#generic)

```bash
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> SET name su
OK
127.0.0.1:6379[1]> MOVE name 0  # 移动name到数据库0
(integer) 1
127.0.0.1:6379[1]> SELECT 0
OK
127.0.0.1:6379> TYPE name  # 查看name的类型
string
127.0.0.1:6379> EXPIRE name 10  # 设置name过期时间10秒
(integer) 1
127.0.0.1:6379> TTL name  # 查看name剩余时间
(integer) 7
127.0.0.1:6379> TTL name
(integer) -2  # -1表示永久，-2表示过期
127.0.0.1:6379> FLUSHDB  # 清空当前数据库
OK
```

##### String（字符串）

```bash
127.0.0.1:6379> SET name su  # 设置值set key value
OK
127.0.0.1:6379> GET name  # 获得值
"su"
127.0.0.1:6379> KEYS *  # 获得所有的key
1) "name"
127.0.0.1:6379> EXISTS name  # 判断某一个key是否存在
(integer) 1
127.0.0.1:6379> APPEND name "kun"  # 追加字符串，如果当前key不存在，就相当于set key value
(integer) 5
127.0.0.1:6379> STRLEN name  # 获取字符串的长度
(integer) 5
########################################################################################################
# 自增、自减
127.0.0.1:6379> SET views 0
OK
127.0.0.1:6379> INCR views  # 自增1
(integer) 1
127.0.0.1:6379> DECR views  # 自减1
(integer) 0
127.0.0.1:6379> INCRBY views 10  # 设置步长，指定增量
(integer) 10
127.0.0.1:6379> FLUSHDB
OK
########################################################################################################
# 字符串范围
127.0.0.1:6379> SET name "hello world"
OK
127.0.0.1:6379> GET name
"hello world"
127.0.0.1:6379> GETRANGE name 0 4  # 截取字符串[0,4]，正索引，从第一个字符开始到第五个字符
"hello"
127.0.0.1:6379> GETRANGE name 0 -1  # -1为倒索引，从第一个字符到最后一个字符
"hello world"
127.0.0.1:6379> SETRANGE name 6 sukun  # 替换指定位置开始的字符串
(integer) 11
127.0.0.1:6379> GET name
"hello sukun"
127.0.0.1:6379> FLUSHDB
OK
########################################################################################################
# setex (set with expire)
# setnx (set if not exists)
127.0.0.1:6379> SETEX name 10 su  # 设置name值su，10秒后过期
OK
127.0.0.1:6379> TTL name
(integer) 7
127.0.0.1:6379> TTL name
(integer) -2
127.0.0.1:6379> SETNX name sukun  # 如果name不存在，则设置name值sukun
(integer) 1  # 成功返回1
127.0.0.1:6379> SETNX name kun
(integer) 0  # 失败返回0
127.0.0.1:6379> FLUSHDB
OK
########################################################################################################
127.0.0.1:6379> MSET k1 v1 k2 v2 k3 v3  # 同时设置多个值
OK
127.0.0.1:6379> KEYS *
1) "k3"
2) "k2"
3) "k1"
127.0.0.1:6379> MGET k1 k2 k3  # 同时获取多个值
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> MSETNX k1 v1 k4 v4  # 原子性操作，要么一起成功，要么一起失败
(integer) 0
127.0.0.1:6379> GET k4
(nil)
127.0.0.1:6379> SET user:1 {name:sukun,age:18}  # 设置一个user:1对象，值为json字符串
OK
127.0.0.1:6379> GET user:1
"{name:sukun,age:18}"
127.0.0.1:6379> MSET user:1:name sukun user:1:age 18  # 利用:区分层级，实现user:{id}:{filed}
OK
127.0.0.1:6379> MGET user:1:name user:1:age
1) "sukun"
2) "18"
########################################################################################################
# getset 先get再set
127.0.0.1:6379> GETSET db redis  # 如果不存在值，返回nil
(nil)
127.0.0.1:6379> GET db
"redis"
127.0.0.1:6379> GETSET db mongodb  # 如果存在值，获取原来的值，再设置新值
"redis"
127.0.0.1:6379> GET db
"mongodb"
```

String 使用场景：value 除了字符串还可以是数字

- 计数器
- 统计多单位数量
- 对象缓存存储

##### List

基本数据类型：列表，但在 Redis 也可以为队列、阻塞队列、栈。

所有 List 的命令都是 L 开头：

```bash
127.0.0.1:6379> LPUSH list 1  # 将一个值或多个值，从左边插入列表
(integer) 1
127.0.0.1:6379> LPUSH list 2 3
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1  # list range，根据区间获取list中的值
1) "3"
2) "2"
3) "1"
127.0.0.1:6379> LPOP list  # 弹出list左边的第一个值
"3"
127.0.0.1:6379> RPOP list 2  # 弹出list右边的指定数量的值
1) "1"
2) "2"
127.0.0.1:6379> RPUSH list 1 2 3  # 将一个值或多个值，从右边插入列表
(integer) 3
127.0.0.1:6379> LINDEX list 0  # list index，通过下标获得list中的值
"1"
127.0.0.1:6379> LLEN list  # 返回list长度
(integer) 3
127.0.0.1:6379> LPUSH list 1 2 3
(integer) 6
127.0.0.1:6379> LRANGE list 0 -1
1) "3"
2) "2"
3) "1"
4) "1"
5) "2"
6) "3"
127.0.0.1:6379> LREM list 2 1  # 移除list中指定个数的value，精确匹配
(integer) 2
127.0.0.1:6379> LRANGE list 0 -1
1) "3"
2) "2"
3) "2"
4) "3"
127.0.0.1:6379> LTRIM list 0 1  # 修剪(trim)，只保留指定区间内的元素，区间外的则删除。
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "3"
2) "2"
127.0.0.1:6379> LPUSH list1 1 2 3
(integer) 3
127.0.0.1:6379> LRANGE list1 0 -1
1) "3"
2) "2"
3) "1"
127.0.0.1:6379> RPOPLPUSH list1 list  # 从source list右边弹出，再压入destination list的左边。
"1"
127.0.0.1:6379> LRANGE list1 0 -1
1) "3"
2) "2"
127.0.0.1:6379> LRANGE list 0 -1
1) "1"
2) "3"
3) "2"
127.0.0.1:6379> LSET list 0 value  # 将列表中指定下标值更新为新值
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "value"
2) "2"
3) "1"
127.0.0.1:6379> RPUSH list2 "hello" "world"
(integer) 2
127.0.0.1:6379> LRANGE list2 0 -1
1) "hello"
2) "world"
127.0.0.1:6379> LINSERT list2 before "world" "other"  # 将值插入到指定元素前或后
(integer) 3
127.0.0.1:6379> LINSERT list2 after "other" "new"
(integer) 4
127.0.0.1:6379> lrange list2 0 -1
1) "hello"
2) "other"
3) "new"
4) "world"
127.0.0.1:6379> FLUSHDB
OK
```

> 小结

* list 本质上是链表，before Node、after Node、head（left）、tail（right）都能进行插入
* 如果 key 不存在，则创建新链表
* 如果 key 存在，新增内容
* 如果移除了所有值，空链表也将不存在
* 在链表双端插入或改动值效率最高。
* 显然 list 链表可实现栈和队列，譬如消息队列

##### Set（集合）

```bash
127.0.0.1:6379> SADD myset "hello" "world" "sukun"  # 向set中添加元素（无序不重复）
(integer) 3
127.0.0.1:6379> SMEMBERS myset  # 查看set所有值
1) "world"
2) "hello"
3) "sukun"
127.0.0.1:6379> SISMEMBER myset hello  # 判断值是否在set中
(integer) 1
127.0.0.1:6379> SREM myset world  # 移除set中指定值
(integer) 1
127.0.0.1:6379> SCARD myset  # 查看set中元素数量
(integer) 2
127.0.0.1:6379> SMEMBERS myset
1) "hello"
2) "sukun"
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> SADD myset 1 2 3 4 5
(integer) 5
127.0.0.1:6379> SRANDMEMBER myset  #随机抽选指定个数的元素，默认一个
"5"
127.0.0.1:6379> SRANDMEMBER myset 3
1) "4"
2) "1"
3) "5"
127.0.0.1:6379> SPOP myset  # 随机删除元素
"5"
127.0.0.1:6379> SPOP myset
"3"
127.0.0.1:6379> SPOP myset
"2"
127.0.0.1:6379> SMEMBERS myset
1) "1"
2) "4"
127.0.0.1:6379> SADD set1 1 2 3
(integer) 3
127.0.0.1:6379> SADD set2 one two three
(integer) 3
127.0.0.1:6379> SMOVE set1 set2 1  # 将指定值移动到另一个set中
(integer) 1
127.0.0.1:6379> SMEMBERS set1
1) "2"
2) "3"
127.0.0.1:6379> SMEMBERS set2
1) "three"
2) "1"
3) "two"
4) "one"
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> SADD set1 1 2 3
(integer) 3
127.0.0.1:6379> SADD set2 2 3 4
(integer) 3
127.0.0.1:6379> SDIFF set1 set2  # 差集
1) "1"
127.0.0.1:6379> SINTER set1 set2  # 交集
1) "2"
2) "3"
127.0.0.1:6379> SUNION set1 set2  # 并集
1) "1"
2) "2"
3) "3"
4) "4"
127.0.0.1:6379> FLUSHDB
OK
```

##### Hash（哈希）

Map 集合，key-map 形式。

```bash
127.0.0.1:6379> HSET myhash one 1 two 2 three 3  # set多个或一个key-value
(integer) 3
127.0.0.1:6379> HGET myhash one  # 获取hash中一个字段值
"1"
127.0.0.1:6379> HMGET myhash one two three  # 获取hash中多个字段值
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> HGETALL myhash  # 获取hash中全部数据
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"
127.0.0.1:6379> HDEL myhash one  # 删除hash中指定key-value键值对
(integer) 1
127.0.0.1:6379> HGET myhash one
(nil)
127.0.0.1:6379> HLEN myhash  # 获取hash中字段数量
(integer) 2
127.0.0.1:6379> HEXISTS myhash one  # 判断hash中指定字段是否存在
(integer) 0
127.0.0.1:6379> HEXISTS myhash two
(integer) 1
127.0.0.1:6379> HKEYS myhash  # 获取hash中所有filed
1) "two"
2) "three"
127.0.0.1:6379> HVALS myhash  # 获取hash中所有value
1) "2"
2) "3"
127.0.0.1:6379> HINCRBY myhash two 4  # 指定增量
(integer) 6
127.0.0.1:6379> HINCRBY myhash two -1
(integer) 5
127.0.0.1:6379> HSETNX myhash two 4  # 如果存在不能set
(integer) 0
127.0.0.1:6379> HSETNX myhash one 1  # 如果不存在则set
(integer) 1
127.0.0.1:6379> FLUSHDB
OK
```

##### Zset（有序集合）

在 set 基础上，增加了一个值：

- `set key value`
- `zadd key score value`

```bash
127.0.0.1:6379> ZADD myset 1 one 2 two 3 three  # 向有序集合添加score-value
(integer) 3
127.0.0.1:6379> ZRANGE myset 0 -1  # 获取全部value，升序
1) "one"
2) "two"
3) "three"
127.0.0.1:6379> ZREVRANGE myset 0 -1  # 降序
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> ZRANGEBYSCORE myset 1 3  # 根据score区间获取value
1) "one"
2) "two"
3) "three"
127.0.0.1:6379> ZRANGEBYSCORE myset -inf +inf WITHSCORES  # 获取负无穷到正无穷score区间内的value-score
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"
127.0.0.1:6379> ZREM myset one  # 删除指定value
(integer) 1
127.0.0.1:6379> ZRANGE myset 0 -1
1) "two"
2) "three"
127.0.0.1:6379> ZCARD myset  # 获取zset中成员数量
(integer) 2
127.0.0.1:6379> ZCOUNT myset 0 3  # 获取指定区间中成员数量
(integer) 2
127.0.0.1:6379> FLUSHDB
OK
```

#### 事务

Redis 事务本质：一组命令的集合。一个事务中的所有命令都会被序列化，在事务执行过程的中，会按照顺序执行。

—次性、顺序性、排他性。

**Redis 事务没有没有隔离级别的概念**

所有的命令在事务中，并没有直接被执行。只有发起执行命令的时候才会执行。

**Redis 单条命令式保存原子性的，但是事务不保证原子性**

Redis 的事务︰

* 开启事务（）
* 命令入队（）
* 执行事务（）

##### 开启事务

```bash
127.0.0.1:6379> MULTI  # 开启事务
OK
# 命令入队
127.0.0.1:6379(TX)> SET k1 v1
QUEUED
127.0.0.1:6379(TX)> SET k2 v2
QUEUED
127.0.0.1:6379(TX)> GET k2
QUEUED
127.0.0.1:6379(TX)> SET k3 v3
QUEUED
127.0.0.1:6379(TX)> EXEC  # 执行事务
1) OK
2) OK
3) "v2"
4) OK
127.0.0.1:6379> FLUSHDB
OK
```

##### 放弃事务

```bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> SET k1 v1
QUEUED
127.0.0.1:6379(TX)> SET k2 v2
QUEUED
127.0.0.1:6379(TX)> DISCARD  # 取消事务
OK
127.0.0.1:6379> GET k1
(nil)  # 命令队列中的命令均不会被执行
```

> 编译异常，事务中所有命令都不会执行。
>
> 运行异常，如果事务队列中存在语法问题，执行命令时其余命令正常执行，错误命令抛出异常。

### Redis.conf解释

完整的配置文件之前已经提供，内有完整注释，请自行查阅！

1. units 单位对大小写不敏感

   ![image-20220704171615673](https://s2.loli.net/2022/07/04/ogbK8r4ihJdtfEQ.png)

2. 配置文件可用`include`包含

   ![image-20220704171853294](https://s2.loli.net/2022/07/04/RV7qcyS9oWrKEun.png)

3. 网络

   ```conf
   bind 127.0.0.1  # 绑定IP
   
   protected-mode yes  # 开启保护模式
   
   port 6379  # 端口设置
   ```

4. 通用

   ```conf
   daemonize yes  # 守护线程的方式启动 
   
   pidfile  # 如果以后台的方式运行，需要指定一个进程文件
   
   loglevel notice  # 日志等级
   
   logfile ''  # 指定日志文件
   
   databases 16  # 数据库数量默认16
   
   always-show-logo yes  # 启动时输出redis的logo
   ```

5. 快照

   持久化，在规定的时间内，执行了多少次操作，则会持久化到文件 .rdb .aof。

   由于 Redis 是内存数据库，如果没有持久化，那么数据断电即失！

   ```conf
   save 900 1  # 如果900s内，如果至少有一个1 key进行了修改，则进行持久化操作
   
   stop-writes-on-bgsave-error no  # 持久化如果出错，仍继续工作
   
   rdbcompression yes  # 是否压缩rdb文件（需要消耗一些cpu资源）
   
   rdbchecksum yes # 保存rdb文件时进行错误校验
   
   dir ./  # rdb文件目录
   ```

6. 安全

   ```shell
   127.0.0.1:6379> config get requirepass  #获取redis的密码
   1) "requirepass"
   2) ""
   127.0.0.1:6379> config set requirepass "123456"  #设置redis的密码
   OK
   127.0.0.1:6379> ping  # 没有权限
   (error) NOAUTH Authentication required.
   127.0.0.1:6379> auth 123456  #使用密码进行登录!
   OK
   ```

7. 限制 CLIENTS

   ```conf
   maxclients 10000  #设置能连接上redis的最大客户端的数量
   
   maxmemory <bytes>  # redis配置最大的内存容量
   
   maxmemory-policy noeviction  # 内存到达上限之后的处理策略
   1、volatile-lru: 只对设置了过期时间的key进行LRU（默认值)
   2、allkeys-lru: 删除1ru算法的key
   3、volatile-random: 随机删除即将过期key
   4、allkeys-random: 随机删除
   5、volatile-ttl: 删除即将过期的
   6、noeviction: 永不过期，返回错误
   ```

8. APPEND ONLY 模式 aof 配置

   ```conf
   appendon1y no  #  默认是不开启aof模式的，默认是使用rdb方式持久化的，在大部分所有的情况下，rdb完全够用
   
   appendfilename "appendon1y.aof"  # 持久化的文件的名字
   
   # appendfsync always  # 每次修改都会sync消耗性能
   
   appendfsync everysec  # 每秒执行一次
   
   # appendfsync no  # 不执行sync，这个时候操作系统自己同步数据，速度最快
   ```

### Redis持久化

Redis 是内存数据库，如果不将内存中的数据库状态保存到磁盘，一旦服务器进程退出，服务器中的数据库状态也会消失。所以 Redis 提供了持久化功能。

#### RDB(Redis DataBase)

![RDB](https://www.freesion.com/images/585/51c3beeb79603340ded6c3ed97239e91.png)

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的 Snapshot 快照，它恢复时是将快照文件直接读到内存里。

Redis 会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何 IO 操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那 RDB 方式要比 AOF 方式更加的高效。RDB 的缺点是最后一次持久化后的数据可能丢失。

默认 RDB，一般情况不需要修改配置。

生产环境中需进行备份。

RDB 保存的文件是 dump.rdb

```conf
dbfilename dump.rdb
```

触发机制：

1. save 规则满足的情况下，触发 rdb 规则
2. 执行 flushall 命令，触发 rdb 规则
3. 退出 redis ，也会产生 rdb 文件

备份就自动生成一个 dump.rdb 文件

只需要将 rdb 文件放在 redis 启动目录，redis 启动的时候会自动检查 dump.rdb 并恢复数据。

优点∶

1. 适合大规模的数据恢复
2. 对数据的完整性要不高

缺点︰

1. 需要一定的时间间隔进程操作！如果 redis 意外宕机了，这个最后一次修改数据就丢失了！
2. fork 进程的时候，会占用一定的内容空间！

#### AOF(Append Only File)

![AOF](https://www.freesion.com/images/885/59363f1f14dca1bca33b4435ca5eaa2d.png)

以日志的形式来记录每个写操作，将 Redis 执行过的所有指令记录下来(读操作不记录），只许追加文件但不可以改写文件，redis 启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

AOF 保存的是 appendonly.aof 文件。

![image-20220704193959142](https://s2.loli.net/2022/07/04/dqkurW2L5JbilO9.png)

```conf
appendon1y no  #  默认是不开启aof模式的，默认是使用rdb方式持久化的，在大部分所有的情况下，rdb完全够用

appendfilename "appendon1y.aof"  # 持久化的文件的名字

# appendfsync always  # 每次修改都会sync消耗性能

appendfsync everysec  # 每秒执行一次

# appendfsync no  # 不执行sync，这个时候操作系统自己同步数据，速度最快
```

重启之后 aof 即可生效

如果 aof 文件错位，redis 无法启动，需要使用`redis-check-aof`来修复。

如果 aof 文件正常，重启 redis 即可恢复数据。

优点：

1. 每一次修改都同步，文件的完整性会更加好
2. 每秒同步一次，可能会丢失—秒的数据
3. 从不同步，效率最高

缺点：

1. 相对于数据文件来说，aof 远远大于 rdb，修复的速度也比 rdb 慢
2. aof 运行效率也要比 rdb 慢，所以 redis 默认的配置就是 rdb 持久化

重写规则说明：

![image-20220705171727344](https://s2.loli.net/2022/07/05/UmMWzqVQgTcbv9r.png)

aof 默认就是文件的无限追加，文件会越来越大。

如果 aof 文件大于 64m，太大了会 fork 一个新的进程来对文件进行重写。

#### 扩展知识

1. RDB 持久化方式能够在指定的时间间隔内对你的数据进行快照存储
2. AOF 持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF 命令以 Redis 协自加保存每次写的操作到文件末尾，Redis 还能对 AOF 文件进行后台重写，使得 AOF 文件的体积不至于过大。
3. 只做缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化
4. 同时开启两种持久化方式
   - 在这种情况下，当 redis 重启的时候会优先载入 AOF 文件来恢复原始的数据，因为在通常情况下 AOF 文件保存的数据集要比RDB 文件保存的数据集要完整。
   - RDB 的数据不实时，同时使用两者时服务器重启也只会找 AOF 文件，建议不要只使用 AOF，因为 RDB 更适合用于备份数据库（AOF 在不断变化不好备份），快速重启，而且不会有 AOF 可能潜在的 Bug，留着作为一个万一的手段。
5. 性能建议
   - 因为 RDB 文件只用作后备用途，建议只在 Slave 上持久化 RDB 文件，而且只要 15 分钟备份一次就够了，只保留`save 900 1`这条规则。
   - 如果 Enable AOF ，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只 load 自己的 AOF 文件就可以了，代价一是带来了持续的 lO ，二是 AOF rewrite 的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少 AOF rewrite 的频率，AOF 重写的基础大小默认值 64M 太小了，可以设到 5G 以上，默认超过原大小 100% 大小重写可以改到适当的数值。
   - 如果不 Enable AOF，仅靠 Master-Slave Repllcation 实现高可用性也可以，能省掉一大笔 IO，也减少了 rewrite 时带来的系统波动。代价是如果 Master/Slave 同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave 中的 RDB 文件，载入较新的那个，微博就是这种架构。

