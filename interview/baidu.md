一面：
1、多城市最短距离
floyd算法了解下

2、mysql分页查询优化
3、重构怎么验证是否有bug？
用单元测试

二面：
1、mysql事务，
第一个事务：select * where name < n from user for update
第二个事务：insert into user (name = n)(插入一条name为n的数据)
请问第二条事务会不会阻塞第一条事务
答：会，参考以下博客
[MySQL的SELECT ...for update](https://www.cnblogs.com/wxgblogs/p/6849064.html)
[mysql select for update + 事务处理数据一致性](https://blog.csdn.net/leyangjun/article/details/88633588)

2、两个栈实现一个队列

3、随机抽奖算法，不知道有多少个人要抽奖，抽奖的时候马上决定是不是中奖，怎样保证公平性
https://gorpeln.com/article/15573877312
可以参考这个算法
