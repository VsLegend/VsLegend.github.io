---
title:  "Java并发（一）---多线程的异常捕获"
author: WangJwi
categories:
- java
tags:
- thread pool
- concurrency
---

导读：当我们的线程池中任务数量多且线程执行存在未知的异常时，我们很可能会忽略它们，从而使得数据缺失或流程中断。因此如何捕获和处理这些异常就显得尤为重要。


<br><br>

------

# Java并发（一）---多线程的异常捕获与处理




------

参考：

[Redis Documentation](https://redis.io/documentation)

[Redis维基百科](https://en.wikipedia.org/wiki/Redis)