# MYSQL函数使用

## REPLACE
获取UUID
```sql
SELECT REPLACE(UUID(),'-','')
```


## CONVERT-转换
数字格式化
```sql
SELECT CONVERT(58.2453421, DECIMAL(4,2))
```

