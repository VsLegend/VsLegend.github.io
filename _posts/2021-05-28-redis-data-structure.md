---
title:  "redis基础知识和底层数据结构"
author: WangJwi
categories:
- redis
tags:
- redis
- 数据结构
---

Redis（Remote Dictionary Server）是一个在内存中运行的数据结构的存储服务器（an in-memory data structure store）。Redis支持各种抽象数据结构，例如字符串，列表，映射，集合，排序集合，HyperLogLogs，位图，流和空间索引。



# Redis基础知识

## **概念：**

Redis（Remote Dictionary Server）是一个在内存中运行的数据结构的存储服务器（an in-memory data structure store）。Redis支持各种抽象数据结构，例如字符串，列表，映射，集合，排序集合，HyperLogLogs，位图，流和空间索引。



## **用途：**

常被用于分布式、键值对数据库、高速缓存、消息代理等应用。


## redis的优点

1. 内存操作，高性能
2. 单线程执行，天然支持并发




<br><br>

------


# Redis的底层数据结构

## 简单动态字符串：SDS
### 描述

Redis由C语言编写。它将SDS（simple dynamic string）用作默认的字符串表示方式。而C字符串仅用作字符串字面量（string literal），即无须对字符串值进行修改的地方。Redis的字符串类型的键值都是由SDS实现的。此外，SDS还可以被用作缓冲区（buffer），例如AOF缓冲区、输入缓冲区等。




### 定义

SDS遵循C字符串以空字符结尾的惯例，保存空字符的1字节空间不计算在SDS的len属性里。该空字符的添加操作由SDS函数自动完成。这一操作的好处是，可以直接复用C语言的一些字符串操作。

每个sds.h/sdshdr表示一个SDS值，SDS的定义如下：

``` c
struct sdshdr {
  int len; // 记录buf数组中已使用的字节数量，等于SDS所保存字符串的长度
  
  int free; // 记录buf数组中未使用的字节数量
  
  char buf[]; // 字节数组，用于保存字符串
}
```





### 特性

前置知识点：
内存的重新分配操作可能出现的问题

- 缓冲区溢出：增长字符串时，程序需要通过内存分配来来确定是否需要对底层数组的空间进行扩充，以便存放增长后的字符串的值。如果忘了这一步，那么保存字符串的数组就会发生越界行为，占用未分配给它的内存区域，从而导致其他数据被意外篡改，这就是缓冲区溢出。
- 内存泄漏：缩短字符串时，同样需要重新分配内存释放掉不需要的那部分空间。如果忘了这一步，那么剩下那一部分空间将会一直处于未使用状态且无法分配给其他程序来使用，这就是内存泄漏。
- 内存重分配设计辅助的算法，并且可能需要执行系统调用，所以它通常是一个比较耗时的操作。




#### 1. 空间预分配

**描述**：空间预分配操作用于优化SDS字符串的增长操作。



**实现**：

- 当修改后的SDS长度小于1MB时，那么程序将分配和len相同大小的未使用空间。这时len的值和free的值相同。
- SDS的长度大于等于1MB时，那么会分配1MB的未使用空间。


**作用**：通过内存重分配和空间预分配的策略，Redis可以减少连续执行字符串增长操作所需的内存重分配次数，从而提高性能




#### 2. 惰性空间释放

**描述**：用于优化SDS字符串缩短操作。


**实现**：

- 缩短字符串时，程序并不立即使用内存重分配来回收缩短后多出的字节，而是使用free属性将这些字节的数量记录起来，并等待将来使用。


**作用**：SDS避免了缩短字符串时所需的内存重分配操作。并为将来可能出现的增长操作提供了优化；不过有可能造成内存空间的浪费。




#### 3. 二进制安全

**描述**：SDS的API都是二进制安全的。所有SDS API都会以处理二进制的方式来处理SDS存放在buf数组里的数据。




#### 4. 兼容部分C字符串函数

