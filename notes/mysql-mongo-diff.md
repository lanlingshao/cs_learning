现在新版的mongo采用的是wiredTiger引擎，该引擎也是用的B+树，但是与mysql的innodb引擎的B+树有区别，不同的是，它的叶子节点之间没有通过链式结构形成一个链表，而是每个叶子节点以page 的方式存储了数据
[mongodb wiredTiger 存储引擎索引原理的理解总结](https://blog.csdn.net/zwzwzw0a0s/article/details/106584863)

[树结构系列（四）：MongoDb 使用的到底是 B 树，还是 B+ 树？](https://www.cnblogs.com/chanshuyi/p/tree-data-structure-04-mongo-db.html)

[WiredTiger存储引擎之一：基础数据结构分析](https://mongoing.com/topic/archives-35143)



mongo新版本也支持事务了

mongo对字段数据类型没有强制约束，同一个字段，不同的document可以使用不同的数据类型
