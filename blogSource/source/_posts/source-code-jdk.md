---
title: JDK 源码笔记
date: 2021-01-25 23:12:48
tags: 
  - jdk
categories:
  - [java]
  - [interview]
  - [source-code]
---



# JDK 源码笔记



## ArrayList

> 核心就是 `newCapacity` 方法，这个方法用于确定扩容后的数组大小，正常是原来的 1.5 倍（老二进制运算了），若扩容后仍不够大，则仅保证能放下新加入的数据即可（当使用 ``addAll` 方法时可能触发）；若扩容后溢出，则仅保证能放下新加入的数据即可；若扩容后逼近溢出，则返回 `MAX_ARRAY_SIZE` 或 `Integer.MAX_VALUE`；另外两次扩容后过大也会检查 `minCapacity` 是否溢出，防止数据错误。



## HashMap

> 核心是根据 hash 取数组下标 index，并且 1.8 的扩容后 rehash 利用二进制高低位优化了计算下标的效率；扩容为原来的两倍，即左移 1 位。



### 为什么 `HashMap` 的初始容量以及扩容后的容量均为 2 的指数幂

> 因为计算机做运算时, 取模运算速度远远慢于位运算, 而若容量始终为 2 的指数幂, 则根据 hash 获取数组下标时只需要 使用 ` (数组长度-1) & hash 值  ` 即可确定数组下标, 与取模得到的下标一样可靠.
>
> 而扩容后后, 因为需要进行 rehash 运算来确定 数据的新下标, 多次进行取数组下标则更能体现位运算的优势.



### 为什么 `HashMap` 的加载因子是 0.75 (3/4)

```
使用排除法:
1.若加载因子为 1. 则每次 HashMap 满了才进行扩容, 必将有更高的几率触发 hash 碰撞导致数组下标一致需要转成链表或红黑树, 导致读取和更新速度降低.
2.若加载因子为 0.5. 则每次 HashMap 都有一半容量剩余, 空间大大浪费, 对内存开销太大. 容易引发 OOM 事故.
3.0.5-1 之间那么多可能, 选哪个都行, 但作为 HashMap 的默认值, 选中间的 0.75, 走中庸之路, 也是解释的通的.
```



### 为什么 `HashMap` 1.8 扩容无需 rehash

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



### 为什么 `HashMap` 从链表达到 8 个时转成红黑树, 达到 6 个时转回链表?

```
1.根据 Poisson distribution 定律, 凑齐8个节点碰撞到同一个下标, 组成长度为 8 的链表概率极低, 约为 0.00000006, 而超过 8 个的几率则更低, 大约为千万分之一. 所以将阈值设置为 8, 因为这种概率极低. 因此可以减少链表转红黑树的, 提高增删改效率.
2.若达到 7 个时转回链表, 则可能会导致HashMap 不停的在链表和红黑树之间转换, 所以阈值设置为 6, 可起到缓冲效果.
```







## JUC



### ReentrantLock

#### 加锁

```java
// AQS 的 acquire： AbstractQueuedSynchronizer#acquire()
public final void acquire(int arg) {
  if (
    !tryAcquire(arg) 
    && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
  ) {
    selfInterrupt();
  }
}
// 1.tryAcquire：判断 state 状态，无线程持有则 CAS 加持 state，并返回 true，加锁结束。
// 2.addWaiter：已被持有则先加入队列
// 3.acquireQueued：两件事，第一，循环尝试持有锁，失败则休眠；第二，持有成功则更新队列头为自己（队列向前移动）
//    加入队列后，不能立刻睡，防止没人唤醒自己（比如还没加入等待队列，上个持有锁的线程就结束了然后触发解锁代码，而解锁代码的唤醒依赖于等待队列）
//    确定得不到锁（需要等待）则进入睡眠，醒来后再尝试 tryAcquire 持有锁，失败则睡眠（循环尝试）
//    醒来尝试持有锁成功后，维护队列（即队列头部设为自己，宏观上队列往前移动了）
```

> 总结：先判断后更新 state  状态（即是否有线程持有此锁），被持有则加入队列并睡眠；醒来后再重复判断 state 过程，若持有成功则更新队列（主要目的是令我的下一个节点被唤醒后可以尝试持有锁，因为一醒来就是判断 node.prev==head）。



#### 释放锁

```java
// AQS 的 release：AbstractQueuedSynchronizer#release()
public final boolean release(int arg) {
  if (tryRelease(arg)) {
    Node h = head;
    if (h != null && h.waitStatus != 0)
      unparkSuccessor(h);
    return true;
  }
  return false;
}
// 1.tryRelease：减持 state 数量，减到 0 代表线程彻底释放了这个锁
// 2.unparkSuccessor：唤醒下一个节点的线程（若下一个节点为空，则从队列尾往前查找队列中状态不是 CANCELLED 的且最靠前的节点来唤醒）
// 3.唤醒后处于 acquireQueued 的循环中。
```

> 总结：先更新后判断 state 状态，若为 0，则代表可唤醒下一个节点，找到下一个节点，唤醒即可！



从上得到可重入锁原理，同一线程第二次加锁，会令 state+1，

而释放锁时，会先令 state-1，若 state 减到 0，才唤醒下一个节点的线程。



### AQS

```
AQS
	acquire：通过 tryAcquire 判断是否需要加入队列以及休眠
	release：通过 tryRelease 判断是否需要唤醒下一个节点的线程
	tryAcquire：正向维护 state，有自己的逻辑决定返回值
	tryRelease：反向维护 state，一般与 tryAcquire 互逆
	
	acquireShared：通过 tryAcquireShared 判断是否需要加入队列以及休眠（此时节点标记为 SHARED）
	releaseShared：通过 tryReleaseShared 判断是否需要唤醒下一个节点，若下一个节点是 SHARED 节点，则还会继续唤醒下一个节点，若都是 SHARED 节点，则都会被唤醒。
	tryAcquireShared：正向维护 state，返回值小于 0 代表需要进入队列
	tryReleaseShared：反向维护 state，一般与 tryAcquireShared 互逆