**描述**：SDS末尾保存的空字串，使得其可以重用一部分<string.h>库定义的函数，从而避免了不必要的代码重复实现。






### 与C字符串相比的区别

- 长度获取简单。SDS用len属性记录了SDS字符串的长度，因此只需常熟复杂度就可以获取到字符串长度，而C字符串并不记录长度，它需要遍历整个字符串才能得出字符串长度。
- len属性解决了扩充时C字符串的缓冲区溢出（buffer overflow）问题；
- free属性减少修改字符串时带来的内存重分配次数。


<br><br>

------

## 链表
### 描述

- 链表提供了高效的**节点重排**能力，以及顺序性的节点访问方式。
- 链表在redis中的应用非常广泛，列表键的底层实现之一就是链表（当一个列表键包含了数量比较多的元素，又或者列表中包含的元素都是比较长的字符串时）。
- 此外，发布与订阅、慢查询、监视器等功能也用到了链表，Redis本身还是用链表保存多个客户端的状态信息。




### 定义

adlist.h/list表示一个链表，而每个链表节点使用一个adlist.h/listNode结构表示。



``` c
typedef struct list {
  listNode *head;
  listNode *tail;
  unsigned long len; // 节点数量
} list;



typedef struct listNode {
  struct listNode *prev;
  struct listNode *next;
  // 任意类型的值
  void *value;
} listNod
```


<br><br>

------


## 字典
### 描述

- 字典，又称符号表、关联数组、映射（map），用于保存键值对的抽象数据结构。
- C语言并没有内置字典。因此字典又Redis本身实现。
- 除了用来表示数据库外，字典还是哈希键的底层实现之一，当一个哈希键包含的键值对比较多，又或者键值对中的元素都是长字符串时，Redis就会使用字典作为哈希键的底层实现。




### 定义

dict.h/dict表示一个字典。

``` c
typedef struct dict {
  // 特定类型函数
  dictType *type;
  // 私有数据
  void *privdata; 
  // 哈希表
  dictht ht[2]; 
  // rehash索引，没有进行rehash时，值为-1
  int trehashidx;
} dict;
```



dict.h/dictht表示一个哈希表。

``` c
typedef struct dictht {
  // 哈希表，这里相当于Java中的 Object [] table;
  dictEntry **table;
  // 大小，指数组
  unsigned long size; 
  // 哈希表大小掩码，总是用于计算索引值，总是等于size - 1
  unsigned long sizemask; 
  
  // 哈希表已有节点的数量
  unsigned long used; 
} dictht;
```



而每个哈希节点使用dictEntry表示。

``` c
typedef struct dictEntry{
  void *key;


  // 值 可以是一个对象指针，或uint64_t整数或int64_t整数
  union {
    void *val;
    uint64_t u64;
    int64_t s64;
  } v; 


  // 指向下一个哈希节点，用于解决键冲突（链地址法）
  struct dictEntry *next;
} dictEntry
```




### 哈希算法

字典用作数据库底层实现，或哈希键的实现时，Redis使用的**MurmurHash2**算法来计算哈希值

key：键

ht[x]：没有rehash时为ht[0]，rehash时ht[1]

计算哈希值：hash = dict->type->hashFunction(key);

计算在数组中的下标：index = hash & dict->ht[x].sizemask;




### 哈希冲突

Redis使用了链地址法解决，在产生冲突的下标地址的链表中，使用头插法插入哈希节点。




### 渐进式的Rehash

Redis也有一个负载因子用于控制，哈希表的数组大小。当负载因子超过承受限制或远低于预期时，就会进行rehash的操作，进行扩展或缩小哈希表。

负载因子（load factor）计算方式：load_factor = ht[0].used / ht[0].size



扩展时：

**条件：**一般情况下，负载因子大于等于1，就进行扩展；在执行BGSAVE或BGREERITEAOF命令时，负载因子大于等于5才开始扩展。

