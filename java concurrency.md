## 内存模型

![](https://github.com/jj2015wing/images/blob/master/jmm%20model.png)

### 并发三大问题

- 可见性
- 原子性
- 有序性

#### 导致原因

- CPU高速缓存--缓存不一致
- 多核心下多线程通过共享内存通信

### CPU层面如何解决缓存一致性问题：

http://www.importnew.com/10589.html

- MESI协议(M modified, E exclusive, S shared, I invalid)
  - CPU0 数据D S->M
  - CPU1 数据D S->I
  - CPU0修改共享数据D
    - 修改数据D，通知其他CPU将数据D改成invalid状态，等待响应
    - 其他CPU将数据改成invalid状态后，CPU0将数据D的新值写入主存（**阻塞**）
- MESI改进：
  - storebuffer: 将数据写入storebuffer，异步等待其他CPU失效确认，得到所有失效确认结果后，将数据写入
  - 失效队列：
    - 对于所有的收到的Invalidate请求，Invalidate Acknowlege消息必须立刻发送
    - Invalidate并不真正执行，而是被放在一个特殊的队列中，在方便的时候才会去执行。
    - 处理器不会发送任何消息给所处理的缓存条目，直到它处理Invalidate

- 内存屏障：解决可见性及指令重排问题
  - load barrier：强制执行失效队列中所有invalidate，从主存中重新读数据
  - store barrier:：将storebuffer中所有数据写入主存
  - full barrier: 强制执行storebuffer及invalidate队列
- 总线锁：影响所有CPU与所有其他组件的通信

### JMM层面如何解决

- 内存屏障: XY 内存屏障：barrier前的X操作必须先于barrier后的Y操作完成
  - loadload
  - loadstore
  - storeload
  - storestore

> Memory barriers只能用在一个线程内。恰当地组合使用它们，你可以保证其他线程在加载这些值的时候看到一致的情况。
```java
// acquire(): acquire之后的操作必须在acquire完成后执行
// release(): release之前的操作必须在release执行前完成
// fence(): fence之前的操作必须先完成，之后的操作不能越过fence
inline void OrderAccess::loadload()   { acquire(); }
inline void OrderAccess::storestore() { release(); }
inline void OrderAccess::loadstore()  { acquire(); }
inline void OrderAccess::storeload()  { fence(); }
```
## `volatile`原理

## `Synchronized`原理

- markOop

https://blog.csdn.net/zqz_zqz/article/details/70233767

#### 锁的获取过程

自旋锁：对象锁存在时间非常短，线程获取锁失败挂起，唤醒代价大，因此采用自旋

- 自旋：没有意义的for循环

  ```java
  for(;;){
      获取锁
  }
  ```

偏向锁：前提--大多数情况下，锁由同一个线程获得，不存在竞争。

- 对象头记录获得锁的线程id
- 后续同一个线程访问同步代码块时，不需要通过CAS获取锁
- **如果在运行过程中，遇到了其他线程抢占锁，则持有偏向锁的线程会被挂起，JVM会消除它身上的偏向锁，将锁恢复到标准的轻量级锁。**
- 如果CAS获取偏向锁失败，则表示有竞争。当到达全局安全点（safepoint）时获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。（撤销偏向锁的时候会导致stop the word）

轻量级锁



锁范围：

1. 作用于方法时，锁住的是对象的实例(this)；
2. 当作用于静态方法时，锁住的是Class实例，又因为Class的相关数据存储在永久带PermGen（jdk1.8则是metaspace），永久带是全局共享的，因此静态方法锁相当于类的一个全局锁，会锁所有调用该方法的线程；
3. synchronized作用于一个对象实例时，锁住的是所有以该对象为锁的代码块。

