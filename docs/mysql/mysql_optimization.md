# MySQL性能优化

## 概览
**目标**：降低响应时间

<br>

**原则**：无法测量就无法有效地优化，应该花费大量的时间测量响应时间花费在哪；

<br>

**优化的作用**

- CPU利用率的提升，不过这只是一种现象，不是一个好的可度量的目标；

- 提升每秒查询量，从而提高吞吐量；

- 降低响应时间，这是优化的最终目标。

<br>

**优化的建议**


- 优化值得优化的查询，一些只占总响应时间很小比重的查询是不值得优化的；

- 注意某些执行频流低，总响应时间占比并不突出的、执行速度慢的查询；

- 分析慢查询时，要注意是执行时间慢还是等待时间长。

<br><br>

---

## 性能剖析(profiling)
**步骤**：测量任务所花费的时间；将结果进行统计和排序，将重要任务排在前面。

<br>

**花费时间的分析**：

- 基于执行时间的分析：什么任务的执行时间长

- 基于等待时间的分析：判断任务在什么地方被阻塞的时间最长，譬如等待I/O或者锁等待

<br>

**使用工具对性能进行剖析**
pt-query-digest


<br><br>

---

## MySQL关键字的性能分析
**整数、实数**

对于所有整数、实数类型，我们仅仅能决定MySQL在内存和磁盘中保存的存储类型（int/4字节 bigint/8 float/4 double/8 ），在实际计算时则不同；整数计算一般使用64位的BIGINT整数，即使32位环境也是如此；实数计算使用DOUBLE作为内部浮点计算的类型。

<br>

**FLOAT、DOUBLE、DECIMAL**

指定浮点精度，会使得MySQL会悄悄选择不同的数据类型，或者在存储时对值进行取舍。这些精度的定义是非标准的，所以最好只指定数据类型，不指定精度。

<br>

**DECIMAL**

用于存储精确的小数，也可以存储比BIGINT还大的整数。DICIMAL支持精确计算（MySQL4.1前使用浮点计算来实现DECIMAL的计算，这样会由于精度损失导致一些的奇怪的结果）。

CPU是不支持对DECIMAL的直接计算的，在MySQL5.0以上的版本中，MySQL服务器自身实现了DECIMAL的高精度计算。相对的，CPU直接支持原生浮点的计算，所以浮点运算明显更快。因为需要额外的空间和计算开销，尽量只在对小数进行精确计算时才使用DECIMAL。在数据量比较大的情况下，可以考虑使用BIGINT代替DECIMAL（根据精度来决定相乘的倍数即可）。

<br>

**VARCHAR、CHAR**

VARCHAR用于存储可变长度的字符串。除了存储时所用的空间外，它还需要1-2个额外的字节记录字符串的长度：列大于等于255字节时，使用2个字节存储长度，否则使用1个。VARCHAR在4.1之前MySQL还会剔除末尾的空格。

CHAR根据定义分配的字符串长度分配空间，它的长度是定长的，并且存储时MySQL会剔除末尾的所有空格。并且CHAR会根据需要采用空格进行填充以方便比较。
> 比较：
> VARCHAR适合存储长度不定，且长短差距较大的字符串；CHAR适合存储很短的字符串、或者所有值都接近同一个长度（MD5密码，Y或N，0或1）。对于很短的字符串来说，CHAR只需要一个字节，而VARCHAR最低需要两个字节存储（一个记录长度的额外字节）。
> 对于字符长度不确定的列来说，VARCHAR更节省存储空间，因为它仅使用必要的空间，这对查询存储性能也有一定的帮助。对于定长字符串来说，CHAR更节省空间，因为VARCHAR需要额外的字节存储字符串长度。

<br>

**DATETIME、TIMESTAMP**

DATETIME保存大范围的值，从1001年到9999年，精度位秒。它把日期和时间封装到格式为YYYYMMDDHHMMSS的整数中，与时区无关。使用8字节的存储空间。

TIMESTAMP保存了从1970年1月1日午夜（格林尼治时间GMT）以来的秒数，表示1970年到2038年的时间。使用4字节的存储空间。
> 比较：
> 两者的时间范围不同，但是表示形式都为yyyy-mm-dd hh:mm:ss。
> TIMESTAMP会自动检索当前时区并进行转换，如果储存时的时区和检索时的时区不一样，那么拿出来的数据也不一样；DATATIME不会进行时区的检索，存什么拿到的就是什么。
> 对于NULL值，TIMESTAMP会自动储存当前时间，而DATETIME会储存NULL。 

<br>

**INET_NTOA INET_ATON**

IP转换
一个 int 型的数据占 4 个字节，每个字节 8 位，其范围就是 0~(2^8-1)，而 ipv4地址 可以分成4段，每段的范围是 0~255 刚刚好能存下
```sql
SELECT INET_NTOA(ip)
FROM log_info
```


<br><br>

---


## Schema设计和数据类型优化
**选择优化的数据类型**

- 更小的通常更好：一般情况下，可以使用正确存储数据的最小数据类型。更小的数据类型通常更快，因为它们占用更少的磁盘、内存和CPU缓存，并且处理时需要的CPU周期更少。

- 简单就好。简单的数据类型通常需要更少的CPU周期。整性比字符串操作代价更低。应该使用数据库内建类型，而不是字符串存储日期和时间。用整性存储IP地址。

- 尽量避免NULL

<br>

**Schema设计优化**

- 避免太多列
>服务器层和存储引擎层之间通过行缓冲格式拷贝数据，然后将缓存内容解码成各个列。这个操作代价是比较高的，通常这个代价依赖于列的数量。

- 查询不要关联太多的表
>MySQL限制了每个关联操作最多关联61张表。如果需要快速查询且并发性好，单个查询最好控制在12张表以内做关联。

- 合理使用NULL值：
>MySQL会在索引中存储NULL值，而Oracle不会。

- 使用范式的优缺点

- 使用非范式的优缺点


<br><br>

---


## 其他优化方法
**使用STRAIGHT_JOIN关键字**

参考：[join clause](https://dev.mysql.com/doc/refman/5.7/en/join.html)

**说明**

STRAIGHT_JOIN is similar to JOIN, except that the left table is always read before the right table. This can be used for those (few) cases for which the join optimizer processes the tables in a suboptimal order.
STRAIGHT_JOIN关键字作用与JOIN类似，但是前者总是会先读取左边的表后读取右边的表，这一功能可以解决join优化器对表顺序读取的优化处理（join优化器会根据规则，选择先读取那一张表的数据，仅对JOIN和INNER JOIN生效）。

**语法规则**
```sql
joined_table: {
    table_reference [INNER | CROSS] JOIN table_factor [join_specification]
  | table_reference STRAIGHT_JOIN table_factor
  | table_reference STRAIGHT_JOIN table_factor ON search_condition
  | table_reference {LEFT|RIGHT} [OUTER] JOIN table_reference join_specification
  | table_reference NATURAL [{LEFT|RIGHT} [OUTER]] JOIN table_factor
}
```


