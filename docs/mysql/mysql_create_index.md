#MySQL中使用InnoDB的索引创建

以下的索引创建都以user表为例，当然在建表的时候可以直接在create见表语句中直接添加索引。

```sql
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL,
  `code` varchar(40) DEFAULT NULL,
  `name` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

```

<br><br>

---

## 主键索引
描述：属于聚簇索引。不允许有NULL和重复值。只能存在一个主键索引。
```sql
-- 
PRIMARY KEY (`id`) USING BTREE
--
ALTER TABLE `user` ADD PRIMARY KEY (id)
-- 或者创建一个多列主键
ALTER TABLE `user`
ADD CONSTRAINT pk_userID PRIMARY KEY (`code`,`name`)
-- 撤销主键
ALTER TABLE `user` DROP PRIMARY KEY
```

以下索引都属于二级索引
<br><br>

---

## 普通索引
描述：允许重复值，允许多个NULL值。多列索引可以重复。
```sql
-- 建表create
KEY `uqIndex` (`code`) USING BTREE
-- 创建索引的方式
CREATE INDEX `nameIndex` ON `user` (`name`)
-- 多列索引
CREATE INDEX `nameIndexs` ON `user` (`code`,`name`)
-- 更新表的方式
ALTER TABLE `user` ADD KEY (`name`)
-- 删除指定索引
DROP INDEX `nameIndex` ON `user`;

```
<br><br>

---

## 唯一索引
描述：不允许重复值，允许多个null值。多列索引要求多列索引的组合是唯一的。
```sql
-- create
UNIQUE KEY `uqIndex` (`code`) USING BTREE
-- 单列
CREATE UNIQUE INDEX `uqIndex` ON `user` (`name`)
-- 多列索引
CREATE UNIQUE INDEX `uqIndexs` ON `user` (`code`,`name`)

```
<br><br>

---

## 前缀索引
描述：实际上就是截取列的部分字段进行索引。这一索引用以解决长字段的索引问题。它的语法跟普通索引一样，只是在列字段后加上表示截取数量的数字。
```sql
CREATE INDEX `nameIndex` ON `user` (`name`（1）)
ALTER TABLE `user` ADD KEY `nameIndex` (`name`(1))

```
