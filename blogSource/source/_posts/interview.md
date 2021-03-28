---
title: 一顿乱写
date: 2020-05-20 08:48:14
top: true
tags: 
  - interview
categories:
  - [java]
  - [interview]
---



## 一句话

### 说一下你重构代码都做了什么?

> 首先是两层改成三层, 把controller 的代码尽量迁移到 service 层. 然后将请求风格和响应数据结构统一. 还有就是处理全局异常, 最后对某些重复代码封装成工具类. 另外还会根据实际业务场景使用一些设计模式, 提高代码可扩展性, 降低代码之间的耦合性.

### 什么是 JVM?

>  JVM 就是由编译器, 类加载器, 执行引擎, 运行时数据区组成. 其中数据区包含 堆,栈,本地方法栈, 方法区和程序计数器(PC 寄存器), 其中栈是由局部变量表, 操作数栈, 动态链接, 返回地址组成的.

### 你是怎么对 jvm 垃圾回收进行优化的?

> 根据服务器的配置, 调整青年代和老年代的内存大小及比例, 在回收频率和回收速度上做取舍, 使用 G1 垃圾回收期控制 STW 停顿时间, 提高吞吐量.

### 说说 MySQL 优化

> 首先是 SQL 查询优化, 通过对联表字段, 查询条件, 分组字段, 排序字段进行综合分析, 根据最左原则建立一个或多个复合索引, 然后使用 explain 分析SQL执行计划, 判断索引使用情况, 根据分析结果进一步改进索引.
>
> 然后是对于数据量大的表, 考虑垂直或水平分表, 读多写少的情况, 可以一主多从集群. 另外对于一些统计类的查询, 可以用定时任务将统计结果存储起来, 而非实时查询.

### 你用 Redis 做了什么?

> 将高访问的首页商品列表缓存到 redis 中, 避免数据库瓶颈, 提高响应速度. 
>
> 商品同步问题: 定时任务刷新. 或修改商品时更新, 缓存设置失效时间, 失效后自动读取数据库.
>
> 将购物车数据存放到 redis, 提高购物车交互体验(加快响应速度).

### 你使用消息队列做了什么?

> 解耦: 如下单系统调用库存系统减库存, 若调用时库存系统挂了或出错了, 下单系统还需要做重试处理, 异常处理, 此时可将减库存请求放到消息队列中, 库存系统读取消息进行处理, 若出错则放回消息队列重试. 这样即使代码 bug 导致一直不成功也可在升级后自动重试, 无需人工干预. 另微信支付回调也可如此处理.
>
> 削峰: 如秒杀瞬间请求过高, 可将请求放到消息队列中, 另一端缓慢消费, 可防止系统卡住.
>
> 异步: 比如下单后发送下单通知, 有短信通知, 微信公众号通知等, 一个一个发送会导致下单这个请求响应很慢, 因此可以将几个通知做成一个消息, 放到消息队列, 由另一处代码异步执行.

### 你使用线程池做了什么?

> 将线程池封装到一个工具类中, 工具类再做成单例模式. 这样使用到多线程的地方都可以使用同一个公共线程池, 减少线程对象创建销毁. 提高线程的利用率.
>
> 一些地方异步操作, 拦截器添加请求日志时异步添加.

### 你在代码中使用了哪些设计模式?

> 单例模式, 静态工厂模式, 模板方法, 观察者, 装饰者, 策略模式, 状态模式, 职责链模式.
>
> 观察者: 监听商品信息更新, 根据佣金变化幅度决定是否删除, 根据佣金变化和价格变化幅度决定是否通知用户收藏商品变化.
>
> 策略模式: 订单不同类型, 对应的商品源不同, 查询数据方式不同, 因此使用策略模式, 便于新增类型的扩展.
>
> 状态模式: 红包状态的变化, 可以做成状态模式, 使得红包新增状态时扩展更简单.

## 架构

### 重构

```
重复代码重构, 抽象出工具类, 返回值/自定义异常 整理重构, 统一请求风格
使用状态模式/策略模式优化 if/else
使用工厂模式统一管理需要的实例对象, 如工具类, 邮件服务等
封装通用 CRUD 接口及实现, 减少 Dao 层代码

模块的拆分, 数据库分库分表, 微服务拆分
```

### 集群

#### MySQL 集群

> MySQL 默认支持主从架构集群, 可配合 mycat 实现读写分离.

#### Redis 集群

> Redis Cluster, Codis

#### Tomcat 集群

> tomcat 集群一般需要考虑 session 共享, 可通过 redis 实现 session 共享.

### 分布式

#### SpringCloud

> Spring Cloud 是一套分布式开发的解决方案, 集合了分布式调用, 链路追踪, 降级处理, 服务注册发现.

#### Dubbo

> Dubbo 是一个分布式 RPC 调用框架, 底层使用 netty 框架.

### 运维

#### Docker

> docker 是一个容器, 提供了标准化的接口, 可用于快速构建部署环境, 简化部署流程

#### docker-compose

> docker-compose 使用 yml 文件描述容器间的关系以及容器的配置, 可用于快速构建复杂的运行环境.

#### K8s

> K8s 是一个根据容器快速搭建和管理集群的工具.



## JVM

### JVM 内存模型

> 每个线程有自己的内存区域, 多线程之间通信主要通过共享内存来实现.

- 有序性: 在 CPU 执行指令时, 可能会对非 happens-before 指令进行重排, 优化执行效率. 在单线程情况, 往往不会产生问题, 但涉及多线程时, 可能导致 bug.
- 可见性: 一个线程修改了一个共享变量, 另一个线程不会知道这个改变, 这就是不可见, 要确保可见性, 一般使用 volatile 关键词, 当然, 加锁也可以.
- 原子性: 即对于某代码, 实际执行时会分为好几个原子指令, 确保原子性必须加锁 (如synchronized) 处理

```
happens-before:
读后写
写后写
锁后解锁
可传递性
```

### 多线程

> 线程是一个进程中的不同执行路径, 一个进程至少有一个主线程.
>
> 进程是一个程序的抽象, 一个程序运行后一般为一个进程.

线程状态:

```
1.New (新建)
2.Runnable (就绪)
3.Running (运行中)
4.Blocked (阻塞)
5.WAITING (等待)
6.TIMED_WAITING (超时等待)
7.Dead (死亡)

[t:thread对象, obj: 同步块中的对象]
New: new Thread()
Runnable: t.start(), t.yield()
Running: after t.start() and cpu run it
Blocked: when enter synchronized block
WAITING: obj.wait(), t.join(), LockSupport.park()
TIMED_WAITING: Thread.sleep(x), obj.wait(x), t.join(x)
Dead: when t.run() is over
```



### JVM 对象结构

```
对象头:
	Mark Word(hash, 锁状态, 分代年龄)
	类型指针
	[数组长度]
实例数据
对齐
```



### 垃圾回收

```
标记算法:
	1.引用计数法
	2.可达性分析算法(根搜索)(根对象: 栈中的对象, 静态属性引用对象, 常量引用对象)
回收算法:
	1.标记-清除算法
	2.标记-整理算法
	3.标记-复制算法
	4.分代算法( Eden 区(复制算法)--> Survivor 区(缓存, 复制算法) --> Old 区(标记-整理)

回收器: (前 3 个 Young GC 使用, 后面的 Full GC 使用)
	1.Serial (串行, 复制算法)
	2.ParNew (多线程, 复制算法)
	3.Parallel Scavenge (多线程, 改进版, 可控制吞吐量)
	4.Serial Old(单线程, 标记-整理)
	5.Parallel Old (多线程, 控制吞吐量, 标记-整理算法)
	6.CMS (多线程, 低停顿, 标记-清除算法) : 初始(STW)-并发-重新(STW)-清除
	7.G1 (多线程, CMS 升级版, 标记-整理算法): 初始(STW)-并发-最终(STW)-筛选清除(可控制停顿时间)
```



## MySQL

### SQL 优化

#### 索引原理

> MySQL 索引一般选择 B+树做为数据结构存储. B+ 树的优点是, 对文件IO的访问次数控制在 3 次, 保证速度的同时, 能存储千万行数据.

#### 索引

```
1.对常用列添加索引, 视具体情况选择单一索引或复合索引(一般为复合)
2.通过 Explain 语句分析执行计划, 将 type 提升到至少 index 级别.
3.通过 Explain 语句分析执行计划, 将 extra 中 Using filesort消除(排序列加索引), Using join buffer消除 (通过给关联表的关联列加索引), Using temporary (一般通过分组列加索引), Using where(根据最左原则对条件列加复合索引)
```



### 事务

