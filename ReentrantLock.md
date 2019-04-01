## `ReentrantLock`

`java.util.concurrent.locks.ReentrantLock#lock()`

- `java.util.concurrent.locks.ReentrantLock.NonfairSync#lock()`

```java
final void lock() {
    if (compareAndSetState(0, 1)) // 尝试上锁，非公平性的体现
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

- `java.util.concurrent.locks.ReentrantLock.FairSync#lock()`

```java
final void lock() {
    acquire(1);
}
```

> 非公平锁 vs 公平锁
>
> - 非公平锁偏向于当前线程，当前线程在排队之前，先获取一回锁
> - 公平锁：当前线程直接去同步队列排队
>
> state 锁同步状态
>
> - state==0：锁空闲
> - state > 0：锁被ownerThread占用，可重入锁，state > 0表示ownerThread的重入次数。释放锁时，每释放一次，state--，直至state = 0, 锁被释放

- `java.util.concurrent.locks.AbstractQueuedSynchronizer#acquire()`

```java
// tryAcquire 尝试获取锁
// addWaiter 将当前线程封装成node，放入同步队列
// acquireQueued 自旋获取锁
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

- `java.util.concurrent.locks.ReentrantLock.NonfairSync#tryAcquire`
  - `java.util.concurrent.locks.ReentrantLock.Sync#nonfairTryAcquire`

```java
// 非公平方式获取锁
// 未锁定，或线程重入时，可获得锁
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    // 未锁定，立即获取锁
    // 非公平锁的实现途径
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        // 线程重入，增加重入次数
        // 因此，state表示线程的重入次数，线程释放锁时，需要清空重入次数
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

- `java.util.concurrent.locks.AbstractQueuedSynchronizer#addWaiter`将当前线程包装成node，并放到队尾
  - `java.util.concurrent.locks.AbstractQueuedSynchronizer#enq`

```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node())) // head被初始化为空节点
                tail = head;
        } else { // 自旋设置node为tail
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

- `java.util.concurrent.locks.AbstractQueuedSynchronizer#acquireQueued`

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) { // head->next == node时，自旋获取锁
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```





## `AQS`

- 

  ```
  A node becomes head only as a result of successful acquire
  ```

- state 同步锁状态

  - `state == 0 `

- waitStatus

  - `CANCELLED = 1`: 线程已取消
  - `SIGNAL = -1`:node的successor线程需要unpack
  - `CONDITION = -2`:线程等待被唤醒
  - `PROPAGATE = -3`:???

```java
public final void acquire(int arg) {
	// 1. 获取锁失败
	// 2. addWaiter将线程封装成node,加入同步队列
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) {
    	selfInterrupt();
    }
}



final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        // 自旋
        // head->next == node(当前线程) && 当前线程成功获得锁
        // -> node出队，head = node
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
