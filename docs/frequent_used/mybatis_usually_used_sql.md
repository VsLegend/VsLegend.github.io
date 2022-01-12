# Mybatis奇淫巧计

## 使用foreach实现：传入参数也放在查询结果中
```sql
SELECT q.id questionId, q.correct_answer correctAnswer, a.studentQuestion
FROM e_question q
LEFT JOIN
<foreach collection="list" item="dto" open="(" separator="UNION" close=")">
    SELECT #{dto.questionId} questionId, #{dto.studentQuestion} studentQuestion
</foreach>
a ON q.id = a.questionId
WHERE q.type = #{code}
```
