#### mysql分页查询优化

#### mysql为什么不推荐使用唯一索引，如果不使用唯一索引，应该如何保证唯一性

#### 如何保证数据库、缓存一致性
腾讯面试官说：一个线程读，一个线程写，要使用线程同步，不过也要看具体的业务场景来选择方案.

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
#### mysql分库分表如何实现？分库分表后业务层应该如何改变？
[不停机分库分表迁移](https://www.jianshu.com/p/223d71421f49)
[mysql千万级数据分表迁移方案](https://jordanzheng.github.io/mysql-sharding/)
[进来抄作业：一次完美的分库分表实践！](https://database.51cto.com/art/202012/637727.htm)

#### 数据库主键为什么用自增，不用uuid
[自增还是UUID？数据库主键的类型选择，为啥不能用uuid做MySQL的主键？](https://www.cnblogs.com/goloving/p/13663276.html)

#### 分库分表之后，id 主键如何处理？
[分库分表之后，id 主键如何处理？](https://zhuanlan.zhihu.com/p/54838983)
[分库分表的 9 种分布式主键 ID 生成方案，挺全乎的](https://xie.infoq.cn/article/ba7af5c25c5dab951f4633212)

#### 雪花id重复如何避免？如果重复发生，应该如何处理？
[雪花算法（snowflake）生成Id重复问题](https://www.jianshu.com/p/71286e89e0c7)
[记一次线上 Snowflake 算法 id 重复事件复盘](https://blog.csdn.net/weixin_42444592/article/details/109643200)
