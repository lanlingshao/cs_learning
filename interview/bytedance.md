### 幸福里
##### 1、redis持久化

##### 2、消息队列用什么？如何选型？redis的消息队列跟rabitmq有什么区别？

参考[Redis与RabbitMQ作为消息队列的比较](https://www.cnblogs.com/afeige/p/10908960.html)

##### 3、合并2个有序数组，不用额外空间存储
参考[88. Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/)

##### 4、合并k有有序数组

参考[23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)和分而治之的方案（分迭代和递归两种，迭代比较麻烦），然后结合上方的合并2个有序数组的方案

这个题目是《算法导论》2.3.1分治法的例子

计算此方法时间复杂度**（以后做算法题，一定要会计算时间复杂度）**

##### 5、数据库写入过快如何解决

可以将写入数据放在redis中暂存起来，用户查询直接查询redis，然后用消息队列降大量的写入请求批量插入到数据库，同时这个数据库索引尽量少，可以让写入更快。还有一点就是不要用数据库自增id了，因为自增id要通过读取数据库获取，所以用自定义的自增字符串（雪花算法）最好了


### 在线教育
##### 1、Redis消息队列实现原理
##### 2、Redis发布订阅原理
##### 3、Redis高可用实现方案
##### 4、Redis主从实现原理
##### 5、Redis多节点实现原理
##### 6、SQL count(*) count(1) count(id)有什么不同
##### 7、two sum算法，有序数组，如何实现时间复杂度O（n），空间复杂度O（1），不使用dict

因为是有序的，所以可以使用双指针算法

