# 初始化管理模块init

## 设计目标
init-manage 是envoy用来管理target初始化的模块，先来看下envoy在注释中对这个模块的描述：
```c++
/**
 * Init::Manager coordinates initialization of one or more "targets." A typical flow would be:
 *
 *   - One or more initialization targets are registered with a manager using `add`.
 *   - The manager is told to `initialize` all its targets, given a Watcher to notify when all
 *     registered targets are initialized.
 *   - Each target will initialize, either immediately or asynchronously, and will signal
 *     `ready` to the manager when initialized.
 *   - When all targets are initialized, the manager signals `ready` to the watcher it was given
 *     previously.
 *
 * Since there are several entities involved in this flow -- the owner of the manager, the targets
 * registered with the manager, and the manager itself -- it may be difficult or impossible in some
 * cases to guarantee that their lifetimes line up correctly to avoid use-after-free errors. The
 * interface design here in Init allows implementations to avoid the issue:
 *
 *   - A Target can only be initialized via a TargetHandle, which acts as a weak reference.
 *     Attempting to initialize a destroyed Target via its handle has no ill effects.
 *   - Likewise, a Watcher can only be notified that initialization was complete via a
 *     WatcherHandle, which acts as a weak reference as well.
 *
 * See target.h and watcher.h, as well as implementation in source/common/init for details.
 */
```

上面的注释描述了init:Manager的使用流程是:

* 一个或多个target调用add接口注册到init:Manager中

* 调用init:Manager的initialize接口，传入一个watcher参数，当所有的target初始化完成后会通知watcher

* 每个target都会被初始化，不管是以立即的方式还是延后的方式，每个target初始化后都会发送ready消息给manager

* 所有的target被初始化完成后会通知刚才initialize传入的watcher参数。

因为在上面的使用流程中牵扯到很多的实体：
- Manager的拥有者
- 注册到Manager中的target
- Manager本身

在某些情况下可能很难或不可能保证它们的生命周期正确排列以避免导致释放后错误。 Manager通过以下设计来解决这个问题:

- 引入的TargetHandle, Target只能被TargetHandle来初始化，以弱引用的凡是解决Target被提前释放的问题
- 同样的引入WatcherHandle，Watcher只能被WatcherHandle通知，也是通过弱引用的方式来解决Watcher被提前释放的问题



## 实现原理

![](C:\Users\lee\Documents\note\envoy\assets\init-manager.drawio.png)
