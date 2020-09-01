##### 1、服务器全局变量count++,这个是不是原子性的，需不需要考虑多线程同步问题,如果需要，用什么方法可以实现，同时避免加锁导致的阻塞,了解下java的CAS，还有java的volatile关键字的作用

个人感觉count++，是要把内存数据复制到加法寄存器到，所以肯定存在复制数据问题，不会是原子性的，多个线程同时操作会存在数据被覆盖到情况。

参考

[字节跳动一面：i++ 是线程安全的吗？](https://www.javazhiyin.com/66316.html)

[Java 并发基础——线程安全性](https://www.cnblogs.com/NeilZhang/p/8682266.html)

[Java原子操作AtomicInteger的用法](https://www.jianshu.com/p/509aca840f6d)

[Java多线程i++线程安全问题，volatile和AtomicInteger解释？](https://segmentfault.com/q/1010000006733274)

##### 2、redis对单个key的操作，如何增加并发量,，如统计单个接口数量的次数pv

方案1:接口数量pv可以放在进程全局变量里面，如果pv增加到一定数量或者到一段时间定时写入到redis（incr命令），这个方法不完美，存在误差，因为不是实时上报，假如进程被kill了，那么就有一定的数量没有上报

方案2:由于redis使用到是集群，可以设置多个key，分布到不同的redis里面，然后统计这些key的value的总和

参考[关于Redis热点key的一些思考](https://juejin.im/post/6844903886667382798)

如果是统计接口uv的话，可以参考[浅析网站PV/UV统计系统的原理及其设计](https://blog.yuanpei.me/posts/3494408209/)

##### 3、redis如何利用多核多线程

##### 4、java的ArrayList和LinkList区别，操作的时间复杂度,arraylist如何很长的情况下，内部数据结构如何变化，是否影响时间复杂度

参考[When to use LinkedList over ArrayList in Java?](https://stackoverflow.com/questions/322715/when-to-use-linkedlist-over-arraylist-in-java)

##### 5、hashmap的内部原理，查找、插入原理和时间复杂度，碰撞系数为什么用0.75，内部红黑树的时间复杂度是多少？python的dict的内部数据结构

##### 6、int、bool、float、double、null多少个字节,int的范围

### Java 8大基本类型所占字节数（或 bit 数）

| 类型    | 存储需求 | bit 数 | 取值范围               | 备注                                                         |
| ------- | -------- | ------ | ---------------------- | ------------------------------------------------------------ |
| int     | 4字节    | 4*8    | -2147483648~2147483647 | 即 (-2)的31次方 ~ (2的31次方) - 1                            |
| short   | 2字节    | 2*8    | -32768~32767           | 即 (-2)的15次方 ~ (2的15次方) - 1                            |
| long    | 8字节    | 8*8    |                        | 即 (-2)的63次方 ~ (2的63次方) - 1                            |
| byte    | 1字节    | 1*8    | -128~127               | 即 (-2)的7次方 ~ (2的7次方) - 1                              |
| float   | 4字节    | 4*8    |                        | float 类型的数值有一个后缀 F（例如：3.14F）                  |
| double  | 8字节    | 8*8    |                        | 没有后缀 F 的浮点数值（例如：3.14）默认为 double             |
| boolean |          |        | true、false            | 布尔类型：布尔数据类型只有两个可能的值：真和假。使用此数据类型为跟踪真/假条件的简单标记。这种数据类型就表示这一点信息，但是它的“大小”并不是精确定义的。可以看出，boolean类型没有给出精确的定义，《Java虚拟机规范》给出了4个字节，和boolean数组1个字节的定义，具体还要看虚拟机实现是否按照规范来，所以1个字节、4个字节都是有可能的。这其实是运算效率和存储空间之间的博弈，两者都非常的重要。 |
| char    | 2字节    | 2*8    |                        | Java中，只要是字符，不管是数字还是英文还是汉字，都占两个字节。是因为 Java 中使用 Unicode 字符，所有字符均以2个字节存储。 |

[java中的null](https://blog.csdn.net/qq_25077777/article/details/80174763)
[python中的None](https://www.cnblogs.com/mika-blogs/p/10981239.html)

##### 7、资金的字段可以用浮点数吗？浮点数相加会不会导致精度问题，比如1.11+1.12会不会有精度问题

资金不能用浮点数，就算小数点少的浮点数相加也会存在精度问题，因为有的十进制小数是没办法用二进制精确表示的，比如0.33

##### 8、mysql如何避免脏数据

##### 9、分布式锁如何解决脏数据，redis的getset的原理

面试的时候说了getset，但是因为隔了一年多，忘了原理，所以答不上来，实际上使用set(ex, nx)就行了

##### 10、堆和栈的区别，方法内部的临时变量是放在堆里还是栈里

[堆与栈的区别](https://blog.csdn.net/K346K346/article/details/80849966)

##### 11、MYSQL、Mongo、Redis主从同步原理
