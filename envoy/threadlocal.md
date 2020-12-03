# envoy threadLocal设计原理
## 设计目标
envoy 使用的是多线程架构，主线程和work线程。主线程负责配置更新，热重启等，work进程负责处理业务。不可避免的需要线程间共享一些数据，典型的例子就是配置，envoy作为一个高性能的l7代理在设计的时候就必须考虑到怎么无锁化的实现线程间数据的共享。
## 类图
![threadlocalclass](./assets/threadlocalclass.drawio.svg)
## 实现原理
ThreadLocal原理图如下:

![threadlocal](./assets/threadlocal.drawio.svg)

需要线程间共享数据的模块调用SlotAllocator分配出一个Slot，该Slot会有一个内部的index用来指示共享的数据指针(ThreadLocalObjectSharedPtr)在每个线程的ThreadLocal中存储的位置，后续main thread(在envoy中设置共享数据都是由main线程执行的，work线程只是读取共享数据)如果需要跟work thread共享数据的话直接调用slot.runOnAllThreads接口就可以将共享数据指针存储到每个TLS的内部槽位的任务提交给每个thread去执行。后续某个thread需要获取共享数据只需要调用Slot.get接口。
## 总结
ThreadLocal机制是Envoy中非常核心的一个机制，其结合C++的智能指针和libevent的事件回调设计的这套资源共享机制其原理有点类似于RCU锁，即解决了配置共享的问题又提供了一种无锁访问的方式，使用也比较简单。