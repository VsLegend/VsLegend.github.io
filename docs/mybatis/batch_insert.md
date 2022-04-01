
## mybatis 批量插入的方法

### 1. 通过for循环调用单次的插入操作


### 2. 通过mybatis<foreach> 标签批量插入操作


### 3. 通过SqlSession将操作方式修改为ExecutorType.BATCH并执行批量插入（官方推荐）


## 性能分析



[Mybatis Java API](https://mybatis.org/mybatis-3/java-api.html)
[Insert Statements](https://mybatis.org/mybatis-dynamic-sql/docs/insert.html)
[Optimizing INSERT Statements](https://dev.mysql.com/doc/refman/5.6/en/insert-optimization.html)