1、项目中使用redis的哪些方面？
答：
1、作为限流，可以对用户访问进行限流，防止用户刷接口，同时还对报警进行限流，防止相同的报警报警多次，可以使用滑动窗口算法，也可以使用令牌算法，令牌算法暂时没有使用，可以参考[python 基于redis的令牌桶限流](https://blog.csdn.net/u011519550/article/details/109246320)
2、分布式锁
3、数据库缓存
4、统计活跃用户，用zset
5、排行榜, zset, 这个待研究，不是自己写的
6、bitmap，统计测试管理平台的端口占用情况