```
ACID:
A: 原子性, 多个操作要么都做, 要么都不做
C: 一致性, 数据库文件的状态必须从一个一致性状态到另一个一致性状态.
I: 隔离性, 事物之间相互隔离, 互不影响.
D: 持续性, 一个事务一但提交, 则对数据库的改变是永久的.
```



### 数据库隔离级别

```
1.读未提交: 可读取其他未提交事务的执行结果(如更新了某个字段), 可能会造成读取错误的数据(未提交的事务回滚了), 造成脏读.
2.读已提交: 可读取其他已提交事务的执行结果, 2次读取数据还是可能不一致(其他事务又提交了), 造成不可重复读.
3.可重复读: 确保同一事务内多次读取数据时, 会看到相同的数据. 但可能造成幻读, 如批量修改登录密码后, 另一个事务新增了一条记录, 导致新纪录未修改.
4.串行化: 事务串行化执行, 效率低.
```

#### MySQL 默认隔离级别

`可重读读`



### 数据库锁

#### 锁原理

```
行锁: 分为排它锁(X) 和共享锁(S). 即写锁和读锁.
表锁: 分为元数据锁(MDL)和表锁.
```

#### 锁触发方式

```
行锁: 隐式(条件带有索引则锁对应列, 不带索引则锁全部行, RR 总会带有 GAP 锁, RC 不会), 显式(使用 for update, lock in share mode)
表锁: 隐式(对整个表不带条件进行增删改, 或任何 DDL 操作) 显示(使用 for update, lock in share mode)
```



## 源码和框架

### `ReentrantLock`

#### 加锁流程 `lock()`

```java
1) acquire(): 尝试获取一个许可证, 获取成功则直接返回(lock结束), 获取失败则需要排队
2) tryAcquire(): 判断当前许可证数量(state), 若为0则尝试获取
     分公平和非公平, 公平锁会判断 hasQueuedPredecessors, 非公平则直接抢 compareAndSetState
     若不为0, 则判断持有锁的人是否为我本身, 是则增加当前许可证数量, 返回true获取成功
     不是则 返回 false, 获取失败(将排队).
3) addWaiter(): AQS 队列尾部添加一个 Node(waiter=X[独占锁]), 若 tail 不存在, 则先初始化一个 空head[空指不代表任何线程] 后再加入队列
4) acquireQueued: 进入队列的节点, 尝试获取许可证, 失败则 park()
     先判断node的上一个节点是否为 head 节点, 若是, 则要尝试获取一次许可证(因为这说明上一个线程已经在执行过程中了, 也许已经走完了unlock() 方法(即已经运行过唤醒队列下一位的代码了,而因为你那时还不在队列中或没进入睡眠中, 唤醒代码是无意义的), 而你则刚加入队列, 如果你此时直接park()去等待唤醒, 则根本无人唤醒你, 同理你的下一个节点也就等不到你去唤醒它.)
     如果不是, 设置了上一个节点的 waitStatus 为 SINGLE 后, 自己睡眠 park(), 等待唤醒

唤醒后:  
5) 判断上一个节点是不是 head, 一般来说是(因为unkock唤醒的一般就是head.next)
     如果不是则进入 shouldParkAfterFailedAcquire: 将队列中一些已取消的节点从队列中删除, 重新设置节点的prev
     因为是for循环, 所以又会再回来判断, 这时应该是head了, 尝试获取许可证, 2种可能, 非公平时被刚lock的人抢了(概率较小吧), 另一种就是获取成功
     获取成功后, 把原head节点删掉, 自己设为head节点(head象征一个拿到许可证的节点,除队列第一次初始化), 然后返回到acquire(), 中途没有线程被打断就正常出方法, lock结束

总结:
得到方式1: acquire 时 state = 0, 且抢到了.
得到方式2: 没抢到或不让抢(公平锁), 进入队列等上一个来唤醒我, 上一个等上上个来唤醒他, 上上个等上上上个唤醒....
    unlock 唤醒队列第二个非取消的线程并删除队列第一个元素 [其他元素移位]
    这样第二个线程就可以唤醒非取消的第三个线程[相对而言的第三个,实际上唤醒时还是第二个, 只是唤醒后会删除第一个, 所以第三变第二]
```

#### 解锁流程 `unlock()`

```
1) release(): 释放一个许可证, 并根据当前许可证数量是否为0 判断是否可以唤醒下一个节点
2) tryRelease(): 释放一个许可证, 判断线程是否正确(是不是当前独占锁), 许可证减一
	当前许可证数量是否为 0 返回是否可以唤醒队列的 bool 标识.
3) unparkSuccessor(): 唤醒队列中除head外第一个处于阻塞(非取消)的节点(查找方式, 先看next, next状态不对则从后往前找最前的非取消的节点, 因为next如果为null, 无法找null的next).
4) 唤醒后, 会将head设置为唤醒的节点, 以此达到下次唤醒下一个的目的.

总结:
唤醒的逻辑就是将排队的所有节点挨个唤醒, 而节点被唤醒后又会出队列; 所以代码将出队列和唤醒逻辑一起做, 先唤醒下一个, 下一个负责把前一个移出队列. 然后唤醒自己的下一个, 以此类推, 就实现了唤醒和出队列的操作.
```

### `ReentrantReadWriteLock`

```java
读锁
	获取锁 tryAcquireShared(), 若当前没有写锁存在, 则 state + 1个读单位, 然后返回获取成功. 防止返回获取失败, 进入队列休眠.
	释放锁 tryReleaseShared(), state - 1个读单位, 然后根据 state = 0 判断返回是否可以唤醒队列.
	
写锁
	获取锁 tryAcquire(), 若存在读锁, 则失败, 若存在写锁, 判断是否重入获取, 是则返回获取成功. 否则失败; 失败就意味着加队列,休眠.
    释放锁 tryRelease(), state - 1, 判断 state 中写锁数量是否为0, 是则可以唤醒队列. 否则代表这时一个可重入锁的释放逻辑.
   
总结: 读写锁也好, 可重入锁也好, CountDownLatch 等工具类也好, 都是对 state 操作为多, 或者说, 实现了 AQS的它们, 只负责操作 state, 而队列, 唤醒, 都交给 AQS 来处理.
```



### `CountDownLatch`

#### `countDown()`

```
1) state 数量减一, 然后判断 state 数量是否为0, 若是则唤醒等待队列的线程. 
```

#### `await()`

```
1) new 之后, state 数量大于0, 所以会进入等待队列, 然后线程会进入休眠.
2) 等待countDown释放锁, 释放到许可证为0时, 唤醒等待队列的线程.
```

> 总结:
>
> 利用了加共享锁进入队列等待特性实现 `await()`
>
> 释放共享锁减少许可证数量且唤醒队列中的等待的线程 实现 `countDown()`

### `Semaphore`

```
与 CountDownLatch 相反, 初始数量一般为 0, acquire() 时判断是否有许可证, 有则成功, 无则队列休眠
而 release 则是添加一个许可证, 添加后总是唤醒队列.
```

### `CycleBarrier`

```
含义: 凑足一定个数线程, 然后批量唤醒.
await(): 利用 ReenrantLock 的 lock 和 condition 的 await 进入休眠
当凑足后，用condition 的 singleAll 唤醒所有 await 的线程.
```





### `HashMap`

#### 请简述 `HashMap` 的底层数据结构

```
1. 使用了数组加链表, 以数组为主, 链表加红黑树为补充的数据结构来存储键值对.
2. 当键发送冲突(碰撞)时, 数据将串成链表存于数组中, 当链表长度超过指定值(默认 8)时, 链表转成红黑树, 当红黑树长度小于指定值时(默认 6), 则又转成链表
```

#### 为什么 `HashMap` 的初始容量以及扩容后的容量均为 2 的指数幂

> 因为计算机做运算时, 取模运算速度远远慢于位运算, 而若容量始终为 2 的指数幂, 则根据 hash 获取数组下标时只需要 使用 ` (数组长度-1) & hash 值  ` 即可确定数组下标, 与取模得到的下标一样可靠.
>
> 而扩容后后, 因为需要进行 rehash 运算来确定 数据的新下标, 多次进行取数组下标则更能体现位运算的优势.

#### 为什么 `HashMap` 的加载因子是 0.75 (3/4)

```
使用排除法:
1.若加载因子为 1. 则每次 HashMap 满了才进行扩容, 必将有更高的几率触发 hash 碰撞导致数组下标一致需要转成链表或红黑树, 导致读取和更新速度降低.
2.若加载因子为 0.5. 则每次 HashMap 都有一半容量剩余, 空间大大浪费, 对内存开销太大. 容易引发 OOM 事故.
3.0.5-1 之间那么多可能, 选哪个都行, 但作为 HashMap 的默认值, 选中间的 0.75, 走中庸之路, 也是解释的通的.
```



#### 为什么 `HashMap` 1.8 扩容无需 rehash

