---
title:  "Java并发： 使用CompletionService来接收线程池计算的结果"
author: WangJwi
categories:
- java
tags:
- thread pool
- concurrency
---

导读：当我们向线程池提交一组任务，并希望返回结果时，我们通常会保留任务的Future，并在循环中反复调用isDone判断该任务是否完成。这一方法虽然可行，但有些繁琐，幸运的是，CompletionService可以更好的完成这一任务。


<br><br>

------

# Java并发 使用CompletionService来接收线程池计算的结果




------

参考：

[Redis Documentation](https://redis.io/documentation)

[Redis维基百科](https://en.wikipedia.org/wiki/Redis)