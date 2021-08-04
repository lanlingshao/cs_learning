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
**代码实现:**

```python
import time

from apps import rds
from apps.logs import logger


# 全量迁移逻辑
def migrate():
    # 查询当前表最大id
    max_id = execute("select max(id) from user_comment")
    temp_min_id = 0
    step_size = 1000
    temp_max_id = 0
    while temp_min_id <= max_id:




# 被动迁移逻辑，所有插入、更新、删除的sql逻辑，都要在update_insert_delete中调用这个
def migrate_passive(user_id, sql):
    cache_key = 'migrate|%s' % user_id
    status = rds.get(cache_key)
    if status == "completed":
        logger.info('user data migrate completed, user_id = %s' % user_id)
    elif status == 'migrating':
        time.sleep(0.1)
        while rds.get(cache_key) == 'migrating':
            # 此处可以增加防止死循环逻辑，虽然正常情况下不会发生 ，防止程序崩溃，缓存状态未修改为completed（可报警，待手动处理）
            time.sleep(0.1)
    else:
        # 开始迁移工作
        lock_key = 'migrate|lock|%s' % user_id
        if rds.set(lock_key, nx=True, ex=3600):
            write_sharded_new_table(sql)
            rds.set(cache_key, 'completed')
            rds.delete(lock_key)
            return True
        else:
            # 没获得分布式锁，说明有其他线程在执迁移，这个时候可以等待迁移完成
            while not rds.set(lock_key, nx=True, ex=3600):
                time.sleep(0.1)
    return False


def update_insert_delete(user_id):
    sql = "insert ... into user_comment ..."  # 插入、删除、更新的sql语句
    if not migrate_passive(user_id, sql):
        # 在migrate_passive中迁移写入新表的，不需要再写入新表了
        write_sharded_new_table(sql)
    write_old_table(sql)


# 写入旧表
def write_old_table(sql):
    pass

# 按照user_id进行hash写入分表后的新表
def write_sharded_new_table(sql):
    pass
```

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
