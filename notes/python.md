#### 1、python的gc垃圾回收

- 1.1 机制：

  - 引用计数（主）

    对象引用计数变为0，用户不可能通过任何方式使用这个对象，当垃圾回收启动时，Python扫描到这个引用计数为0的对象，就将它所占用的内存进行回收。

  - 标记清除（辅）

  - 分代回收（辅）

- 1.2 时机

  - 自动回收：当gc模块的计数器达到阀值的时候，触发自动回收
  - 手动回收：使用gc模块中的collect 方法
  - 程序退出

参考[Python垃圾回收中引用计数、标记清除、分代回收机制详解](https://www.pythonf.cn/read/26626)

[记一次面试问题——Python 垃圾回收机制](https://testerhome.com/topics/16556)

#### 2、闭包

[理解Python闭包概念](https://www.cnblogs.com/yssjun/p/9887239.html)