```

>总结：一个具体的 AQS 维护一个队列以及 state 数据，总是通过 tryAcquire、tryRelease 维护 state，并在 acquire 中完成队列的添加与移除。
>
>AQS 维护的队列总是在尾部增加节点，并且移除节点时，总是移除 head 节点。
>
>通过实行具体的 tryAcquire、tryRelease 方法，可控制 线程是否等待，何时唤醒线程。
>
>再总结： AQS 的主要作用就是将线程加入到队列、移除队列、控制休眠和唤醒。



### ReentrantReadWriteLock

- 读锁加锁：只要 state 中没有写锁，又没有超过最大数量（受 32 位二进制长度限制），则能加锁成功（非公平锁不进队列等待，公平锁只可能会排在队列中写锁节点后面）；
- 读锁解锁：先减少 state 中的读锁数量，减到 0 则可触发 releaseShared 来唤醒队列中可能存在的写锁节点；
- 写锁加锁：若没有写锁也没有读锁，令 state+1 写锁，记录重入次数；
- 写锁解锁：先减少 state （可重入锁），若写锁数量为 0，则可唤醒队列中的下一个节点，若下一个节点为读锁加入的节点，则会唤醒下下个是读锁的节点，重复，则唤醒大量读锁节点。

> 总结：重写 AQS 的核心方法，使其得以控制读锁、写锁能否加锁成功，利用 AQS 的 SHARED 节点在 releaseShared 时的传播性，使得写锁结束后，可以唤醒一连串的读锁。



### CountDownLatch

```java
// new CountDownLatch(n)：设置 state 的初始值为 n，使得需要调用 n 次 countDown 才会唤醒调用 await 的线程
// countDown：state - 1，若 state 减到 0，则唤醒所有队列中的节点
// await：加入到 AQS 队列休眠，节点标记为 SHARED
```

> 总结：利用 releaseShared 会传播唤醒队列上相连的 SHARED 节点特性和 state 的维护，使 await 的线程进入休眠队列，直到 countDown 调用次数足够多才会唤醒这些线程。



### Semaphore

```java
// new Semaphore(n)：设置 state 的初始值为 n，当想唤醒调用 acquire(x) 的线程，则需要调用 release(x-n)
// acquire(x)：加入到 AQS 队列休眠（前提 state < x) 
// release(x)：state + x，总是尝试唤醒队列

理解方式：acquire 多少个数，则代表需要等待多少个 release，若有初始值，则需要的 release 再减去初始值；满足则唤醒，为满足则进入队列休眠
```

> 总结：利用 AQS 可控制线程休眠和唤醒的特性，重写 tryAcquireShared、tryReleaseShared 方法实现逻辑；另外，虽然 acquire、release 的语义与正常 AQS 保持一致，但其对 state 的操作（加减）却是反着来的。



### CyclicBarrier

> 含义: 凑足一定个数线程, 然后批量唤醒.
> await(): 利用 ReenrantLock 的 lock 和 condition 的 await 进入休眠
> 当凑足后，用condition 的 singleAll 唤醒所有 await 的线程.



### ConcurrentHashMap

> 不同之处有二
>
> 一是 put 一个新的 key 时，会使用 CAS 确保这次 put 没有线程竞争；
>
> 二是 put 覆盖一个 key 时，直接使用 synchronized 同步块进行同步。





## ThreadPoolExecutor

```java
// java.util.concurrent.ThreadPoolExecutor#execute
public void execute(Runnable command) {
  if (command == null)
    throw new NullPointerException();
  int c = ctl.get();
  if (workerCountOf(c) < corePoolSize) {
    if (addWorker(command, true))
      return;
    c = ctl.get();
  }
  if (isRunning(c) && workQueue.offer(command)) {
    int recheck = ctl.get();
    if (! isRunning(recheck) && remove(command))
      reject(command);
    else if (workerCountOf(recheck) == 0)
      addWorker(null, false);
  }
  else if (!addWorker(command, false))
    reject(command);
}
```

1. 若当前线程数少于核心线程数，则添加一个线程来执行任务，直到线程数超过核心数；
2. 超过之后，会放入到 workQueue 队列中，并重新检查线程池状态及是否有线程可执行队列中的任务
3. 若放入失败（即队列已满），则尝试添加一个线程来执行这个任务，此时若线程达到最大线程数，则会失败并调用 reject 方法触发用户设置的拒绝策略。



> 总结：线程池先创建满核心线程数，超出放入队列，队列也放不下则会新建线程来救急，如果救急的线程创建过多，最终总线程数超过最大线程数，则触发拒绝策略，线程池不再接收任务，除非队列空出位置或线程数量降下来。



