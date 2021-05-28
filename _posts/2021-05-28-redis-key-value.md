---
title:  "redis常用键值对解析"
author: WangJwi
categories:
- redis
tags:
- redis
- key-value
---

导读：Redis不是简单的键值（Key-Value）存储，它实际上是一个支持不同类型值的数据结构服务器。在传统键值存储中，一般将字符串键与字符串值相关联，而在Redis中，其值不仅限于简单的字符串，还可以容纳更复杂的数据结构。


<br><br>

------

# Redis的常用键值对

## Redis的键值对

Redis不是简单的键值（Key-Value）存储，它实际上是一个支持不同类型值的数据结构服务器。在传统键值存储中，一般将字符串键与字符串值相关联，而在Redis中，该值不仅限于简单的字符串，还可以容纳更复杂的数据结构。



### Redis-Key

Redis的键（Key）是二进制安全的字符串，这意味着可以使用任何二进制序列作为key，从“ foo”之类的字符串到JPEG文件的内容，同时空字符串也是有效的key键。

在Redis的底层中由SDS实现。

设置键的注意点：

- **key使用的字节不宜过长**：太长的key字节，在内存方面不仅是个负担，并且在数据集中查找key时可能需要进行一些代价高昂的密钥比较。
- **key也不宜设置太短**：与“ user：1000：followers”对比，“ u1000flw”写为key毫无意义，前者往往更具可读性。且与键对象本身和值对象使用的空间相比，单单添加key消耗的空间更少。
- **key使用同一种设置类型**：例如，“ object-type：id”是一个好主意，例如“ user：1000”。点或破折号通常用于多字字段，例如“ comment：1234：reply.to”或“ comment：1234：reply-to”中。
- 允许的最大key大小为**512 MB**。

<br><br>

------

###  Redis-Value

Redis的值（Value）支持以下几种类型：

- **Binary-safe strings**：二进制安全字符串。
- **Lists**：根据插入顺序排序的字符串元素的集合。它们基本上是*链表*。
- **Sets**：唯一，未排序的字符串元素的集合。
- **Sorted sets**：类似于集合，但是每个字符串元素在存入时都将于一个浮点数值的分数相关联，元素总是按照它们的分数排序，因此与Sets不同，可以检索一系列元素
- **Hashes**：键值组成的哈希映射，键值都是字符串。
- **Bit arrays** (or simply bitmaps)：可以使用特殊命令像位数组一样处理字符串值：可以设置和清除单个位，计数所有设置为1的位，找到第一个设置或未设置的位，等等。
- **HyperLogLogs**：这是一个概率数据结构，用于估计集合的基数。
- **Streams**：提供抽象日志数据类型的类地图项的仅追加集合。



redis值的内部实现结构如下：

| type类型     | encoding编码              | ptr指向的数据结构               |
| ------------ | ------------------------- | ------------------------------- |
| REDIS_STRING | REDIS_ENCODING_INT        | 整数值的实现的字符串对象        |
| REDIS_STRING | REDIS_ENCODING_EMBSTR     | embstr编码的sds实现的字符串对象 |
| REDIS_STRING | REDIS_ENCODING_RAW        | sds字符串实现的字符串对象       |
| REDIS_LIST   | REDIS_ENCODING_ZIPLIST    | 压缩列表实现                    |
| REDIS_LIST   | REDIS_ENCODING_LINKEDLIST | 链表实现                        |
| REDIS_HASH   | REDIS_ENCODING_ZIPLIST    | 压缩表实现                      |
| REDIS_HASH   | REDIS_ENCODING_HT         | 字典实现                        |
| REDIS_SET    | REDIS_ENCODING_INTSET     | 整数集合实现                    |
| REDIS_SET    | REDIS_ENCODING_HT         | 字典实现                        |
| REDIS_ZSET   | REDIS_ENCODING_ZIPLIST    | 压缩表实现                      |
| REDIS_ZSET   | REDIS_ENCODING_SKIPLIST   | 跳跃表和字典实现                |


<br><br>

------

### Redis值的常用数据类型

**字符串：String**

Redis字符串类型是与Redis键关联的最简单的值类型。它也是Memcached中唯一的数据类型。

值是二进制安全的字符串，意味着redis的string可以包含任何数据。比如jpg图片或者序列化的对象。值最大能存储**512 MB**的数据。



**列表：List**

redis是由双向链表（Linked List）的方式实现，而非数组形式。它能从链表的头尾进行操作。它是根据插入的顺序进行排序的有序列表。

列表的常用案例：记住用户发布到社交网络上的最新更新。频繁查看的日志。





**集合：Set**

set 的内部实现是一个 value永远为null的HashMap，实际就是通过计算hash的方式来快速排重的，这也是set能提供判断一个成员是否在集合内的原因。

可以理解为一堆值不重复的列表，类似数学领域中的集合概念，且Redis也提供了针对集合的求交集、并集、差集等操作。



**有序集合：Sorted Set**

Redis有序集合类似Redis集合，不同的是增加了一个功能，即集合是有序的。一个有序集合的每个成员带有分数，用于进行排序。

Redis有序集合添加、删除和测试的时间复杂度均为O(1)(固定时间，无论里面包含的元素集合的数量)。列表的最大长度为2^32- 1元素(4294967295，超过40亿每个元素的集合)。

Redis sorted set的内部使用HashMap和跳跃表(SkipList)来保证数据的存储和有序，HashMap里放的是成员到score的映射，而跳跃表里存放的是所有的成员，排序依据是HashMap里存的score,使用跳跃表的结构可以获得比较高的查找效率，并且在实现上比较简单。



**哈希表：Hash**

对应Value内部实际就是一个HashMap，实际这里会有2种不同实现，这个Hash的成员比较少时Redis为了节省内存会采用类似一维数组的方式来紧凑存储，而不会采用真正的HashMap结构，对应的value redisObject的encoding为zipmap,当成员数量增大时会自动转成真正的HashMap,此时encoding为ht。

每个 hash 可以存储 232 -1 键值对（40多亿）。

------

参考：[Redis Documentation](https://redis.io/documentation)

[Redis维基百科](https://en.wikipedia.org/wiki/Redis)