```
1. 因为1.8的获取 hash 值的算法优化了. 无需一个 hashSeed 进行辅助运算 (主因)
2. 由于 hash 值不变, 原链表中的所有节点只有 2 种可能:
	一是 hash 值高于原数组长度, 则属于高位, 这些高位的节点, 新的下标一定是 (当前下标 + 旧数组长度). 
	另一种是 hash 值低于原数组长度, 属于低位, 这些节点的下标无需重新计算, 必然与当前下标一致
	(不信自己那几个示例数据用画出完整二进制计算一下)(神奇的位运算)
3. 重新计算下标时, 根据第 2 点可知, 其下标大小一定不高于(当前下标+旧数组长度), 即下一次循环的下标必然比上一次循环的下标要高, 所以 1.8 源码 resize 进行高低位分组然后转移数据时, 无需担心下一次循环会将刚刚放到新数组的值覆盖(下标相同则会覆盖)
4. 1.8 的 resize 优化了算法, 保持了原有的链表顺序(不知道有啥用)
```

> 总得来说, 1.8 优化了 hash 算法, 使 `hashcode` 的高 16 位与 低 16 位进行异或运算, 降低了碰撞率
>
> 而 resize 算法也优化链表节点的迁移, 避免了 1.7 的链环产生
>
> 最大的区别就是, 1.7 没有将二进制的神奇发挥到极致, 依然像普通 java 程序一般逻辑. 而 1.8 则充分利用了二进制的优点(也充分的让人头晕), 提高了 `HashMap` 的效率.

#### 为什么 `HashMap` 从链表达到 8 个时转成红黑树, 达到 6 个时转回链表?

```
1.根据 Poisson distribution 定律, 凑齐8个节点碰撞到同一个下标, 组成长度为 8 的链表概率极低, 约为 0.00000006, 而超过 8 个的几率则更低, 大约为千万分之一. 所以将阈值设置为 8, 因为这种概率极低. 因此可以减少链表转红黑树的, 提高增删改效率.
2.若达到 7 个时转回链表, 则可能会导致HashMap 不停的在链表和红黑树之间转换, 所以阈值设置为 6, 可起到缓冲效果.
```



### Spring

#### 关键类解析

```
ApplicationContext
	是一个只读的 bean 容器
	可以加载解析配置文件(如xml)
	可以发布事件和注册监听
	具有国际化消息处理能力
ConfigurableApplicationContext
	是一个具有可配置能力的 容器(可设置各个参数, 如id, 父容器)
	具有容器生命周期概念, 如启动,停止,关闭.
AbstractApplicationContext
	模板方法模式的抽象类, 定义了容器的模板(refresh方法), 但由具体的子类实现部分方法
	管理Bean和BeanFactory的PostProcessor
	管理事件的监听和处理
AbstractRefreshableApplicationContext
	为可重复刷新的容器提供基类
	加入了BeanFactory的管理(创建/关闭等)
AbstractRefreshableConfigApplicationContext
	加入了configLocation字段, 用于某些容器初始化BeanFactory和Bean
AbstractXmlApplicationContext
	定义了读取xml配置文件来加载BeanFactory的代码, 使得子类只需提供配置文件地址或Resource
ClassPathXmlApplicationContext
	继承基类, 提供配置文件地址的构造方法, 调用refresh加载BeanFactory
						
BeanFactoryPostProcessor
	用于给BeanFactory添加插件式功能, 如配置文件解析 ${} 占位符
	如ConfigurationClassPostProcessor 将@Configuration类下的带@Bean的method返回值注册到beanDefinitions 中

BeanPostProcessor
	用于给bean添加功能, 如ApplicationContextAware的自动注入就是如此实现的

```

#### 容器初始化流程

```
1) 从XmlClassPathApplicationContext构造方法中进入 refresh 方法
2) 先设置容器状态
3) 调用子类初始化 BeanFactory
4) 设置BeanFactory 一些属性,添加一些内置的PostProcessor 注册一些 environment 相关的bean
5) 子类设置一些内置的PostProcessor
6) 扫描添加并执行容器内的 BeanFactoryPostProcessor
7) 扫描容器内的 BeanPostProcessor 并注册
8) 初始化国际化消息处理器
9) 初始化事件广播处理器
10) 执行子类的 refresh 逻辑
11) 扫描容器内的 ApplicationEvent (指实现类) 并注册到事件广播处理器
12) 完成BeanFactory的初始化, 并加载一些单例对象(设置了急于加载的bean)
13) 初始化LifcycleProcessor, 调用onRefresh方法, 发布 ContextRefreshedEvent 事件.
14) 清除一些缓存(如反射缓存, 注解等)
```

#### 某些实现原理

##### 实现 `ApplicationContextAware` 为何会自动注入 `applicationContext`?

```java
1) 首先 AbstractApplicationContext#prepareBeanFactory 会添加一个ApplicationContextAwareProcessor
2) 这个 beanPostProcessor 负责在bean初始化之前注入context对象.
3) 这个 beanPostProcessor 的执行时机是在 doCreateBean 中的 postProcessBeforeInitialization()
```

##### 实现 `ApplicationListener` 为何会在事件触发时自动执行我们实现的方法?

```java
1) 在 AbstractApplicationContext#registerListeners() 中扫描容器内所有相关实现类加入到事件监听者集合中
2) 然后在publishEvent时，遍历事件监听者集合调用bean的方法即可。观察者模式！
3) 另外也用了BeanPostProcessor去实现, 叫 ApplicationListenerDetector, 加入时机同1, 执行时机同1.
4) 至于为何使用2种机制，与多例有关吧！(scope="prototype")
```

##### 实现`Order`接口或注解时如何自动排序的?

```bash
1) 比如说 BeanPostProcesser, 容器扫描后, 会像对bean集合排序, 再遍历执行.
2) 详细过程见 PostProcessorRegistrationDelegate#sortPostProcessors()
```

##### 单例对象如何实现循环依赖注入？

```bash
1) 首先， 设定对象A，B， A 持有 B, B 持有A， 构成循环
2) 此时程序调用getBean获取A，则在 doCreateBean 中 创建后将bean缓存到 singletonFactories 中
3) 然后设置属性B, 解析属性, 需要获取B对象
4) 获取B, 则执行doCreateBean 后执行解析属性, 需要获取 A对象 (又一次)
5) 获取A, 进入 doGetBean 中的 getSingleton, 此时判断singletonFactories中有A, 则可以直接取出A
6) 获得A后, 即可完成B的属性赋值, 然后会完成B的创建.
7) B创建完后, A就能获得B, 则A也完成了属性赋值, 最后完成创建A.
8) 到此, 返回即可.

> 总结: 首次获取A, 创建A对象后缓存一个存储A对象的 ObjectFactory 实例, 再解析属性时触发 getBean(B), 同理也会做缓存, 然后也解析属性, 触发getBean(A), 第二次获取A, 进入另一个逻辑, 返回 ObjectFactory 实例中存储的对象A, 即可完成getBean(A), 然后完成getBean(B), 再完成外层的getBean(A).  
```

TIPS

>观察源码, 发现有2个缓存, 一个是 `singletonFactories`, 另一个是 `earlySingletonObjects`.
>其中`earlySingletonObjects`的管理都在 `getSingleton` 方法中做, 而 `singletonFactories` 则在`doCreateBean`中加入, 在 `getSingleton` 中删除(有`earlySingletonObjects`后就可以删除了).
>虽然有2个缓存, 但如果你的bean没有使用`BeanFactory`创建, 则其实一个缓存也足够了
>(因为这样的话 `singletonFactories` 每次创建返回的都是同一个, 因为此时 `singletonFactories`存的只是代码包装的一个内部类, 而非用户自定义的.)



##### `InstantiationAwareBeanPostProcessor`等一些特殊`BeanProcessor`的扩展方法是何时自动调用的?

```java
1) 首先 getBeanPostProcessorCache 获取一些特殊的BeanPostProcessor
2) 如 InstantiationAwareBeanPostProcessor/SmartInstantiationAwareBeanPostProcessor
3) 然后 createBean时, 会在正确的时机使用到这些特殊的 PostProcessor, 取出来, 然后执行对应方法
4) 具体何时可以查看 getBeanPostProcessorCache() 的调用位置一一查看.
```

#### 注解的实现