方法：

1. 分配ht[1]的空间，size为：大于等于ht[0].used * 2的第一个2^n数
2. 



收缩时：

**条件：**负载因子小于0.1，就开始收缩


<br><br>

------


## 跳跃表
### 描述

Redis使用其作为有序集合键的底层实现之一。当一个有序集合包含的元素数量比较多，又或者集合中的元素都是长字符串时，Redis就会使用跳跃表作为有序集合的底层实现。


<br><br>

------


## 整数集合


### 描述

整数集合是Redis用于保存整数值的集合抽象数据结构，当一个集合只包含整数值元素，且元素数量不多时，Redis就会使用整数集合作为集合键的底层实现。

它可以保存int16_t、int32_t、int64_t的整数值，并且保证集合有序且不会出现重复。




### 定义

每个intset.h/intset结构表示一个整数集合

``` c
typedef struct intset {
  uint32_t encoding; // 编码 三种类型int16_t、int32_t、int64_t
  uint32_t length; // 元素数量
  int8_t contents []; // 保存元素的数组，其保存的正真类型取决于encoding
}
```




### 升级

新元素超过当前类型的范围时，就要向上转型，redis称为升级（upgrade）。例如16-32，32-64。当然，整数集合不支持降级操作，一旦升级，编码就会一直保持升级后的状态，这一点跟Java一样。

**步骤：**

1. 根据升级类型，扩展底层数组的空间大小，并未新元素分配空间
2. 把数组中现有的元素转换成升级后的类型，并从最后一个元素开始将之移动到新的位置上（例如：16位的整数转为32位时，数组为每一个元素的空间都分配了32位。从最后一个元素开始移动，可以确保一次移动完毕，且不会造成数据损坏）
3. 将新元素添加到数组（由于新元素是向上转型，那么新元素只能是大于或小于当前数组的所有数的情况。比如一个32位的正整数或负整数）




<br><br>

------


## 压缩列表
### 描述

压缩列表是Redis为了节约内存而开发的，是由一系列**特殊编码**的**连续内存块**组成的顺序性数据结构。

也是列表键和哈希键的底层实现之一，当一个列表键只包含少量列表项，并且列表项要么是小整数值、要么是比较短的字符串，那么Redis就会使用压缩列表做列表键的底层实现；当一个哈希键只包含少量键值对，并且键值对的键和值要么是小整数值、要么是比较短的字符串，那么Redis就会使用压缩列表做列表键的底层实现。




###  定义

由于是连续的内存块，且经过特殊编码的，因此它跟之前的几种数据结构不同。整个压缩列表的结构如下：

``` c
zlbyte | zltail | zllen | entry1~~~entryn | zlend
zlbyet：列表占用的内存字节数
zltail：记录列表尾节点离列表的起始地址有多少字节
zllen：记录列表包含的节点数量
entry：每一个列表节点，数量不定
zlend：特殊值0xFF（十进制255），用于标记压缩列表的末端
```



每一个实体entry包含的内容如下：

``` c
previous_entry_lenth  |  encoding  |  content
previous_entry_lenth：记录压缩列表前一个节点的长度
encoding：记录节点content属性所保存数据的类型和长度
content：保存节点值，可以是一个字节数组或者整数
```



<br><br>

------

##  对象
###  描述：

Redis没有直接使用上述的数据机构来实现键值对数据库，而是基于这些数据结构创建一个对象系统。

Redis创建一个键值对时，最少会建两个对象，键的对象和值的对象；键总是一个字符串对象，而值可以是对象系统的任一种。




###  对象系统：

- **对象系统**包含：字符串对象、列表对象、哈希对象、集合对象和有序集合对象五种。
- 对象系统实现了基于引用计数技术的内存回收机制：当程序不再使用某个对象的时候（什么情况下才表示不再使用呢，即引用计数为0时），这个对象所占用的内存就会被自动释放。此外该技术实现了对象共享机制，通过让多个数据库键公用同一对象来节约内存。
- 对象带有访问时间记录信息，其用于计算数据库键的空转时长（未被调用），在服务器启用了maxmemory功能的情况下，空转时长较大的键可能会被服务器优先删除。




