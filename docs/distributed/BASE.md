# BASE理论
参考：[Eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency)
## 概念
BASE即Basically Available（基本可用），Soft state（柔性状态），Eventual consistency（最终一致性）的缩写。

其本质上是对CAP中AP的一种延伸，其更注重可用性。
最终一致性类的服务也可以归类于BASE。
- Basically Available：系统保证了CAP理论中的可用性。
- Soft state：即使没有输入，系统的状态也可能随时间改变。
- Eventual consistency：随着时间推移，系统数据将最终一致。