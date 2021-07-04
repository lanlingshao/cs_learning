### 限流算法
在开发**高并发**系统时，有三把利器用来保护系统：**缓存、降级和限流**。那么何为限流呢？顾名思义，限流就是限制流量，就像你宽带包了1个G的流量，用完了就没了。通过限流，我们可以很好地控制系统的qps，从而达到保护系统的目的。本篇文章将会介绍一下常用的限流算法以及他们各自的特点。

我们可以使用**redis**用作**分布式**环境的限流

### 一、四种常见限流算法

#### 1、计数法(固定时间窗口限流算法)：

选定一个时间的起点，之后每当有接口请求到来，我们就将计数器加1，如果在当前时间窗口内，根据限流规则(每秒钟允许100次访问请求)，出现累加访问次数超过限流值情况，我们请拒绝后续访问请求。当进入下一个时间窗口后，计数器就清零重新计数。

**缺点：限流策略过于粗略，无法应对两个时间窗口临界时间内的突发流量**

算法实现：

redis的ttl特性完美的满足了这一需求,将时间窗口设置为key的失效时间,然后将key的值每次请求+1即可.**伪代码**实现思路:

```python
def can_pass_fixed_window(user, action, time_zone=60, times=30):
    """
    :param user: 用户唯一标识
    :param action: 用户访问的接口标识(即用户在客户端进行的动作)
    :param time_zone: 接口限制的时间段
    :param time_zone: 限制的时间段内允许多少请求通过
    """
    key = '{}:{}'.format(user, action)
    # redis_conn 表示redis连接对象
    count = redis_conn.get(key)
    if not count:
        count = 1
        redis_conn.setex(key, time_zone, count)
    if count < times:
        redis_conn.incr(key)
        return True

    return False
```

#### 2、滑动时间窗口限流算法：

在任意1s的时间窗口内，接口的请求次数都不能大于K次。

维护一个K+1的循环队列，用来记录1s内到来的请求，【当队列满时，tail指向的位置实际上是没有存储数据的，所以循环队列会浪费一个数组的存储空间】

当有新的请求到来时，我们将与这个新请求的时间间隔超过1s的请求，从队列中删除。然后我们再来看循环队列中是否有空闲位置。如果有，则把新请求存储在队列尾部，如果没有，则说明1s内的请求次数已经超过了限流值K，所以这个请求被拒绝服务。

**缺点：只能在选定时间粒度上限流，对选定时间粒度内的更加细粒度的访问频率不做限制。**

常用的更平滑的限流算法：漏桶算法和令牌桶算法。

算法实现：

```python
def can_pass_slide_window(user, action, time_zone=60, times=30):
    """
    :param user: 用户唯一标识
    :param action: 用户访问的接口标识(即用户在客户端进行的动作)
    :param time_zone: 接口限制的时间段
    :param time_zone: 限制的时间段内允许多少请求通过
    """
    key = '{}:{}'.format(user, action)
    now_ts = time.time() * 1000
    # value是什么在这里并不重要，只要保证value的唯一性即可，这里使用毫秒时间戳作为唯一值
    value = now_ts 
    # 时间窗口左边界
    old_ts = now_ts - (time_zone * 1000)
    # 记录行为
    redis_conn.zadd(key, value, now_ts)
    # 删除时间窗口之前的数据
    redis_conn.zremrangebyscore(key, 0, old_ts)
    # 获取窗口内的行为数量
    count = redis_conn.zcard(key)
    # 设置一个过期时间免得占空间
    redis_conn.expire(key, time_zone + 1)
    if not count or count < times:
        return True
    return False
```

#### 3、漏桶算法：

水(请求)先进入到漏桶里，**初始状态**桶是**空的**，漏桶以**一定的速度出水**(接口有响应速率)，当水流入速度过大会直接溢出(访问频率超过接口响应速率)，然后就拒绝请求可以看出漏桶算法能强行限制数据的传输速率。