```java
1) 首先是指定包名或指定类名
	如指定包名则scan时会执行, 如指定类名则在构造方法初始化 reader 时执行
2) 无论哪种, 最终都会走一段代码 AnnotationConfigUtils#registerAnnotationConfigProcessors()
3) 这段代码会添加一些 BeanFactoryPostProcessor
	如 ConfigurationClassPostProcessor 负责解析 @Configuration/@Import/@Bean 注解
    	postProcessBeanDefinitionRegistry() 中 parse 所有的带以上注解的 beanDefinitions
    	ConfigurationClassParser 将注解信息解析保存，
    	然后由 ConfigurationClassBeanDefinitionReader 负责注册到容器。
	如 AutowiredAnnotationBeanPostProcessor 负责解析 @Autowired/@Value 注解
    如 CommonAnnotationBeanPostProcessor 负责解析 @Resource 注解
    解析放在 postProcessProperties() 方法中， 先扫描bean的字段和方法， 然后一一调用方法和为字段注入值
4) 之后, 他会将扫描的类放到 beanDefinitions 中(或指定的类注册进去)
5) BeanFactory加载完毕后, 回到AbstractApplicationContext的refresh逻辑
	如会执行 postProcessBeanFactory(), 调用前面加入的ConfigurationClassPostProcessor
	然后会添加更多的类到容器中.
    
注意事项：
    @Configuration 和 @Component的区别？
    观察发现，即使使用@Component 其下带 @Bean 的方法依然可以注入到容器中。所以似乎两者没有区别？
    仔细查看源码和资料后，发现 postProcessBeanFactory() 方法在 processConfigBeanDefinitions() 后还会调用 enhanceConfigurationClasses()
    而在这个方法中, 对前面解析了class 是 CONFIGURATION_CLASS_FULL (即代表@Configuration)的类
    会生成一个 cglib 的代理, 这样获取@Bean注解的方法的bean时,不会每次调用方法new 一个, 而是有缓存.
```

#### AOP 流程

```java
1) 使用 @EnableAspectJAutoProxy
2) @EnableAspectJAutoProxy 中使用了 @Import(AspectJAutoProxyRegistrar.class)
3) ConfigurationClassPostProcessor 会解析@Import, 进入 registerBeanDefinitions() 中
4) registerBeanDefinitions() 中添加了 AnnotationAwareAspectJAutoProxyCreator 到容器中
5) AnnotationAwareAspectJAutoProxyCreator 本质上时一个 BeanPostProcessor
6) 因此在 createBean 时, 会被自动调用. 其中 postProcessAfterInitialization() 负责创建代理对象
7) 而 getAdvicesAndAdvisorsForBean() 则负责查找对应的增强. 然后会调用子类的findCandidateAdvisors
8) 如 AnnotationAwareAspectJAutoProxyCreator#findCandidateAdvisors() 负责注解编写增强@Before/@After等
9) 简单说下逻辑, 就是查找容器所有类, 判断这个类有没有 @Aspect 注解, 然后先找出所有Pointcut
	再遍历所有方法, 找出方法上带有@Before等注解且有关联的Pointcut的方法,
    然后使用这个方法和关联的Pointcut 来new 一个Advisor, 加入到Advisor集合中, 遍历结束后返回即可.
10) 查找到所有的增强后, 再比较Pointcut表达式是否匹配当前的bean, 如可以则加入.
11) 根据找到的Advisor集合, 创建一个带配置(advisor集合等)的代理对象, 代理对象执行方法前
12) 会先根据配置中的advisor集合生成一个执行链, 然后在拦截代理方法处调用. 执行链会负责执行通知.
13) 不同的通知由不同的适配器执行.
```

#### Spring 事务实现

```java
0) 事务是由AOP实现的, 所以需要找到对应的Pointcut 和 Advisor
1) 打开了 @EnableTransactionManagement 注解
2) 然后@Import 了 TransactionManagementConfigurationSelector
3) 之后导入了 ProxyTransactionManagementConfiguration 到容器中
4) ProxyTransactionManagementConfiguration 带有 @Configuration
5) @Bean 注入了一个通用的Advisor: BeanFactoryTransactionAttributeSourceAdvisor
6) 这个Advisor的 Pointcut 是由 TransactionAttributeSourcePointcut 实现的
	实现逻辑是 TransactionAttributeSourcePointcut 的 matches()
    这个方法调用了 getTransactionAttributeSource() 获取 AnnotationTransactionAttributeSource
    然后通过 getTransactionAttribute() 调用了 findTransactionAttribute()
    最终使用SpringTransactionAnnotationParser 类判断方法是否有@Transactional注解
    并解析注解信息然后返回. 另外这个方法还可以获取@Transactional注解的信息, 而这里只用于判断是否需要拦截这个方法.
7) TransactionInterceptor 是一个Advisor
    也可以通过AnnotationTransactionAttributeSource获取@Transactional注解上的信息
    然后在invoke中, 拦截方法, 打开事务, 在执行完方法后, 提交事务, 报错时回滚事务
    这个 Advisor 不同于传统的前置/后置, 而是更具体的 MethodInterceptor.
```



### Spring Boot 源码

#### 简化了多少操作？

```java
1.MultipartAutoConfiguration
	添加了一个bean StandardServletMultipartResolver, 并设置上传文件大小等属性
	使得Spring MVC中的 DispatcherServlet 可以获取 multipartResolver, 处理文件上传.
    若不是Spring Boot, 需要在xml或注解中手动添加一个 这样的bean才能处理文件上传, 而且配置文件还要自己读取.

2.MailSenderAutoConfiguration
    添加了一个 JavaMailSenderImpl 的bean到容器中, 并设置邮箱服务器/账号/密码等属性
    若不是Spring Boot, 需要在xml中配置一个bean即配置他的属性(邮箱配置)

3.TransactionAutoConfiguration
    在内部类添加 @EnableTransactionManagement，剩下的参考上面的 Spring 事务实现
    
4.WebMvcAutoConfiguration
    添加了 RequestMappingHandlerAdapter/RequestMappingHandlerMapping
    添加了 ContentNegotiatingViewResolver

```

#### 启动流程

```java
1) 创建 SpringApplicationRunListeners 管理 run 过程的事件, 监听者取自 spring.factories
2) 触发 run 的开始(starting)事件
3) 初始化 environment 对象, 并利用 run 的 environmentPrepared 事件将application.yml的数据注入
4) 打印 Banner, 可自定义 Banner 通过 banner.txt 文件
5) 根据webApplicationType创建一个ApplicationContext 容器
    默认使用 AnnotationConfigApplicationContext
6) 为 context 做一些初始化和设置
    设置环境变量， 使得 context 可以获取 application.yml 中的配置
    调用子类扩展的设置
    加载 容器的 initializers
    触发 run 的 contextPrepared 事件
    打印日志
    添加spring boot 启动参数信息到 bean 容器中
    设置beanFactory的 allowBeanDefinitionOverriding 属性
    设置 懒加载策略， 添加 postProcessor 则会将每个 beanDefinition 的 lazyInit 设置为 true
    把启动类class封装成 BeanDefinition 放到容器中, 使得@Configuration之类的注解生效
    触发 run 的 contextLoaded 事件
7) 调用 context 的 refresh()
    执行 BeanFactoryPostProcessor, 如 ConfigurationClassPostProcessor 解析 @Import 注解
    @Import 会实现 @EnableAutoConfiguration, 总之都是熟悉的 spring 套路. boot的东西就少了.
8) 调用留给子类的 afterRefresh() 方法, 默认空实现
9) 打印启动完毕信息
10) 触发 run 的 started 事件
11) 调用 容器内所有的 ApplicationRunner 和 CommandLineRunner 实现类, 类似事件监听.
12) 使用 spring.factories 中的 exceptionReporters 处理可能出现的异常.
13) 触发 run 的 running 事件
```



#### 一些东西的实现原理

##### `@ConfigurationProperties` 如何实现自动注入`application.properties`中配置的值?

```java
1) 首先加了 @EnableConfigurationProperties 也会解析里面的 @Import 
2) @Import 则引入了 EnableConfigurationPropertiesRegistrar.class
3) 这是一个 ImportBeanDefinitionRegistrar 的实现类, 会在解析 @Configuration 注解时调用指定方法
4) 指定方法 registerBeanDefinitions() 获取 @EnableConfigurationPropertiesRegistrar 的数据
    如 @EnableConfigurationProperties(RabbitProperties.class) 加载 RabbitProperties.class
    然后, 将这些 class 都注册到容器中
5) 指定方法还注册了一些工具bean和一个重要的 BeanPostProcessor 在 registerInfrastructureBeans()中
6) registerInfrastructureBeans() 加载了 ConfigurationPropertiesBindingPostProcessor.class
7) 在 postProcessorBeforeInitialization() 中 调用 ConfigurationPropertiesBinder
8) 调用链很长, 最后 property.setValue(beanSupplier, bound);设置了值 -- JavaBeanBinder

总结: 就是先将 XxxProperties 类定义注入到容器中, 这样可以getBean, 然后通过 BeanPostProcessor
    再实例化后将属性值一一绑定.
```