###  定义：

每个对象都由一个redisObject结构表示：

``` c
typedef struct redisObject{
  // 类型 REDIS_STRING REDIS_LIST REDIS_HASH REDIS_SET REDIS_ZSET
  unsigned type:4;
  unsigned encoding; // 编码 决定了该类型使用什么底层数据结构
  void *ptr; // 指向底层实现数据结构的指针
  int refcount; // 引用计数 用于内存回收机制、共享机制
  unsigned lru:22; // 记录对象的最后一次被访问时间
  // ...
} robj;
```



ptr指针所指向数据结构，它的类型由encoding属性决定。encoding记录了对象所使用的编码，即对象使用了什么数据结构作为底层实现。






###  对象的常用命令：

| object encoding | 值的编码                                    |
| --------------- | ------------------------------------------- |
| object refcount | 对象的引用计数                              |
| object idletime | 对象的空转时长，该命令不会修改对象的lru属性 |
| type            | 对象类型                                    |
| del             | 删除键以及值                                |
| rename          | 重命键                                      |
| expire          |                                             |




###  对象系统各编码使用的数据结构：

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




###  数据结构的转换：

| 类型         | 转换规则                                                     |
| ------------ | ------------------------------------------------------------ |
| REDIS_STRING | 为整数值且可以用long型表示使用int；当保存字符串，且长度小于等于39字节，使用embstr编码；否则使用SDS |
| REDIS_LIST   | 全是整数或短字符串且元素少使用压缩列表，否则使用链表         |
| REDIS_HASH   | 全是整数或短字符串且元素少使用压缩列表，否则使用字典         |
| REDIS_SET    | 全是整数且元素少使用整数列表，否则使用字典                   |
| REDIS_ZSET   | 全是整数或短字符串且元素少使用压缩列表，否则使用跳跃表和字典 |




###  字符串对象

int embstr raw

字符串对象是五种类型中，唯一一种会被其他四种对象嵌套的类型。

embstr编码与raw编码异同：

- 两者都使用了redisObject机构和sdsstr结构表示字符串
- 但是raw编码会调用两次内存分配函数来分别创建两个结构，而embstr只需一次且分配了连续的内存空间。
- 意味着释放内存时，前者也需要释放两次，而后者只需要释放一次。
- embstr是读取连续内存空间的数据，因此读取速度更快
- embstr是只读的，而raw可读写。embstr是专门用于保存短字符串的一种优化编码方式。embstr无法修改的只读对象。若要对其进行修改，会将embstr先转为raw对象，再执行修改命令



常用命令：

| set    |            |
| ------ | ---------- |
| get    |            |
| append | 在值后添加 |
| strlen | 字符串长度 |






###  列表对象

ziplist linkedlist

编码转换：
- 字符串元素的长度小于64字节，元素数量小于512个时，使用ziplist
- 通过配置文件修改：list-max-ziplist-value、list-max-ziplist-entries


常用命令：

| lpush          | 头部添加         |
| -------------- | ---------------- |
| lpop           | 头部删除         |
| rpush          | 尾部添加         |
| rpop           | 尾部删除         |
| llen           | 长度             |
| lindex         | 返回下标的元素   |
| lset           | 更新节点的元素   |
| lrange key s e | 展示范围内的元素 |






###  哈希对象

ziplist hashtable

当使用ziplist保存哈希键值时，将键和值都作为一个entry，然后以键在前值在后的顺序插入压缩表

编码转换：

- 所有键值对的键和值，都是用字符串，且长度小于64字节，键值对数量小于64个时，使用压缩表
- 通过配置文件修改：hash-max-ziplist-value、hash-max-ziplist-entries

