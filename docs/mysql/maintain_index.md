# 维护索引和表
参考：高性能MySQL

## 表损坏问题


1. 查看表状态：show table status;

1. 检查是否发生表损坏或索引错误：check table table_name;

1. 修复损坏的表：repair table table_name;

1. 重建表：alter table table_name engine=inndb;

<br><br>

---

## 更新索引统计信息

MySQL优化器基于成本的模型，而衡量成本的一个主要指标就是一个查询需要扫描多少行。

1. 重新生成统计信息：analyze table table_name;

1. 查看索引的基数（Cardinality）：show index from table_name;


<br><br>

---

## 减少索引和数据的碎片
B-tree索引可能回碎片化，碎片化的索引可能会以很差或者无序的方式存储在磁盘上。同时，由于设计，B-Tree需要随机磁盘访问才能定位到叶子页，所以随机访问不可避免。

**表的数据也可能碎片化**，且更加复杂，有以下三种：


- 行碎片：指数据行被存储在多个地方的多个片段中

- 行间碎片：指逻辑上顺序的页或行在磁盘上不是顺序存储的

- 剩余空间碎片：指数据页中有大量的空余空间，导致服务器读取大量不需要的数据，造成浪费。


<br><br>

---

## 其他

重新整理数据（也可以通过导入再导出的方式，相当于为表重新建立空间并保存，同时保存的数据一般都是连续保存在磁盘中的）：optimize table table_name;

对于不支持optimize命令的存储引擎，可以通过重建表的方式：alter table table_name engine=innodb;

索引和表的碎片化需要通过实际测量而非随机假设来确定。