##### `@ConditionalXxx` 的实现原理

````java
1) 在类上加上注解 @Conditional 或 带有 @Conditional 的其他注解:其他扩展实现
2) 在所有的扫描类和注解的地方,如解析@Configuration, AnotatedBeanDefinitionReader等reader
    会使用 ConditionEvaluator 的 shouldSkip() 判断是否可以加载, 时机点如下
    AnnotatedBeanDefinitionReader#doRegisterBean() 的第二行代码
    ConfigurationClassBeanDefinitionReader#loadBeanDefinitionsForBeanMethod() 第四行
    ConfigurationClassBeanDefinitionReader#loadBeanDefinitionsForConfigurationClass()
    ConfigurationClassParser#doProcessConfigurationClass() 处理 ComponentScan 那段
3) 然后再 shouldSkip 中判断， 判断逻辑大致如下：
    先遍历所有注解取得所有的 @Conditional 下的 所有 value, 这个 value 是具体的Condition实现, 如OnClassCondition
    实例化 Condition 然后添加到 conditions中
    排序并遍历调用 matches(), 一个不匹配则返回true, 代表应该跳过.

TIPS:
ConditionOutcome 封装了是否匹配和匹配日志信息[为啥成功/为啥失败]
SpringBootCondition 提供了通用的根据 ConditionOutcome 判断是否匹配并记录日志信息的抽象类.
    子类只需实现 getMatchOutcome(): 根据 metadata[注解信息] 返回 ConditionOutcome 对象.
    因此, 如果我们要实现自己的 Condition, 可以继承它.
````

> 另发现 `AutoConfigurationImportSelector` 也含有判断Condition的逻辑, 
> 刚开始以为是 `AutoConfiguration` 的类没有走之前说到的判断, 所以这里要做判断.
> 后来我一想, 这是 @Import 引入的, 所以, 是走了判断的. 因此这里多出的一个 `AutoConfigurationImportFilter`, 应该是一个额外的插件, 专门过滤配置在 `spring.factories` 中的 `AutoConfiguration` 类的. 而插件的读取, 也是读取 `spring.factories` 来遍历. 

##### `@AutoConfigureAfter`  的实现原理

```java
1) 首先 AutoConfigurationImportSelector 是一个 DeferredImportSelector  
2) 这种 DeferredImportSelector 会延迟加载, 原理是 parse 后再加载, 而非parse执行过程中就加载.
3) 延迟加载机制 会先调用 process 方法, 将要加载的class保存起来, 然后再调用 selectImports 返回.
4) 此时 AutoConfigurationImportSelector.AutoConfigurationGroup 的 selectImports() 会调用 sortAutoConfigurations(), 也就是调用了 AutoConfigurationSorter.getInPriorityOrder()
5) getInPriorityOrder() 调用了 sortByAnnotation() 这个方法根据2个注解 @AutoConfigureBefore @AutoConfigureAfter 排序.
6) 最后返回的就是有序的了. 另外, 这两个注解只能作用再 AutoConfiguration 上.

总结:
所有的 AutoConfiguration 所引入的class文件解析完毕后, 再准备加载之前, 进行排序, 然后一一加载.
```



##### 各种 `AutoConfiguration` 实现大致流程.

```
1) 首先是 @SpringBootApplication 启用了 @EnableAutoConfiguration
2) @EnableAutoConfiguration 又使用 @Import 导入了 AutoConfigurationImportSelector.class
3) 然后 AutoConfigurationImportSelector 导入 spring.factories所有的 EnableAutoConfiguration
4) 最后，当一个项目依赖一个starter-xxx，会继承starter的依赖项, spring-boot-autoconfigure 由此被依赖
5) spring-boot-autoconfigure 带有所有 boot 实现的 AutoConfiguration 和 spring.factories 配置
6) 所以会加载该项目下 spirng.factories 中定义的 EnableAutoConfiguration
7) 每个 AutoConfiguration 实现类被导入到容器中后
	又被 ConfigurationClassPostProcessor 解析@Configuration, @Import 等注解 (老千层饼了)
	这些AutoConfiguration会添加一些提供服务的 bean，或者再嵌套一层@Import，@Configuration等。
	另外, 这些bean还是被自动配置了属性值的, 属性值哪里来? 都在 application.yml 中, 或是默认配置中.
9) 这样，容器中就加入了一个或多个配置好的bean了, 可以直接使用. 如 stringRedisTemplate, jdbcTemplate
```

#####  Spring Boot 是如何自动扫描main方法所在类所在包的?

```
1) 首先是 @SpringBootApplication 启用了 @EnableAutoConfiguration
2) @EnableAutoConfiguration 又使用 @AutoConfigurationPackage
3) @AutoConfigurationPackage 中的 @Import 会被解析, Registrar.class 的registerBeanDefinitions会被执行
4) 最终根据带有 @SpringBootApplication 的类对应的包名, 然后自动扫描到容器中, 效果同 @ComponentScan
```

##### `application.properties` 是如何被加载到Environment中的?

````java
1) run方法中创建了Environment对象, 当初始化好一些东西后会触发事件
2) 通过 SpringApplicationRunListeners 的 environmentPrepared() 告知监听者
3) 默认存在 spring.factories 中的 EventPublishingRunListener 监听者负责转发事件
4) 事件被转发到 ApplicationListener 下的监听者来处理
5) 监听者配置在 spring.factories 中, 其中 ConfigFileApplicationListener 监听了此事件
6) 此监听者接受 ApplicationEnvironmentPreparedEvent 事件后
7) 加载一些 postProcessor 专门用于处理 environment 对象的 postProcessor
8) 其他的 postProcessor 暂时不管, 真正做了加载的是 postProcessors.add(this); 自己的实现
9) 自己的处理方法是: addPropertySources(), 此方法将会扫描指定的路径下指定的某些文件
10) 然后使用 spring.factories 下的 PropertySourceLoader 一一尝试解析
11) 文件存在且解析正确则加入到 environment 的 propertySources.
	某些路径: getSearchLocations() ,默认: classpath:/,classpath:/config/ ...
	某些文件: getSearchNames() ,默认: application

TIPS: 
PropertySourceLoader 有 PropertiesPropertySourceLoader/YamlPropertySourceLoader
一个尝试后缀有 xml/properties, 另一个是 yml/yaml, 所以所有可能性有:
    classpath:/application.xml; classpath:/application.properties
	classpath:/application.yml; classpath:/application.yaml
    ......
````

##### `SpringApplication.run()` 如何加载 tomcat 的?

```java
1) boot 的 run 里面会创建 applicationContext, 如是 webApplicationType = SERVLET, 则实现类为 AnnotationConfigServletWebServerApplicationContext
    另外一提, webApplicationType 是根据 classpath 下是否有哪些类来推断的.
2) 这个类继承了 ServletWebServerApplicationContext
3) ServletWebServerApplicationContext 实现了 onRefresh()
4) onRefresh() 调用了 createWebServer()
5) createWebServer() 使用 ServletWebServerFactory.getWebServer() 获取 webServer 对象
6) ServletWebServerFactory 会被 ServletWebServerFactoryAutoConfiguration 引入
    详细见 ServletWebServerFactoryConfiguration.EmbeddedTomcat.class
7) 引入后调用 getWebServer(), 大概为 new Tomcat(), 设置属性, 然后启动.
8) 至此, run() 启动了 tomcat 或 jetty/undertow.
 
TIPS:
spring 使用工厂模式获取webServer, 然后工厂又通过AutoConfiguration自动注入(还会判断Condition).
    这样如果新增一种 webServer, 只需要在写一个 AutoConfiguration 注入一个 工厂即可. 非常灵活.
```





### Spring MVC 源码

#### 关键类解析

```
WebMvcConfigurationSupport
	默认注册了很多东西,如HandlerMapping几个实现, HandlerAdaptor几个实现

HandlerMapping
	添加容器内所有带有RequestMaping的类的公开方法到 mappings 中
		(AbstractHandlerMethodMapping#afterPropertiesSet中)
    根据request的uri查找对应的HandlerMethod, 步骤概述:
    	把RequestMapping注解内的path作为key保持到一个map1
    	其他信息封装成mapping作为key也保持到另一个map2
    	根据uri去 map1 获取 mapping, 再根据mapping 获取 HandlerMethod
    	封装成Match对象, 与其他匹配对象做比较后, 返回 HandlerMethod

HandlerAdapter
	初始化参数解析，返回值解析等
		(RequestMappingHandlerAdapter#afterPropertiesSet)
	根据Handle确定对应的HandlerAdapter, 然后执行这个 handler
    如RequestMappingHandlerAdapter 则负责执行 HandlerMethod
    	简单说就是封装 HandlerMethod, 根据参数值设置参数, 然后调用方法, 再处理返回值封装成ModelAndView
    另外， 这里如果使用了@ResponseBody，会进入 RequestResponseBodyMethodProcessor
    	然后使用messageConverters（json）写入到响应流
    	最后mv也直接返回null， 不需要render了。

ViewResolver
	负责将ModelAndView解析成HTML, 如JSP, FreeMarker

HandlerExecutionChian
	管理拦截器和封装Handler, 负责拦截器的实际调用逻辑实现
	
DispatcherServlet
	调度整个HTTP请求响应流程, 调用各个子组件负责执行处理方法, 解析视图, 处理异常等.
```

