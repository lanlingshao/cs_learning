### MySQL事务原理

重做日志，回滚日志以及锁技术就是实现事务的基础

- 事务的原子性是通过 undo log 来实现的
- 事务的持久性性是通过 redo log 来实现的
- 事务的隔离性是通过 (读写锁+MVCC)来实现的
- 而事务的终极大 boss 一致性是通过原子性，持久性，隔离性来实现的！！！

原子性，持久性，隔离性折腾半天的目的也是为了保障数据的一致性！

总之，ACID只是个概念，事务最终目的是要保障数据的可靠性，一致性。

参考[MySQL事务的实现原理](https://mp.weixin.qq.com/s/79HhQsZRzzuskP5p5LNONA)
