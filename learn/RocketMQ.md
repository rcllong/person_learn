## RocketMQ 四个模块
* NameServer: 存储当前集群所有Broker信息 Topic与Broker的对应关系
* Broker:     集群最核心模块,主要负责Topic消息存储,消费者的消费进度管理
* Producer:   消息生产者,每个生产者都有一个ID,多个生产者实例可以共用同一个ID,同一个ID下所有实例组成一个生产者集群
* Consumer:   消息消费者,每个订阅者也有一个ID,多个消费者实例可以共用同一个ID,同一个ID下所有实例组成一个消费者集群

# 如何保证消息不丢失
## 1.Producer保证消息不丢失
* 默认情况下,可以通过同步的方式阻塞式的发送,check SendStatus,状态是OK,表示消息一定成功的投递到了Broker
状态超时或失败,则会触发默认的2次重试.此方法的发送结果,可能Broker存储成功了,也可能没成功

* 采取事务消息的投递方式,并不能保证消息100%投递成功到了Broker,但是如果消息发送Ack失败的话,此消息会存储在CommitLog当中
但是对ConsumerQueue是不可见的.可以在日志中查到这条异常的消息,严格意义上讲,消息并没有完全丢失

* RockerMQ支持 日志的索引,如果一条消息发送之后超时,也可以通过查询日志的API,来check是否在Broker存储成功

## 2.Broker保证消息不丢失
* 消息支持持久化到CommitLog里面,即使宕机后重启,未消费的消息也是可以加载出来的
* Broker自身支持同步刷盘.异步刷盘的策略,可以保证接收到的消息一定存储在本地的内存中
* Broker集群支持1主N从的策略,支持同步复制和异步复制的方式,同步复制可以保证即使Master磁盘崩溃,消息仍然不会丢失

## 3.Consumer保证消息不丢失
* 消费者可以根据自身的策略批量pull消息
* Consumer自身维护一个持久化的offset(对应MessageQueue里面的min offset),标记已经成功消费或者已经成功发回到Broker的消息下标
* 如果Consumer消费失败,那么它会把这个消息发回给Broker,发回成功后,更新自己的offset
* 如果Consumer消费失败,发回给Broker时,Broker挂掉了,那么Consumer会定时重试这个操作
* 如果Consumer和Broker一起挂了,消息也不会丢失,因为Consumer里面的offset是定时持久化的,重启之后,继续拉去offset之前的消息到本地

## 其他相关

* 生产者端配置的发送失败重试次数 默认为2
* 消费者消费消息后,需要给Broker返回消费状态
* 在消费者端,消息重试16次,如果仍然失败,消息丢失
* 广播消费方式能保证一条消息至少被消费一次,但消费失败后不做重试操作
* 当重试次数超过所有延迟级别之后,消息会进入死信
* 进入死信之后的消息不会在投递了,可以通过接口查询当前RocketMQ中死信队列的消息

### 死信消息的特性
* 不会再被消费者正常消费
* 有效期与正常消息相同,3天内会被自动删除


### 顺序消息
* Consumer消费时通过一个分区只能有一个线程消费的方式来保证消息顺序
#### Producer端确保消息顺序唯一要做的事情就是将消息路由到特定的分区,在RocketMQ中,通过MessageQueueSelector来实现分区的选择:
1. PullMessageService单线程的从Broker获取消息
2. PullMessageService将消息添加到ProcessQueue中(ProcessQueue是一个消息的缓存),之后提交一个消费任务到ConsumerMessageOrderService
3. ConsumerMessageOrderService多线程执行,每个线程在消费信息时需要拿到MessageQueue的锁
4. 拿到锁之后从ProcessQueue中获取消息

### 保证消费顺序的核心思想是:
* 获取到消息后添加到ProcessQueue中,单线程执行,所以ProcessQueue中的消息是顺序的
* 提交的消费任务是"对某个MQ进行一次消费",这个消费请求是从ProcessQueue中获取消息消费,所以也是顺序的
* 顺序消息需要Producer和Consumer都保证顺序,Producer需要保证消息被路由到正确的分区,消息需要保证每个分区数据只有一个线程消息