**缺点：对于突发的流量缺乏效率。在某一个时间段流量激增,则漏桶算法处理就比较无能为力.这个时候就需要用到和他相反设计的令牌桶算法**

```python
import time


class Funnel:
    def __init__(self, capacity, leaking_rate):
        self.capacity = capacity  # 漏斗容量
        self.leaking_rate = leaking_rate  # 流出速率
        self.left_water = 0  # 漏斗剩余水量，初始为0，一开始就可以注入水(请求)
        self.leaking_ts = time.time()  # 上一次漏水时间

    def watering(self, water):
        now_ts = time.time()
        delta_ts = now_ts - self.leaking_ts
        self.left_water -= self.leaking_rate * delta_ts
        self.left_water = max(0, self.left_water)  # 剩余水不能为负数
        if self.left_water + water >= self.capacity:  # 桶满了，水（请求）不能注入了
            return False
        self.leaking_ts = now_ts
        self.left_water += water
        return True
```

问题来了，分布式的漏斗算法该如何实现？能不能使用 Redis 的基础数据结构来搞定？我们观察 Funnel 对象的几个字段，我们发现可以将 Funnel 对象的内容按字段存储到一个 hash 结构中，灌水的时候将 hash 结构的字段取出来进行逻辑运算后，再将新值回填到hash 结构中就完成了一次行为频度的检测。
但是有个问题，我们**无法保证**整个过程的**原子性**。从 hash 结构中取值，然后在内存里运算，再回填到 hash 结构，这三个过程无法原子化，意味着需要进行适当的加锁控制。而一旦加锁，就意味着会有加锁失败，加锁失败就需要选择重试或者放弃。
如果重试的话，就会导致性能下降。如果放弃的话，就会影响用户体验。同时，代码的复杂度也跟着升高很多。这真是个艰难的选择，我们该如何解决这个问题呢？
Redis 4.0 提供了一个限流 Redis 模块，它叫 redis-cell。**该模块也使用了漏斗算法，并提供了原子的限流指令**。有了这个模块，限流问题就非常简单了。
 [官网地址](https://github.com/brandur/redis-cell?_ga=2.223339035.1806735569.1597653119-1859921426.1584325709)

[Redis:限流算法](https://blog.csdn.net/wangdamingll/article/details/108084646)

#### 4、令牌桶：Google开源项目Guava中的RateLimiter使用的就是令牌桶控制算法。

系统会按恒定1/QPS时间间隔(如果QPS=100,则间隔是10ms)往桶里加入Token，如果桶已经满了就不再加了。新请求来临时,会各自拿走一个Token,如果没有Token可拿了就阻塞或者拒绝服务。

**初始状态token是满的**

**好处：允许流量一定程度的突发，因为取走token是不需要耗费时间的。**

可以方便的改变速度. 一旦需要提高速率，则按需提高放入桶中的令牌的速率. 一般会定时(比如100毫秒)往桶中增加一定数量的令牌, 有些变种算法则实时的计算应该增加的令牌的数量

伪代码

```python
# 令牌桶法，具体步骤：
# 请求来了就计算生成的令牌数，生成的速率有限制
# 如果生成的令牌太多，则丢弃令牌
# 有令牌的请求才能通过，否则拒绝
```

```python
def can_pass_token_bucket(user, action, need_token, time_zone=60, times=30):
    """
    :param user: 用户唯一标识
    :param action: 用户访问的接口标识(即用户在客户端进行的动作)
    :param time_zone: 接口限制的时间段
    :param time_zone: 限制的时间段内允许多少请求通过
    """
    # 请求来了就倒水，倒水速率有限制
    key = '{}:{}'.format(user, action)
    rate = times / time_zone # 令牌生成速度
    capacity = times # 桶容量
    tokens = redis_conn.hget(key, 'tokens') # 看桶中有多少令牌
    last_time = redis_conn.hget(key, 'last_time') # 上次令牌生成时间
    now = time.time()
    tokens = int(tokens) if tokens else capacity
    last_time = int(last_time) if last_time else now
    delta_tokens = (now - last_time) * rate # 经过一段时间后生成的令牌
    if delta_tokens > 1:
        tokens = tokens + tokens # 增加令牌
        if tokens > tokens:
            tokens = capacity
        last_time = time.time() # 记录令牌生成时间
        redis_conn.hset(key, 'last_time', last_time)

    if tokens >= need_token:
        tokens -= need_token # 请求进来了，令牌就减少
        redis_conn.hset(key, 'tokens', tokens)
        return True
    return False
```

### **二、算法对比**

**2.1 计数器 VS 滑动窗口：**

计数器算法是最简单的算法，可以看成是滑动窗口的低精度实现。滑动窗口由于需要存储多份的计数器（每一个格子存一份），所以滑动窗口在实现上需要更多的存储空间。也就是说，如果滑动窗口的精度越高，需要的存储空间就越大。

**2.2漏桶算法 VS 令牌桶算法：**

漏桶算法和令牌桶算法最明显的区别是令牌桶算法允许流量一定程度的突发。因为默认的令牌桶算法，取走token是**不需要耗费时间**的，也就是说，假设桶内有100个token时，那么可以瞬间允许100个请求通过。漏桶限制的是常量流出速率（即流出速率是一个固定常量值，比如**都是1的速率流出，而不能一次是1，下次又是2**），从而平滑突发流入速率

- 令牌桶是按照固定速率往桶中添加令牌，请求是否被处理需要看桶中令牌是否足够，当令牌数减为零时则拒绝新的请求；
- 漏桶则是按照常量固定速率流出请求，流入请求速率任意，当流入的请求数累积到漏桶容量时，则新流入的请求被拒绝；
- 令牌桶限制的是平均流入速率（允许突发请求，只要有令牌就可以处理，支持一次拿3个令牌，4个令牌），并允许一定程度突发流量；
- 漏桶限制的是常量流出速率（即流出速率是一个固定常量值，比如都是1的速率流出，而不能一次是1，下次又是2），从而平滑突发流入速率；
- 令牌桶允许一定程度的突发，而漏桶主要目的是平滑流入速率；
- 两个算法实现可以一样，但是方向是相反的，对于相同的参数得到的限流效果是一样的。

令牌桶算法由于实现简单，且允许某些流量的突发，对用户友好，所以被业界采用地较多。当然我们需要具体情况具体分析，只有最合适的算法，没有最优的算法。

### 三、适用场景

并不能说明令牌桶一定比漏洞好，她们使用场景不一样。**令牌桶**可以用来**保护自己**，主要用来对调用者频率进行限流，为的是让自己不被打垮。所以如果自己本身有处理能力的时候，如果流量突发（实际消费能力强于配置的流量限制），那么实际处理速率可以超过配置的限制。而**漏桶算法**，这是用来**保护他人**，也就是保护他所调用的系统。主要场景是，当调用的第三方系统本身没有保护机制，或者有流量限制的时候，我们的调用速度不能超过他的限制，由于我们不能更改第三方系统，所以只有在主调方控制。这个时候，即使流量突发，也必须舍弃。因为消费能力是第三方决定的。

总结起来：如果要让自己的系统不被打垮，用令牌桶。如果保证别人的系统不被打垮，用漏桶算法

引自

[常见的限流算法](https://www.cnblogs.com/wjh123/p/11442632.html)

[python+redis 实现限流](https://www.cnblogs.com/xiaozengzeng/p/12642394.html)

[水满自溢「限流算法第四把法器：漏桶算法](https://zhuanlan.zhihu.com/p/135889557)

[高并发系统限流-漏桶算法和令牌桶算法](https://www.cnblogs.com/xuwc/p/9123078.html)