常用命令：

| hset    | 添加键值对         |
| ------- | ------------------ |
| hget    | 获取键的值         |
| hdel    | 删除键对应的键值对 |
| hlen    | 长度               |
| hgetall | 获取所有键值对     |






###  集合对象

intset hashtable

使用hashtable时，字典的每个键都是一个字符串对象，值则全是NULL。

编码转换：

- 集合对象保存的元素都是用整数值，且元素数量不超过512个时，使用intset
- 通过配置文件修改：set-max-intset-entries

常用命令：

| sadd      | 添加元素                                               |
| --------- | ------------------------------------------------------ |
| scard     | 集合元素的数量                                         |
| sismember | 指定元素是否存在                                       |
| smembers  | 获取所有元素                                           |
| spop      | 随机删除一个元素并返回，在返回给客户端值后才会正真删除 |






###  有序集合对象

ziplist skiplist&hashtable

压缩表的表现形式：

- 使用压缩表时，元素按分值从小到大进行排序，分值小的靠近表头，分值大的靠近表尾。
- 压缩表中，每个元素用两个连续的entry表示，第一个保存元素，第二个保存元素的分值。

跳跃表和哈希表的表现形式：

skiplist编码的有序集合对象使用zset结构作为底层实现，其同时包含一个字典和一个跳跃表：

``` c
typedef struct zset{
  zskiplist *zsl; // 跳跃表
  dict *dict; // 字典
}zset;
```

虽然zset同时使用字典和跳跃表保存有序集合，但这两种结构都通过指针来共享相同元素的成员和分值，因此不会造成浪费额外的内存。

编码转换：

- 有序集合对象保存的元素长度都小于64字节，且元素数量小于128个时，使用ziplist
- 通过配置文件修改：zset-max-ziplist-value、zset-max-ziplist-entries

常用命令：

| zadd   | 添加元素                    |
| ------ | --------------------------- |
| zrem   | 删除元素中的指定成员        |
| zcard  | 集合元素的数量              |
| zcount | 分值在给定范围内的元素数量  |
| zrange | 返回给定索引范围的所有元素  |
| zrank  | 元素的排名（相当于index+1） |
| zscore | 给定元素的分值              |








###  内存回收

描述：

C语言并不具备自动内存回收的功能，Redis构建了一个引用计数（reference counting）计数实现的内存回收机制。

对于一个对象而言，它的生命周期为创建对象、操作对象、释放对象三个阶段。

引用计数：

程序通过跟踪对象的引用计数信息，在适当的时候自动释放对象并进行内存回收。

引用计数在对象系统中的属性为：refcount

refcount的状态变化：

- 创建新对象时，对象的refcount值初始化为1
- 对象被新程序使用时，refcount++
- 新程序使用结束时，refcount--
- 当对象的引用计数为0时，对象占用的内存会被释放






###  对象共享

描述：

对象的共享也是通过上述的refcount实现的。

哪些对象会被共享：

- Redis只对包含整数值的字符串对象进行共享。
- Redis初始化服务器时，创建0-9999的字符串对象，用以实现这些整数值的共享
- Redis不共享包含字符串的对象，那有会增加判断的复杂度，从而影响内存性能

共享的实现：

1. 将数据库键的值指针指向一个现有的值对象；
2. 将被共享的值对象的引用计数加一






###  对象的空转时长

描述：

lru属性，记录对象最后一次被访问的时间。

作用：

当服务器打开了maxmemory选项时，且回收内存的算法为volatile-lru或者allkeys-lru，那么当服务器占用的内存数超过了maxmemory设置的上限值时，空转时长较高的那部分键会优先被服务器释放，从而回收内存。


参考：[Redis Documentation](https://redis.io/documentation)

[Redis维基百科](https://en.wikipedia.org/wiki/Redis)

[Redis设计与实现-黄健宏]