#### 执行流程

```
GET
HttpServlet#service()
HttpServlet#doGet()
FrameworkServlet#doGet()
FrameworkServlet#processRequest()
FrameworkServlet#doService()
DispatcherServlet#doService()
DispatcherServlet#doDispatch()
	调用容器内所有的HandlerMapping的实现类的getHandler方法, 返回HandlerExecutionChain
	调用容器内所有的HandlerAdaptor的实现类寻找适合的当前Hanlder的HandlerAdaptor
	执行拦截器的 preHandle 方法, 并根据返回结果判断是否接续执行
	HandlerAdaptor执行handle方法
	进入RequestMappingHandlerAdaptor执行handle->handlerInternal->invokeHandlerMethod
	生成ServletInvocableHandlerMethod(就是实现了反射调用方法和设置参数,处理返回值等操作)
	调用invokeAndHandle-->invokeForRequest
		其中getMethodArgumentValues挨个调用HandlerMethodArgumentResolver获取参数值
	然后执行 doInvoke, 利用反射技术调用 Method, method.invoke(obj, args);
	执行完后, 返回结果, 回到 invokeAndHandle 调用 returnValueHandlers 处理返回结果
	返回后 回到 invokeHandlerMethod, 调用 getModelAndView 获取通用的返回值(可能是空)
	返回ModelAndView后, 回到doDispatch, 设置默认viewName (mv为空则不需要设置)
	执行拦截器的 postHandle 方法
	processDispatchResult 中的 render 解析视图后通过response 响应这个view
	根据异常情况执行异常处理器, 以及执行拦截器的 afterCompletion 方法
```

#### 拦截器原理

```
何时加入?
	从WebMvcConfigurationSupport的子类中调用addInterceptors 
	添加一些拦截器和拦截器的路径配置 InterceptorRegistry 和 MappedInterceptor
    实现拦截器路径匹配, 在 new HandlerExecutionChian 时判断

何时执行?
	DispatcherServlet 负责在正确的时机调用 HandlerExecutionChian 来调用 preHanlde 等方法.
	拿到 HandlerExecutionChian 后调用 preHanlde
	HandlerAdapter执行完handler后, 调用 postHandle
	解析视图并渲染到response之后, 调用 afterCompletion
	如果中途出现异常, 或preHandle提前结束, 则也调用afterCompletion

总结
	DispatcherServlet 去调用 HandlerExecutionChian去调用 拦截器具体方法. 
	复杂点是添加一个拦截器到被加入到HandlerExecutionChian比较复杂一点, 以及带路径匹配的拦截器实现略复杂一些.
```



### `Mybatis` 源码

#### 关键类解析

```
Configuration
	作用: 解析和保存大配置(mybatis全局配置,如数据库连接, 别名等), 小配置(每个mapper文件)信息

MapperProxy, MapperMethod
	作用: 生成daoImpl代理对象和实现接口方法: 调用sqlSession 操作方法

Executor
	作用: 协调和管理StatementHandler, ParameterHandler, ResultSetHandler, 解析mapperStatement配置信息成BoundSql.

StatementHandler
	作用: 生成preparedStatment对象(JDBC的), 调用execute方法.

ParameterHandler
	作用: 管理并使用TypeHandler为preparedStatment对象设置参数,

ResultSetHandler
	作用: 将ResultSet结果集转成接口返回值认识的数据, 负责延迟加载.

```

#### 生命周期与执行流程

```
1.读取配置文件, 解析xml(或properties)获得配置信息(数据库配置和mapperSql等) Configuration
2.获取数据库连接, 开启事务, 准备执行sql (openSession)
3.使用动态代理技术生成代理类
4.代理类根据代理方法对应的mapperStatement类型调用sqlSession的insert,update,delete,selectXxx方法 (MapperProxy, MapperMethod)
4.sqlSession调用Executor执行sql (Exector, SimpleExecutor, CachingExecutor)
5.Executor 生成StatementHandler对象并解析xml中sql参数, 再使用ParameterHandler设置参数. (BaseExecutor, StatmentHandler, ParameterHandler, TypeHandler 前三个可被插件拦截任意方法)
6.最后使用OGNL表达式解析库对 if,foreach,where等标签解析生成最总sql (ONGL)
7.执行sql后获得resultSet, 使用 ResultSetHandler 处理后返回结果(resultMap, resultType的处理在这里) (ResultSetHandler 可被插件拦截任意方法)
8.根据事务配置提交事务(自动提交), 保存一级缓存
9.如开启二级缓存, Executor还会被装饰器模式包装一层, 将结果缓存到MapperStatement的cache变量中.

TIPS:
1.插件的使用实在Executor, StatementHandler, ParameterHandler, ResultSetHandler这几个对象实例化的时候, 使用jdk动态代理将配置里配置的形成代理链. 返回代理对象, 但调用方法时, 会判断是否拦截此方法并执行插件的代码.
```

#### 使用到的设计模式

```
1.代理模式
	MapperProxy 生成 mapper 实现类 jdk代理
	Plugin 生成代理对象实现插件 jdk代理
	ResultSetHandler(LazyLoad) cblib, javassist
2.装饰者模式
	CachingExecutor: 装饰SimpleExecutor添加二级缓存功能
	Cache,FifoCache... 使用装饰者模式为缓存添加不同特性(功能)
3.适配器模式
	StatementHandler
4.责任链模式
	Interceptor(拦截Executor、StatementHandler、ParameterHandler、ResultSetHandler对象)
5.策略模式			
	StatementHandler
		PreparedStatementHandler,CallableStatementHandler,SimpleStatementHandler
	Executor
		SimpleExecutor,ReuseExecutor,BatchExecutor
	TypeHandler
		UnknownTypeHandler, IntegerTypeHandler, NStringTypeHandler
6.建造器模式
	org.apache.ibatis.mapping.MappedStatement.Builder
	org.apache.ibatis.builder.xml.XMLConfigBuilder
```

#### 一级缓存实现

```
1.put时机: 
	org.apache.ibatis.executor.BaseExecutor#queryFromDatabase 的 localCache.putObject(key, list)
	
2.get时机:
	org.apache.ibatis.executor.BaseExecutor#query 的 localCache.getObject(key)
	
3.存在哪里
	org.apache.ibatis.executor.BaseExecutor#localCache
	这个类每次openSession会新建一个Executor实例
```

#### 二级缓存实现

```
1.put时机: 
	org.apache.ibatis.executor.CachingExecutor#query的tcm.putObject
	然后实际上是调用了cache的put, cache实际上来自Configuration解析mapper文件时创建的, 即同一个mapper共用同一个cache.
	
2.get时机:
	org.apache.ibatis.executor.CachingExecutor#query的tcm.getObject, cache实际是: 同上

3.remove时机:
	org.apache.ibatis.executor.CachingExecutor#flushCacheIfRequired

注意事项: 上述时机都是TransactionCacheManager的调用, 而TransactionCacheManager会根据事务的提交和回滚来处理缓存的写入与移除时机, 如真实移除时机是下一次commit触发, 真实写入也是commit时触发.

	
4.存在哪里
	org.apache.ibatis.mapping.MappedStatement
	也算是在 org.apache.ibatis.session.Configuration 中
```





### `Mybatis-Plus`





## IO / 网络

### AIO, NIO, BIO

```
BIO: 从 jdk1.4 前, 采用 ServerSocket API 进行网络连接, 所有操作都是阻塞的(如监听, 读/写)
NIO: 从 jdk1.4 起, 对于所有操作都是依赖事件驱动, 只需较少的线程即可处理大量请求
AIO: jdk1.7 起, 在 NIO 基础上实现异步, 所有事件通知由系统通知, 而非NIO 那样轮询
```

### TCP/UDP

```
TCP: 是面向连接的协议, 收发数据前, 必须建立可靠的连接(通过三次握手)
UDP: 是无连接的协议, 收发数据不需要建立连接(握手)

区别:
速度: TCP 满, UDP 快
正确性/完整性: TCP 好, UDP 无
顺序性: TCP 好, UDP 无

应用:
TCP: 文件下载, HTTP, DNS, IM 通讯
UDP: 网络游戏, 直播推流
```

