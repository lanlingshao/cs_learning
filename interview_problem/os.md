#### 线上服务器的进程数和线程属应该设置为多少比较好？
即使服务器的配置都是一样的，但是不一样的程序，处理请求所用的时间也是不同的，这就能影响到uwsgi参数的配置。最好的办法就是通过 压测来测试，从而不断调优uwsgi的参数
[解决uwsgi开多少进程才能最大使用服务器](https://blog.csdn.net/dqchouyang/article/details/106569657)
[面试问我，创建多少个线程合适？我该怎么说](https://www.cnblogs.com/FraserYu/p/12657701.html)

