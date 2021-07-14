#### MySQL事务原理

重做日志，回滚日志以及锁技术就是实现事务的基础

- 事务的原子性是通过 undo log 来实现的
- 事务的持久性性是通过 redo log 来实现的
- 事务的隔离性是通过 (读写锁+MVCC)来实现的
- 而事务的终极大 boss 一致性是通过原子性，持久性，隔离性来实现的！！！

原子性，持久性，隔离性折腾半天的目的也是为了保障数据的一致性！

总之，ACID只是个概念，事务最终目的是要保障数据的可靠性，一致性。

参考[MySQL事务的实现原理](https://mp.weixin.qq.com/s/79HhQsZRzzuskP5p5LNONA)

#### mysql事务悲观锁
例1: 
事务1: select * from user where age=30 for update; 
事务2: update user set name='dfd' where age = 40;
如果age上面没有索引，会产生表锁，则事务2会阻塞，如果age上面有索引，会产生行锁，则事务2不会阻塞。

例2: 
事务1: select * from user where age<30 for update; 
事务2: update user set name='dfd' where age = 10; 
事务3: update user set name='dfd' where age = 30; 
事务4:update user set name='dfd' where age = 40; 
事务5:insert into user (name, age) values ('fdgdd',30); 
事务6:insert into user (name, age) values ('fdgdd',31);
如果age上面没有索引，会产生表锁，事务2、3、4、5、6都会阻塞
如果age上面有索引，则会产生行锁，事务2、3、会阻塞，事务4、5、6不会阻塞，**这里就很奇怪了，为什么age=30的update会阻塞，而insert不阻塞？**

例3: 
事务1: select * from user where id<30 for update; 
事务2: update user set name='dfd' where age = 10; 
事务3:update user set name='dfd' where age = 40;
事务1是在id<30的行上面加了行锁，与例2不同的是，例2不是对主键进行范围加锁，所以例2会导致表锁。

例4: 
事务1: select * from user where id=30 for update; 
事务2: update user set name='dfd' where age = 10; 
事务3:update user set name='dfd' where age = 40;
事务1只对id=30的行加行锁，不会影响其他行

例子5: 
事务1：select * where name < n from user for update  
事务2：insert into user (name = n)(插入一条name为n的数据)
如果name上面有索引的话，第一条事务不会阻塞第二条事务，如果name上面没有索引，则会阻塞。

例子6: 
事务1：select * where name <= n from user for update  
事务2：insert into user (name = n)(插入一条name为n的数据)
无论有没有索引，第一条事务都会阻塞第二条事务

例子7:
事务1: select * from user where age=30 for update;
事务2: insert into user (name, age) values ('vcvv',28);  不阻塞
事务3: insert into user (name, age) values ('vcvv',29);  阻塞
事务4: insert into user (name, age) values ('vcvv',30);  阻塞
事务4: insert into user (name, age) values ('vcvv',31);  不阻塞
事务5: update user set name = 'dfdf' where age=28;       不阻塞
事务6: update user set name = 'dfdf' where age=29;       不阻塞
事务7: update user set name = 'dfdf' where age=30;       阻塞
事务7: update user set name = 'dfdf' where age=31;       不阻塞
# TODO 待总结经验

例子7:
事务1: select * from user where age<=30 for update;
事务2: insert into user (name, age) values ('vcvv',28);  阻塞
事务3: insert into user (name, age) values ('vcvv',29);  阻塞
事务4: insert into user (name, age) values ('vcvv',30);  阻塞
事务4: insert into user (name, age) values ('vcvv',31);  阻塞
事务5: update user set name = 'dfdf' where age=28;       阻塞
事务6: update user set name = 'dfdf' where age=29;       阻塞
事务7: update user set name = 'dfdf' where age=30;       阻塞
事务7: update user set name = 'dfdf' where age=31;       阻塞
# TODO 待总结经验



总结：
- 对于主键指定值或者范围的悲观锁，只会导致行锁
- 对于非主键的指定值或者范围的查询加悲观锁，分为两种情况：
  1、有索引，导致行锁
  2、没索引，导致表锁
- 对于非主键的指定范围查询加悲观锁：
  1、有索引，导致行锁**（这里有个特例，就是如果age<30的话，age=30的update会阻塞，但是insert不会阻塞，原理是使用了next_key_lock）**。
  2、无索引，导致表锁

参考以下博客
[MySQL的SELECT ...for update](https://www.cnblogs.com/wxgblogs/p/6849064.html)
[mysql select for update + 事务处理数据一致性](https://blog.csdn.net/leyangjun/article/details/88633588)