### Netty

> 封装了复杂的NIO接口, 提供了简单的 API 实现服务端和客户端高并发通讯, 并封装了很多工具, 如心跳超时, 半包处理, websocket 等.



## 数据结构与算法

### 数据结构

#### 队列,  栈

```
队列: 先进先出 FIFO
栈: LIFO
```

#### 链表

```
单向链表: Node{next: Node}
双向链表: Node{prev: Node, next: Node}
循环链表: Node{first: Node, last: Node, prev: Node, next: Node}
```

#### 树

```
二叉树:
	先序遍历: 根->左->右
	中序遍历: 左->根->右
	后序遍历: 左->右->根

红黑树:
	红黑树是一种含有红黑结点并能自平衡的二叉查找树.
	任意一结点到每个叶子结点的路径都包含数量相同的黑结点.
	速度: O(log2 n) 最坏: O(log n)
	对比 BST: 查找更快, 插入更慢.
	对比 AVL: 查找略慢, 插入更快.
	
B+树:
	所有记录节点存放在叶子节点上，且是顺序存放，由各叶子节点指针进行连接。
	如果从最左边的叶子节点开始顺序遍历，能得到所有键值的顺序排序。
	查找速度和树高度有关, 如 MySQL 树高度为 3, 则总是查询 3 次找到子节点, 即得到数据.
```



### 算法

#### 排序算法

```
快速排序: O(nlog(n))
	将数组分为2半，并计算一个中间值，将左边所有大于中间值的移到右边，把右边所有小于中间值的移到左边，
	然后分别对左边和右边递归处理（再分2半，再取中间值，再移动，再递归），直到无需递归为止。

归并排序: O(nlog(n))
	将数组不断两两拆分，直到无法拆分（只有2个元素）时开始对拆分后的元素进行排序，然后不断回溯，
	不断排序，直到第一次拆分，即所有元素都是有序的了。排序方式为先取两个部分的第一个下标，
	互相判断，谁小谁放到有序数组中，然后小的那个下标移位，继续判断，直到其中一个部分所有元素取完，
	则将另一部分剩余元素批量直接拷贝到有序数组即可。

冒泡排序: O(n^2) 最好 O(n)
	进行 n - 1 轮， 每轮遍历数组两两比较交换，从而达到把最大的交换到最后面。下轮遍历数组个数减一。

选择排序: O(n^2)
	进行 n - 1 轮，每轮遍历选择数组最小值放到最前面，下轮遍历数组个数减一

插入排序: O(n^2) 最好 O(n)
	分2个数组，有序和无序，开局有序默认含有数组第一个元素，无序含有剩余的元素。
	遍历无序所有元素，从后向前比较大小，然后进行插入到有序数组中或对数组元素移位。

希尔排序: O(n^1.25)
	对数组进行分组，对每组使用插入排序的移位进行排序，第一次分 数组长度/2 组，
	下一次分组为数组长度/2再/2组，若分 长度/2 组，则每组2个， 若分 长度/2/2 组，
	则每组 2*2 个，直到长度为1. 这是一个将数组后段数据尽可能与前段数据排序，
	改进插入排序后端数据过小导致移位操作过多的问题。

基数排序: O(n)
	创建10个桶，每个桶大小均为数组长度，然后遍历数组，从元素的最低位（个位）开始，
	取出并放到对应的桶（如个位是8，则放到第8个桶中），放完后，顺序遍历所有桶，
	顺序取出每个元素，组成新数组，再将新数组按以上流程处理十位，再百位，直到数组中的最大数的最高位。
	
堆排序：
	。。。
```

#### 优化算法

```
贪心算法
爬山算法
模拟退火算法
遗传算法
蚁群搜索算法
```



## 设计模式

### 7 大原则

```
单一职责: 每个类只负责一个职责(或每个方法)
接口隔离: 一个类对另一个类的依赖应建立在最小的接口上
依赖倒转: 高层模块不应依赖低层模块, 二者都应该依赖接口而非细节. 细节依赖抽象, 面向接口编程
里式替换: 子类应该做到可以替换父类, 及子类应尽量不重写父类方法.
开闭原则: 对提供者而已可以修改, 对使用者而言不需要修改(即代码兼容性), 尽量使用扩展增加功能, 而非修改原有类
迪米特法则: 一个对象应该对其他对象保持最小了解(最少知道原则)
合成复用原则: 一个类使用另一个类的代码(方法), 尽量使用合成, 而不是继承
```



### 创建型

#### 单例模式

```
原理: 确保一个类只有一个实例，并提供该实例的全局访问点。

饿汉式:
	静态常量
	静态代码块
懒汉式:
	直接判断(线程不安全)
	方法加 synchronized(线程安全, 效率低)
	判断后再同步(错误写法)
	双重判断(if-同步-if) (推荐写法)
	匿名静态内部类 (简单, 推荐)
	枚举(简单, 但对象方法写在枚举中, 略有不适)
	
示例:
	java.lang.Runtime#getRuntime()
	java.awt.Desktop#getDesktop()
```

#### 原型模式

```
原理: 使用原型实例指定要创建对象的类型，通过复制这个原型来创建新对象.
示例: Java 的 Object 对象的 clone 方法, java.util.Arrays.ArrayList#toArray()

浅拷贝: 仅对基础类型及字符串类型的字段拷贝值
深拷贝: 同时对引用类型(如数组,对象) 也进行拷贝

深拷贝实现:
1.重写 clone, 一一处理每个引用对象(调用对象的 clone), 麻烦, 且若对象之间关系复杂, 其中一个未实现深拷贝则导致 bug.
2.利用序列化和反序列化, 如 Json, 或 Java 自带的序列化方式(二进制)
```

#### 创建者模式(生成器模式)

```
原理:
	封装一个对象的构造过程，并允许按步骤构造.
	若对象的生成过于复杂(字段极多且赋值还有依赖关系, 需要顺序调用), 则可将赋值过程封装成一个build(), 并放到一个 Builder 类中. 此类对外提供各个字段的赋值方法并先保存起来, 直到调用 build(), 此方法返回对象实例. 
	使用此模式, 调用者无需关注构建过程, 只需设置自己想要的值, 然后调用 build() 即可得到对象实例. 且若增加或修改字段, 构造过程变化, 调用者无感知, 无需修改代码. 符合开闭原则.

示例: StringBuilder, 一些框架的 ConfigurationBuilder(如 xmpp), 用于构建配置.
```

#### 简单工厂模式

```
原理:
	在创建一个对象时不向客户暴露内部细节，并提供一个创建对象的通用接口。
	此模式可避免多个调用者创建对象时判断创建哪个子类的重复代码, 且若多一个子类, 调用者无需修改代码.
	
示例: Spring ApplicationContext 的 getBean 方法.
```

#### 工厂方法模式

```
原理:
	定义了一个创建对象的接口，但由子类决定要实例化哪个类。工厂方法把实例化操作推迟到子类。
	此模式解决了简单工厂每增加一个子类需要修改工厂类的问题.
	此模式存在问题, 若新增一个子类, 需同时新增一个子类工厂, 系统复杂性更高.

示例: Calendar, NumberFormat
```

#### 抽象工厂模式

```
原理:
	提供一个接口，用于创建 相关的对象家族.
  同上, 由子类工厂决定创建哪些对象.
	此模式是工厂方法的升级版, 不同之处在于它同时创建多个种类的对象(工厂类具有多个方法).
	此模式将一个对象家族的新建集合到一个工厂类创建管理, 这些对象家族相互之间一般有关联, 在创建时就可以处理这些关联. 且对于 2 个子类工厂, 一般可以无缝切换, 使得修改代码极为方便(即换一个子类工厂).
	此模式在新增一个对象家族的成员时非常麻烦(即所有工厂类需要新增一个方法), 但再新增一类对象家族时比较简单(即新增一个子类工厂).
```



### 结构型

#### 适配器模式

```
原理:
	把一个接口转换成另一个用户需要的接口.
	定义一个类, 实现用户需要的接口, 并聚合一个需要转换的接口对象, 在重写的方法(用户需要的方法)中调用聚合的对象的方法, 若需要返回值, 且返回值类型不一致, 则还需要在方法中处理一番, 然后返回. 这个过程叫做适配.这个类叫做适配器类.
	使用此模式可对一些老旧接口适配兼容.
	
示例: java.util.Arrays#asList() 将数组适配成 List, Spring MVC的 HandlerAdapter
```

#### 装饰者模式

