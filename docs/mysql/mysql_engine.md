#MySQL数据库引擎

## InnoDB：
事务存储引擎（ transactional storage engine），MySQL默认存储引擎。它被设计用来处理大量的短期事务。

采用MVCC来支持高并发，并实现四个隔离级别。其默认级别是REPEATABLE READ，并且通过间隙锁（next-key locking）策略防止幻读的出现。间隙锁使得InnoDB不仅仅锁定查询涉及的行，还会对索引中的间隙进行锁定，以防止幻影行的插入。

<br><br>

---

### MyISAM：
非事务存储引擎（nontransactional storage engine）在MySQL5.1及以前是默认的存储引擎。提供了全文索引、压缩、空间函数。但是不支持事务和行级锁。并且，崩溃后无法安全恢复。但对于只读的数据，或者表比较小、可以忍受修复的操作，则还是可以使用该引擎。
	
在MySQL5.7中，InnoDB是新建表的默认引擎。InnoDB is the default storage engine for new tables. In practice, the advanced InnoDB performance features mean that InnoDB tables often outperform the simpler MyISAM tables, especially for a busy database.