# disruptor-并发编程的复杂性
yuxuan 2019-03-25

这篇文章中涉及的内容主要来自
> [Disruptor: High performance alternative to bounded queues for exchanging data between concurrent threads](http://lmax-exchange.github.io/disruptor/files/Disruptor-1.0.pdf)

# 概述
并发编程中开销最大的部分是**带争用的写操作（contended write access）**，通常通过锁或者类似策略解决。

## 锁的开销
获取锁的时候，涉及到上下文切换（“context switch”，将控制权交还给内核，内核会挂起等待锁的所有线程，内核接管线程的时候可能会做一些额外的清理工作），
上下文切换的过程中，执行上下文（“execution context”）可能会丢失数据以及指令的缓存，（由于各级缓存速度的巨大差异，）这种情形会严重影响性能。  
某些情况下，可以使用更快的用户态的锁（只有当没有资源争用的时候才有实际收益）。

## “CAS”的开销
CAS不需要上下文切换，但是CAS需要处理器做如下操作：
- `lock its instruction pipeline` 以保证原子性。
- `employ a memory barrier` 以使内存变化对其他线程可见。

使用CAS和memory barrier实现无锁算法非常复杂，并且很难验证其正确性。

作者建议的理想算法是：对单个资源，单线程写入，多线程读取。这种情况下，多核环境中的读取需要memory barrier。

## Memory Barriers

## Cache Lines

## 队列问题

## 管道与图

# 更多参考资料
- [高效内存无锁队列 Disruptor](http://www.okyes.me/2016/11/01/disruptor.html)
- [论文中文翻译](http://blog.sina.com.cn/s/blog_68ffc7a4010150yl.html)
