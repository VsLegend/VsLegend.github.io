# Mysql常用语句

## 获取当前数据库的所有表名：
```sql
SELECT GROUP_CONCAT( a.table_name SEPARATOR ',' ) 
FROM ( 
SELECT table_name FROM INFORMATION_SCHEMA.TABLES WHERE table_schema = '库名' AND table_type = 'BASE TABLE' 
) a
```

## 获取表字段信息
```sql
SELECT *
FROM INFORMATION_SCHEMA.COLUMNS 
WHERE table_schema = '库名' AND table_name = '表名'
```

## 获取表信息
```sql
SELECT *
FROM INFORMATION_SCHEMA.TABLES
WHERE table_schema = '库名' AND table_name = '表名'
```

## 获取表属性（可转表格等内容）
```sql
SELECT
	COLUMN_NAME 列名,
	COLUMN_TYPE 数据类型,
	DATA_TYPE 字段类型,
	IS_NULLABLE 是否为空,
	COLUMN_DEFAULT 默认值,
	COLUMN_COMMENT 备注
FROM
    INFORMATION_SCHEMA.COLUMNS 
WHERE
    table_schema = '库名' 
AND table_name = '表名'
```


## 获取数据库表的所有字段：
```sql
SELECT GROUP_CONCAT( a.COLUMN_NAME SEPARATOR ',' ) 
FROM (
SELECT COLUMN_NAME 
FROM information_schema.COLUMNS 
WHERE table_name = '表名' AND table_schema = '库名'
) a
```

## 更新字段的部分数据
```sql
UPDATE 表名 SET url = REPLACE(url,'/document','/opt/storage');
```

## 引用-多表关联删除：
```sql
set @userId = 'id';
DELETE FROM 表名1 WHERE user_id = @user;
DELETE FROM 表名2 WHERE user_id = @user;
```

## 按指定规则排序
```sql
SELECT *
FROM 表名
ORDER BY FIELD(sequence, 2, 3, 1, 5, 4)
```

## 增量更新字段
```sql
-- 新增
alter table user add COLUMN new1 VARCHAR(20) DEFAULT NULL;
-- 删除
alter table user DROP COLUMN new2;
-- 修改
alter table user MODIFY new1 VARCHAR(10);  -- 修改一个字段的类型
alter table user CHANGE new1 new4 int;  -- 修改一个字段的名称，此时一定要重新指定该字段的类型
```

## 增量更新索引脚本
```sql
ALTER TABLE `user` ADD KEY (`name`)
-- 添加前缀索引
ALTER TABLE `user` ADD KEY (`name`(1))
```

## 查看数据库事务相关的表
```sql
-- 查询正在执行的事务
SELECT * FROM information_schema.innodb_trx;
-- 查看正在锁的事务
SELECT * FROM information_schema.innodb_locks;
-- 查看等待锁的事务
SELECT * FROM information_schema.innodb_lock_waits;
```
