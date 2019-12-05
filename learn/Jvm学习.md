## JVM内存模型划分
 - New(年轻代)
     分为Eden(80%)区和两个(Servivor  %20)幸存区 
     两个Servivor增加了对象在年轻代中的停留时间
     
     用来存放JVM刚分配的Java对象
 - Tenured(年老代)
     在年轻代中存活超过16次的对象会划入老年代中
 - Perm(永久代)
     存放类和方法的元信息

其中老年代和新生代属于堆内存,在JVM启动参数中可以指定分配内存
永久代由虚拟机直接分配内存

## 垃圾回收执行时间
 - 当年轻代内存满时,会进行一次普通GC,该GC仅仅回收年轻代(Eden)
 - 当老年代满时会触发Full GC,同时回收年轻代和老年代
 - 当永久代满时会触发Full GC,此时会卸载类和方法的元信息
 
## JVM调优的注意事项
 1. 多数的Java应用不需要在服务器上进行GC优化
 2. 多数导致GC问题的Java应用,大多数都是代码问题引起的
 3. 在应用上线前,先考虑将机器的JVM参数设置最优
 4. 尽量减少创建对象的数量
 5. 减少使用全局变量和大对象
 6. GC优化是迫不得已的手段
 7. 分析GC情况优化代码比优化GC参数用的多
 
## 基本的回收算法
 1. 引用计数法
 2. 标记-清除法
 3. 复制算法
 4. 标记-整理法
 5. 分代收集: 根据对象的不同声明周期,选择合适的收集算法


## 常用的垃圾回收器
 1. 串行处理器: 适用于数据量比较小,单处理器下并且对响应时间无要求的应用
 2. 并行处理器: (对吞吐量高要求)适用于多CPU,对响应时间无要求的中或大型应用
 3. 并发处理器: (对响应时间高要求)多CPU,对响应时间要求高的应用
 
## GC优化的目的
 1. 将转移到老年代的对象数量降低到最小
 2. 减少Full GC的执行时间
 优化方法: 
 1. 减少使用全局变量和大对象
 2. 调整新生代和老年代的大小
 3. 选择合适的GC收集器

## Java线程池的配置参数
 - corePoolSize: 核心线程数
 - maximumPoolSize: 最大线程数,超过这个数量的任务会被拒绝,可通过RejectExecutionHandler接口自定义处理方式
 - keepAliveTime: 线程活跃时间
 - workQueue: 线程工作队列,用来存放需要执行的任务 三种不同的Queue
 -- SynchronousQueue: 会为每个任务分配一个新线程
 -- LinkedBlockQueue: 无界队列,线程池仅用corePoolSize的线程处理任务,没有处理的任务将会在队列中排队(通常使用这种)
 -- ArrayBlockQueue: 有界队列,采用FIFO进行等待,吞吐量慢
 - ThreadFactory: 用于设置创建线程的工厂,可以在工厂中配置线程的优先级
 - RejectExecutionHandler: 饱和策略,需要根据场景自定义策略
 - TimeUnit: 线程活动保持的时间单位
 
常见的配置参数
 1. 堆配置
     - -Xms: 初始堆大小
     - -Mmx: 最大堆大小
     - -XX:NewSize=n: 设置年轻代大小
     - -XX:NewRatio=n: 设置年轻代和年老代的比值
     - -XX:SurvivorRatio=n: 年轻代中的Eden和两个Servivor区的比值
     - -XX:MaxPermSize=n: 设置永久代大小
 2. 收集器配置
     - -XX:+UseSerialGC: 设置串行收集器
     - -XX:+UseParallelGC: 设置并行收集器
     - -XX:+UseParalledlOldGC: 设置并行年老代收集器
     - -XX:+UseConcMarkSweepGC: 设置并发收集器
 3. 并行收集器
     - -XX:ParallelGCThreads=n: 设置并行收集器收集时使用的CPU数,即并行收集线程数
     - -XX:MaxGCPauseMillis=n: 设置并行收集最大暂停时间
     - -XX:GCTimeRatio=n: 设置垃圾回收时间占程序运行时间的百分比
 4. 并发收集器设置
     - -XX:+CMSIncrementalMode: 设置为增量模式,适用于单CPU
     - -XX:ParallelGCThreads=n: 设置并发收集器年轻代收集方式为并行收集器时,使用的CPU数
     
     
### 多线程
* CountDownLatch: 允许一个或多个线程等待其他线程完成操作
* CyclicBarrier: 让一组线程到达一个屏障时被阻塞,直到最后一个线程到达屏障时,所有被屏障拦截的线程才会继续运行
* Semaphore: 用来控制同时访问特定资源的线程数量,可以协调各个线程,以保证合理的使用公共资源
#### 线程池的三种阻塞队列:
* 基于数组的先进先出队列   有界
* 基于链表的先进先出策略   无界
* 无缓冲的等待队列        无界

#### 线程池的四种拒绝策略:
* AbortPolicy 默认  队列满了丢弃任务  抛出异常
* DiscardPolicy  队列满了丢任务  不抛异常
* DiscardOldestPolicy 删除最早进入队列的任务  再尝试加入队列
* CallerRunsPolicy  如果添加到线程池失败,主线程会自己去执行该任务