```
原理:
	将一个或多个功能(方法)动态的新增到一个类中.
	把需要新增功能类称为 A,定义一个类B,实现A的上层接口, 并聚合一个A 的实例对象, B类实现的接口中, 对其他不关心的方法直接调用聚合的对象的方法. 对于关心的方法则可以在调用前后进行加料处理(如一个方法返回一个数, 可以在原来的返回值上乘以 2), 同时, B类也可以新增一些其他方法, 这些方法就是多出的功能. B类就是装饰者类, A就是被装饰类.
	此模式的优点是, 装饰类也可以当做被装饰类, 然后再来一层装饰, 可以无限的装饰.
	
示例: java IO 流
```

#### 代理模式

```
原理:
	控制其他对象的访问(方法级), 将一些前置或后置的处理, 通过代理对象注入到目标对象的方法前后. 面向切面编程.

类型:
	静态代理: 定义一个代理类实现目标对象的上层接口, 并聚合一个目标对象, 重写方法时将前置后置处理加上.
	动态代理: 
		JDK 动态代理: 需要目标对象有上层接口(自然接口内的方法才可以代理) 
			使用java.lang.reflect.Proxy#getProxyClass
		CGLIB 动态代理: 是个类就行. 实现原理是 ASM 框架动态生成目标对象类的子类字节码, 然后通过反射生成代理对象.

示例: Spring AOP
```

#### 桥接模式

```
原理:
	将抽象与实现分离开来，使它们可以独立变化。
	桥接的含义是, 一个桥, 放在哪里都有桥的 2 边, 桥的 2 边可以变化, 但桥始终不变. 此处, 桥代表一个操作(如手机上运行软件), 2 边代表 一个操作的 2 个维度(如手机和软件). 同时, 桥接后的操作也可以视为一个维度, 与另一个维度桥接(如手机上运行软件和人这 2 个维度, 可以进行桥接, 组成 3 维度嵌套桥接).
	
示例: JDBC 获取连接, 获取连接是一个维度, 数据库是一个维度, 数据库有多个, 所以这是一个数据库维度变化, 另一维度不变的桥接模式.
```

#### 享元模式

````
原理:
	利用共享的方式来支持大量细粒度的对象，这些对象一部分内部状态是相同的。
	如常见的 线程池, 常量池等, 使得对象的获取速度加快.
	
示例: java.lang.Integer#valueOf() java.lang.Boolean#valueOf()
````

#### 组合模式

```
原理:
	将对象组合成树形结构来表示“整体/部分”层次关系，允许用户以相同的方式处理单独对象和组合对象。
	一般需要部分和整体具有一定的相似度, 才能对其进行抽象.
	对部分/整体进行抽象, 得出一个公共抽象类或接口, 再实现类中根据具体角色做不同处理. 

示例:
	javax.swing.JComponent#add(Component) 
  java.util.Map#putAll(Map)
  java.util.List#addAll(Collection)
  java.util.Set#addAll(Collection)
```

#### 外观模式

```
原理:
	提供了一个统一的接口，用来访问子系统中的一群接口，从而让子系统更容易使用.
```



### 行为型

#### 职责链(责任链)模式

```
原理:
	使多个对象都有机会处理请求，将这些对象连成一条链，并沿着这条链发送该请求，直到有一个对象处理它为止, 从而避免请求的发送者和接收者之间的耦合关系。
	
示例:
	javax.servlet.Filter#doFilter()
	netty 的 Handler Chain
```

#### 观察者模式

```
原理:
	定义对象之间的一对多依赖，当一个对象状态改变时，它的所有依赖都会收到通知并且自动更新状态。
	主题（Subject）是被观察的对象，而其所有依赖者（Observer）称为观察者。

示例: 
	swing 的事件监听(按钮事件, 鼠标事件)
	JS 的 事件监听
```

#### 状态模式

```
原理:
	允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它所属的类。
	状态模式主要是用来解决状态转移的问题，当状态发生转移了，那么 Context 对象就会改变它的行为.
```

#### 策略模式

```
原理:
	定义一系列算法，封装每个算法，并使它们可以互换。
	策略模式可以让算法独立于使用它的客户端。
	策略模式主要是用来封装一组可以互相替代的算法族，并且可以根据需要动态地去替换 Context 使用的算法.

示例: java.util.Comparator#compare() javax.servlet.http.HttpServlet
```

#### 模板方法模式

```
原理:
	定义算法框架，并将一些步骤的实现延迟到子类。
	通过模板方法，子类可以重新定义算法的某些步骤，而不用改变算法的结构。

示例: java.util.Collections#sort()
```

#### 命令模式

```
原理:
	将一个对象(命令接收者)的每个操作拆分到每一个命令类中, 再使用一个命令管理类来管理这些命令. 使得命令可以放入队列中有序执行, 且可以统一记录命令的操作日志, 还可以支持撤销操作(每个命令都实现对应的撤销即可).
	此模式的好处是, 若将命令抽象为几个标准的命令(如开,关), 然后管理多个命令接收者(如灯,电视机,空调)的操作, 可使新增命令接收者变得简单, 即扩展性好.
	
	又称万能遥控器.
```

#### 中介模式

```
原理:
	集中相关对象之间复杂的沟通和控制方式。降低子系统之间的耦合.
	类似一个消息收发中心, 负责字系统的消息中转, 使得子系统之间可以进行一定的交互.
	
示例: 线程池管理者线程和要执行的任务.
```

#### 备忘录模式

```
原理:
	在不违反封装的情况下获得对象的内部状态，从而在需要时可以将对象恢复到最初状态。
	如对游戏的当前状态进行一个保存, 然后在后续游戏中死亡后可以读取这个状态重新开始.
```

#### 访问者模式

```
原理: 
	为一个对象结构（比如组合结构）增加新能力。
	使用访问者模式可实现重载的动态绑定(即伪双分派), 效果与重载方法内使用 instanceof 是一样的, 但使用访问者模式, 可扩展性更好.
```

#### 迭代器模式

```
原理:
	提供一种顺序访问聚合对象元素的方法，并且不暴露聚合对象的内部表示。

示例: java.util.Iterator
```

#### 解释器模式

```
原理:
	为语言创建解释器，通常由语言的语法和语法分析来定义。

示例: EL 表达式, Freemaker模板
```

#### 空对象模式

```
原理:
	使用什么都不做的空对象来代替 NULL, 避免空对象判断, 避免空指针异常.
```



## 高并发

### Redis 缓存

#### 缓存穿透(攻击型)

```
含义:
	对于一个不存在的 key 进行访问, 会导致数据库不停地查询这个 key 进行缓存.
	
解决方案:
	1.使用布隆过滤器, 一定不存在的数据会被过滤.
	2.查询后缓存一个空结果, 但很快超时.(有缺点, 但简单)
```

#### 缓存雪崩

```
含义:
	缓存雪崩是指在我们设置缓存时采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到DB，DB瞬时压力过重雪崩。
	
解决方案:
	1.设置超时时, 在原有的失效时间基础上增加一个随机值，比如1-5分钟随机
	2.加锁或者队列的方式保证缓存的单线 程（进程）, 避免大量请求打到数据库.
```

#### 缓存击穿

```
含义:
	单一热点 key 失效时导致大量请求打到数据库。

解决方案:
	1.分布式互斥锁(redis,zookper)
	2.添加超时字段记录超时(比实际超时小一些), 每次获取数据根据字段判断是否超时, 若是, 则马上延长超时字段, 然后加载数据库重新缓存
	3.redis不设置过期, 通过添加超时字段判断, 超时则代码异步跑一个重新缓存的任务(这里代码需要先本地加锁, 再加分布式锁).
```



### RabbitMQ

> 解耦, 异步, 削峰

#### 解耦

```
定义:
	为面向服务的架构（SOA）提供基本的最终一致性实现. 即将 2 个系统的交互通过消息队列中转, 以防止某个系统临时挂了导致调用失败.
示例: 下单系统调用库存系统, 若当时库存系统正好挂了, 则导致下单失败. 此时将请求放到消息队列中, 库存系统读取消息进行处理, 若当时库存挂了也没关系, 处理失败也没关系(可重试, 且重试代码比较简单).
```

#### 异步

```
定义:
	对于某些不要求返回值的耗时操作, 可异步处理.
示例:
	用户下单后, 需发送多个下单提醒(微信通知, 短信通知, 邮件通知), 每个操作都比较耗时, 可考虑将其放入消息队列后直接返回, 由另一段代码负责读取消息发送通知.
```

#### 削峰

```
定义:
	将请求高峰打平, 使得系统可以处理过来.
示例:
	某次秒杀 1 分钟过来 1 万请求, 而系统一分钟大概只能处理 1千 请求, 系统要处理完这些请求理论需要 10 分钟, 但如果不做处理, 请求瞬间打过来, 系统直接卡死, 卡住时候一分钟可能只能处理 100 请求. 此时需要将所有请求都打到队列里面, 系统再慢慢从队列中读取处理.
```



