#### mysql分页查询优化

#### mysql为什么不推荐使用唯一索引，如果不使用唯一索引，应该如何保证唯一性

#### 如何保证数据库、缓存一致性
腾讯面试官说：一个线程读，一个线程写，要使用线程同步，不过也要看具体的业务场景来选择方案.

#### mysql分库分表如何实现？分库分表后业务层应该如何改变？
#### mysql如何防止sql注入
使用pymysql提供的参数化语句防止注入
内部执行参数化生成的SQL语句，对特殊字符进行了加\转义，避免注入语句生成
例如：
select * from project where id = 1 or 1 = 1,
会被转义成select * from project where id = '1 or 1 = 1',导致只返回id=1的记录，不会返回全表记录
select * from project where name = 'wang' or 1 = 1
会被转义成select * from project where name = '\'wang\' or \'1 = 1\''，不返回任何记录，避免了返回全表记录

[PyMySQL防止SQL注入](https://www.cnblogs.com/freely/p/6798717.html)

#### 线上数据库迁移，表拆分如何不停服进行