---
title:       "redis概念学习"
subtitle:    "快速回顾redis知识"
date:        2024-08-29T02:00:00
author:     "LBT"
tags:        ["redis", "concept"]
categories:  ["Tech" ]
---
## 1、数据结构
redis的诞生也有较久的时间，国内已有经典入门书籍[《redis设计与实现》](https://redisbook.readthedocs.io/en/latest/)，本篇章中的图片来自相应网站
> 学习redis的设计理念可以从[《redis设计与实现》](https://redisbook.readthedocs.io/en/latest/)入手，配合[redis3.0源码](https://github.com/huangzworks/redis-3.0-annotated)使用

本书从3个方面全面解析redis初期的源码：数据结构与对象、单机数据库的实现、多机数据库的实现和独立功能的实现四部分，开发基本只需要，多机部分和独立功能部分完全可跳过
### 1.1 对象（理解）
> redis的数据设计为 **对象 -> 数据结构** 两层，用户直接操作 **对象**，redis决定用何种 **数据结构**。

redis的主要数据结构有，简单动态字符串（SDS）、双端链表、字典、压缩列表、整数集合等。\
redis其实没有直接使用这些数据结构，而是围绕这些数据结构创建了一个对象系统，详情看[redis对象](https://redisbook.readthedocs.io/en/latest/datatype/object.html#redisobject-redis)。\
**对象的类型**
| 类型常量 | 对象的名称 | TYPE 命令输出 |
| ---- | --------- | ---------- |
| REDIS_STRING | 字符串对象 | "string" |
| REDIS_LIST | 列表对象 | "list" |
| REDIS_HASH | 哈希对象 | "hash" |
| REDIS_SET | 集合对象 | "set" |
| REDIS_ZSET | 有序对象集合 | "zset" |

redis对象结构如下:
```c
typedef struct redisObject {
    // 类型
    unsigned type:4;
    // 对齐位
    unsigned notused:2;
    // 编码方式
    unsigned encoding:4;
    // LRU 时间（相对于 server.lruclock）
    unsigned lru:22;
    // 引用计数
    int refcount;
    // 指向对象的值
    void *ptr;

} robj;
```
以上encoding属性记录了对象的编码，即使用什么数据结构作为对象的底层实现。
| 类型 | 编码 | 对象 |
| ---- | --------- | ---------- |
| REDIS_STRING | REDIS_ENCODING_INT（long类型整数） | 使用整数值实现的字符串对象 |
| REDIS_STRING | REDIS_ENCODING_EMBSTR（embstr编码的SDS） | 使用embstr编码的SDS实现的字符串对象 |
| REDIS_STRING | REDIS_ENCODING_RAW（SDS） | 使用SDS实现的字符串对象 |
| REDIS_LIST | REDIS_ENCODING_ZIPLIST（压缩列表） | 使用压缩列表实现的列表对象 |
| REDIS_LIST | REDIS_ENCODING_LINKEDLIST（双端链表） | 使用双端链表实现的列表对象 |
| REDIS_HASH | REDIS_ENCODING_ZIPLIST（压缩列表） | 使用压缩列表实现的哈希对象 |
| REDIS_HASH | REDIS_ENCODING_HT（字典） | 使用字典实现的哈希对象 |
| REDIS_SET | REDIS_ENCODING_INTSET（整数集合） | 使用整数集合实现的集合对象 |
| REDIS_SET | REDIS_ENCODING_HT（字典） | 使用字典实现的集合对象 |
| REDIS_ZSET | REDIS_ENCODING_ZIPLIST（压缩列表） | 使用压缩列表实现的有序集合对象 |
| REDIS_ZSET | REDIS_ENCODING_SKIPLIST（跳跃表和字典） | 使用跳表和字典实现的有序集合对象 |

对象和类型的关系如下图，其中每种对象都有多种编码，满足某对象的特定条件时，会触发对象编码的转换，以适应不同场景:
![](/img/redis/redis-object.svg)
掌握到这里差不多了，延伸信息：命令的类型和多态（同一个命令可用于不同类型）、对象共享（通过预分配一些常见的值对象，并在多个数据结构之间共享这些对象）、引用计数和对象销毁（没错c都是自己实现的回收）

### 1.2 简单动态字符串（SDS）
#### 1.2.1 用途
- 实现字符串对象（StringObject）；
- 在 Redis 程序内部用作 char* 类型的替代品；
#### 1.2.2 操作
```
# 以下命令创建了一个新的数据库键值对， 这个键值对的键和值都是字符串对象， 它们都包含一个 sds 值
redis> SET book "Mastering C++ in 21 days"
OK

redis> GET book
"Mastering C++ in 21 days"
```
```
# 以下命令创建了另一个键值对， 它的键是字符串对象， 而值则是一个集合对象
redis> SADD nosql "Redis" "MongoDB" "Neo4j"
(integer) 3

redis> SMEMBERS nosql
1) "Neo4j"
2) "Redis"
3) "MongoDB"
```
#### 1.2.3 深入
定义如下：
```c
typedef char *sds;

struct sdshdr {
    // buf 已占用长度
    int len;
    // buf 剩余可用长度
    int free;
    // 实际保存字符串数据的地方
    char buf[];
};
```
以上结构有几点优势：
- 1、直接获取长度 
- 2、不会造成缓冲区溢出，API安全
- 3、修改字符串长度N次**最多**只需要N次内存重新分配
- 4、可以保存文本或者二进制数据
- 5、可以用部分<string.h>库中的函数

### 1.3 双端链表
#### 1.3.1 用途
- 作为 Redis 列表类型的底层实现之一；
- 作为通用数据结构，被其他功能模块所使用；
#### 1.3.2 操作
```
redis> RPUSH brands Apple Microsoft Google
(integer) 3

redis> LPOP brands
"Apple"

redis> LLEN brands
(integer) 2

redis> LRANGE brands 0 -1
1) "Microsoft"
2) "Google"
```
#### 1.3.3 深入
> Redis 列表使用两种数据结构作为底层实现：  
> 1、双端链表  
> 2、压缩列表  
> 因为双端链表占用的内存比压缩列表要多， 所以当创建新的列表键时， 列表会优先考虑使用压缩列表作为底层实现， 并且在有需要的时候， 才从压缩列表实现转换到双端链表实现。

结构定义：
```c
// 双端链表节点
typedef struct listNode {
    // 前驱节点
    struct listNode *prev;
    // 后继节点
    struct listNode *next;
    // 值
    void *value;
} listNode;
// 双端链表本身
typedef struct list {
    // 表头指针
    listNode *head;
    // 表尾指针
    listNode *tail;
    // 节点数量
    unsigned long len;
    // 复制函数
    void *(*dup)(void *ptr);
    // 释放函数
    void (*free)(void *ptr);
    // 比对函数
    int (*match)(void *ptr, void *key);
} list;
```
![](/img/redis/list.svg)
双端链表本身的迭代器listIter这里不展示了，包括下一节点和当前方向。

### 1.4 字典
#### 1.4.1 用途
- 实现数据库键空间（key space）
- 用作 Hash 类型键的底层实现之一
redis中的数据库操作都依赖于字典类型，如**DBSIZE、FLUSHDB、RANDOMKEY**等
#### 1.4.2 操作
```
redis> HSET book name "The design and implementation of Redis"
(integer) 1

redis> HSET book type "source code analysis"
(integer) 1

redis> HSET book release-date "2013.3.8"
(integer) 1

redis> HGETALL book
1) "name"
2) "The design and implementation of Redis"
3) "type"
4) "source code analysis"
5) "release-date"
6) "2013.3.8"
```
#### 1.4.3 深入
> Redis 的 Hash 类型键使用以下两种数据结构作为底层实现:  
>   1、字典  
>   2、压缩列表  
> 因为压缩列表比字典更节省内存， 所以程序在创建新 Hash 键时， 默认使用压缩列表作为底层实现， 当有需要时， 程序才会将底层实现从压缩列表转换到字典。

结构如下：
```c
/*
 * 字典
 *
 * 每个字典使用两个哈希表，用于实现渐进式 rehash
 */
typedef struct dict {
    // 特定于类型的处理函数
    dictType *type;
    // 类型处理函数的私有数据
    void *privdata;
    // 哈希表（2 个）
    dictht ht[2];
    // 记录 rehash 进度的标志，值为 -1 表示 rehash 未进行
    int rehashidx;
    // 当前正在运作的安全迭代器数量
    int iterators;
} dict;
/*
 * 哈希表
 */
typedef struct dictht {
    // 哈希表节点指针数组（俗称桶，bucket）
    dictEntry **table;
    // 指针数组的大小
    unsigned long size;
    // 指针数组的长度掩码，用于计算索引值
    unsigned long sizemask;
    // 哈希表现有的节点数量
    unsigned long used;
} dictht;
/*
 * 哈希表节点
 */
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;
    // 链往后继节点
    struct dictEntry *next;
} dictEntry;
```
![](/img/redis/dict.svg)
redis的hash使用桶+链表实现，更多细节看[redis的哈希解读](https://redisbook.readthedocs.io/en/latest/internal-datastruct/dict.html)

### 1.5 跳跃表
#### 1.5.1 用途
和字典、链表或者字符串这几种在 Redis 中大量使用的数据结构不同， 跳跃表在 Redis 的唯一作用， 就是实现有序集数据类型。\
跳跃表将指向有序集的 score 值和 member 域的指针作为元素， 并以 score 值为索引， 对有序集元素进行排序。
#### 1.5.2 操作
```
redis> ZADD s 6 x 10 y 15 z
(integer) 3

redis> ZRANGE s 0 -1 WITHSCORES
1) "x"
2) "6"
3) "y"
4) "10"
5) "z"
6) "15"
```
#### 1.5.3 深入
结构如下：
```c
// 跳跃表定义
typedef struct zskiplist {
    // 头节点，尾节点
    struct zskiplistNode *header, *tail;
    // 节点数量
    unsigned long length;
    // 目前表内节点的最大层数
    int level;
} zskiplist;
// 跳跃表节点
typedef struct zskiplistNode {
    // member 对象
    robj *obj;
    // 分值
    double score;
    // 后退指针
    struct zskiplistNode *backward;
    // 层
    struct zskiplistLevel {
        // 前进指针
        struct zskiplistNode *forward;
        // 这个层跨越的节点数量
        unsigned int span;
    } level[];
} zskiplistNode;
```
![](/img/redis/skiplist.svg)
跳跃表较复杂有趣，但是不需要太了解，详情看[redis跳跃表细节](https://redisbook.readthedocs.io/en/latest/internal-datastruct/skiplist.html)

### 1.6 整数集合
Intset 是集合键的底层实现之一，如果一个集合：
- 只保存着整数元素；
- 元素的数量不多；
那么 Redis 就会使用 intset 来保存集合元素。\

结构定义：
```c
typedef struct intset {
    // 保存元素所使用的类型的长度，三个类型INTSET_ENC_INT16、INTSET_ENC_INT32、INTSET_ENC_INT64 (sizeof(int64_t))
    uint32_t encoding;
    // 元素个数
    uint32_t length;
    // 保存元素的数组
    int8_t contents[];
} intset;
```
contents 数组是实际保存元素的地方，数组中的元素有以下两个特性：
- 元素不重复；
- 元素在数组中由小到大排列；

在编码较小的集合中插入大数为升级，具体升级策略查看[redis的整数集合详解](https://redisbook.readthedocs.io/en/latest/compress-datastruct/intset.html)

### 1.7 压缩列表
> 因为 ziplist 节约内存的性质， 哈希键、列表键和有序集合键初始化的底层实现皆采用 ziplist。

下图展示了一个 ziplist 的典型分布结构：
```
area        |<---- ziplist header ---->|<----------- entries ------------->|<-end->|

size          4 bytes  4 bytes  2 bytes    ?        ?        ?        ?     1 byte
            +---------+--------+-------+--------+--------+--------+--------+-------+
component   | zlbytes | zltail | zllen | entry1 | entry2 |  ...   | entryN | zlend |
            +---------+--------+-------+--------+--------+--------+--------+-------+
                                       ^                          ^        ^
address                                |                          |        |
                                ZIPLIST_ENTRY_HEAD                |   ZIPLIST_ENTRY_END
                                                                  |
                                                         ZIPLIST_ENTRY_TAIL
```
一个 ziplist 可以包含多个节点，每个节点可以划分为以下几个部分：
```
area        |<------------------- entry -------------------->|

            +------------------+----------+--------+---------+
component   | pre_entry_length | encoding | length | content |
            +------------------+----------+--------+---------+
```
具体查看[redis压缩列表详解](https://redisbook.readthedocs.io/en/latest/compress-datastruct/ziplist.html)

## 2、redis单机、多机设定
### 2.1 RDB
RDB是redis的一种持久化机制
- rdbSave 会将数据库数据保存到 RDB 文件，并在保存完成之前阻塞调用者。
- SAVE 命令直接调用 rdbSave ，阻塞 Redis 主进程； BGSAVE 用子进程调用 rdbSave ，主进程仍可继续处理命令请求。
- SAVE 执行期间， AOF 写入可以在后台线程进行， BGREWRITEAOF 可以在子进程进行，所以这三种操作可以同时进行。
- 为了避免产生竞争条件， BGSAVE 执行时， SAVE 命令不能执行。
- 为了避免性能问题， BGSAVE 和 BGREWRITEAOF 不能同时执行。
- 调用 rdbLoad 函数载入 RDB 文件时，不能进行任何和数据库相关的操作，不过订阅与发布方面的命令可以正常执行，因为它们和数据库不相关联。
RDB 文件的组织方式如下：
```
+-------+-------------+-----------+-----------------+-----+-----------+
| REDIS | RDB-VERSION | SELECT-DB | KEY-VALUE-PAIRS | EOF | CHECK-SUM |
+-------+-------------+-----------+-----------------+-----+-----------+

                      |<-------- DB-DATA ---------->|
```
键值对在 RDB 文件中的组织方式如下：
```
+----------------------+---------------+-----+-------+
| OPTIONAL-EXPIRE-TIME | TYPE-OF-VALUE | KEY | VALUE |
+----------------------+---------------+-----+-------+
```
RDB 文件使用不同的格式来保存不同类型的值。
细节看[RDB详解](https://redisbook.readthedocs.io/en/latest/internal/rdb.html)

### 2.2 AOF
AOF是redis的一种持久化机制
- AOF 文件通过保存所有修改数据库的命令来记录数据库的状态。
- AOF 文件中的所有命令都以 Redis 通讯协议的格式保存。
- 不同的 AOF 保存模式对数据的安全性、以及 Redis 的性能有很大的影响。
- AOF 重写的目的是用更小的体积来保存数据库状态，整个重写过程基本上不影响 Redis 主进程处理命令请求。
- AOF 重写是一个有歧义的名字，实际的重写工作是针对数据库的当前值来进行的，程序既不读写、也不使用原有的 AOF 文件。
- AOF 可以由用户手动触发，也可以由服务器自动触发。
细节看[AOF详解](https://redisbook.readthedocs.io/en/latest/internal/aof.html)

## 3、附录
**常用命令**
| 命令                                       | 说明                                       |
| ------------------------------------------ | ------------------------------------------ |
| set                                        | 设置一个key/value                          |
| get                                        | 根据key获得对应的value                     |
| mset                                       | 一次设置多个key value                      |
| mget                                       | 一次获得多个key的value                     |
| getset                                     | 获得原始key的值，同时设置新值              |
| strlen                                     | 获得对应key存储value的长度                 |
| append                                     | 为对应key的value追加内容                   |
| getrange 索引0开始                         | 截取value的内容                            |
| setex                                      | 设置一个key存活的有效期（秒）              |
| psetex                                     | 设置一个key存活的有效期（毫秒）            |
| setnx                                      | 存在不做任何操作,不存在添加                |
| msetnx原子操作(只要有一个存在不做任何操作) | 可以同时设置多个key,只要有一个存在都不保存 |
| decr                                       | 进行数值类型的-1操作                       |
| decrby                                     | 根据提供的数据进行减法操作                 |
| Incr                                       | 进行数值类型的+1操作                       |
| incrby                                     | 根据提供的数据进行加法操作                 |
| Incrbyfloat                                | 根据提供的数据加入浮点数                   |

