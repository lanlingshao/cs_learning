#### 使用 memory_profiler 分析python程序内存占用情况    

##### 安装

```
$ pip install memory_profiler
```

##### 使用

因为要排查的项目使用的是flask框架，所以只需要在代码的函数上加上@profile装饰器，然后再运行项目，只要添加了装饰器的代码运行了，就会自动打印日志。

代码如下：

```python
    @profile(precision=4)  # precision 显示小数点后的位数，如果不加话，增加内存小于1M就不会显示了
    def request_test_config(cls, md5_id):
        test_id = getattr(cls, 'test_id')
        try:
            ab_test_channel = grpc.insecure_channel(ABTest_RPC_ADDR)
            ab_test_client = ab_test_pb2_grpc.StragegyStub(ab_test_channel)
            response = ab_test_client.GetStragegy(ab_test_pb2.GetRequest(md5_id=md5_id)
            rs = json.loads(response.result)
            if rs['code'] == 1:
                return rs['t_config']
            else:
                if rs['code'] not in [-1001, -1002]:
                return None
```

打印出来的日志如下：

```
Line #    Mem usage    Increment  Occurences   Line Contents
============================================================
    29 145.8594 MiB 145.8594 MiB           1       @classmethod
    30                                             @profile(precision=4)
    31                                             def request_test_config(cls, md5_id):
    32 145.8594 MiB   0.0000 MiB           1           test_id = getattr(cls, 'test_id')
    33 145.8594 MiB   0.0000 MiB           1           try:
    34 145.8633 MiB   0.0039 MiB           1               ab_test_channel = grpc.insecure_channel(ABTest_RPC_ADDR)
    35 145.8633 MiB   0.0000 MiB           1               ab_test_client = ab_test_pb2_grpc.StragegyStub(ab_test_channel)
    36 145.8672 MiB   0.0039 MiB           1               response = ab_test_client.GetStragegy(ab_test_pb2.GetRequest(md5_id=md5_id, test_id=test_id), timeout=cls.timeout)
    37 145.8672 MiB   0.0000 MiB           1               rs = json.loads(response.result)
    38 145.8672 MiB   0.0000 MiB           1               if rs['code'] == 1:
    39 145.8672 MiB   0.0000 MiB           1                   return rs['t_config']
    40                                                     else:
    41                                                         # -1001 实验不存在  -1002 实验已停用
    42                                                         if rs['code'] not in [-1001, -1002]:
    43                                                             DingTalkMessage().send_msg(cls.__name__ + '实验返回code:%s' % str(rs))
    44                                                         return None
```

从日志可以看出，内存的增加是在第36行，增加了0.0039 MiB。

现在已经知道发生内存泄露的代码位置了，那么为什么会泄露呢？这个问题也是分析了好久，具体原因如下：

这个request_test_config方法中，每一次请求都创建了一个ab_test_client，而ab_test_client是一个grpc的连接，也就是说，每次请求都会新建一个grpc连接。而这个代码是运行在定时框架**apscheduler**中的一个**BackgroundScheduler**定时任务中的，因为这个定时任务是后台周期运行的线程，而这个线程是不会被销毁了，所以内存不会回收。经过验证，这段代码如果运行在flask的http请求代码里面的话，内存不会增加，因为请求返回给客户端后，线程就销毁了，内存也就回收了。

**解决方案**：

创建一个全局的ab_test_client连接，每次调用request_test_config方法的时候引用ab_test_client连接，而不是重新创建一个连接。代码如下：

```python
	ab_test_channel = grpc.insecure_channel(ABTest_RPC_ADDR)
  ab_test_client = ab_test_pb2_grpc.StragegyStub(ab_test_channel)
  
  def request_test_config(cls, md5_id):
        test_id = getattr(cls, 'test_id')
        try:
            ab_test_channel = grpc.insecure_channel(ABTest_RPC_ADDR)
            ab_test_client = ab_test_pb2_grpc.StragegyStub(ab_test_channel)
            response = ab_test_client.GetStragegy(ab_test_pb2.GetRequest(md5_id=md5_id)
            rs = json.loads(response.result)
            if rs['code'] == 1:
                return rs['t_config']
            else:
                if rs['code'] not in [-1001, -1002]:
                return None
```

