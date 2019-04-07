java

- 线程创建： `new Thread()`

- 进程创建：

  ```java
  Runtime runtime = Runtime.getRuntime();
  Process process = runtime.exec("calc");
  ```

- 线程运行

  - `Runnable`
  - `Callable`

- 线程销毁

  - java不支持
  - C++: `pthread.delete()`

- Q：线程t1, t2, t3按照t1->t2->t3的顺序执行
- `join`:

- 线程异常：
  - 线程发生异常时会中断
  - 如何捕获异常：`java.lang.Thread#setDefaultUncaughtExceptionHandler`
  - ThreadPoolExecutor如何捕获异常？

- 获取所有线程信息

  - jstack pid

  - jmx:

    ```java
    java.lang.management.ThreadMXBean#getAllThreadIds();
    java.lang.management.ThreadMXBean#getThreadInfo(long);
    
    public interface com.sun.management.ThreadMXBean extends java.lang.management.ThreadMXBean {
        long[] getThreadCpuTime(long[] var1);
    
        long[] getThreadUserTime(long[] var1);
    
        long getThreadAllocatedBytes(long var1);
    
        long[] getThreadAllocatedBytes(long[] var1);
    
        boolean isThreadAllocatedMemorySupported();
    
        boolean isThreadAllocatedMemoryEnabled();
    
        void setThreadAllocatedMemoryEnabled(boolean var1);
    }
    ```

- 线程同步

  - synchronized关键字在代码块字节码为monitorenter, monitorexit, 在方法为ACC_SYNCHRONIZED修饰符

  - 为什么wait, notify, notifyAll由Object提供

    - java5之前没有互斥对象

    - java5提供了`Condition`: 

      ```
      a {@code Condition} replaces the use of the Object
      * monitor methods
      ```

  - wait
    - 先获得锁
    - 然后释放锁，阻塞当前线程
  - notify, notifyAll
    - 前提：已经获得锁
    - 唤起一个/所有被阻塞的线程
  - 类似的，LockSupport#park(), LockSupport#unpark()
  - wait && park
    - park需要显式唤起
    - wait由jvm帮它唤起

- 线程退出

  - 主线程退出时，daemon子线程会退出吗？

  - `shutdownhook`

    ```java
    java.lang.Runtime#addShutdownHook
    ```

  - 如何确保主线程退出前，所有子线程执行完毕

    ```java
    java.lang.ThreadGroup#enumerate(java.lang.Thread[], boolean)
    ```

- J.U.C

  - java.util.concurrent.ConcurrentSkipListMap

    - 写不需要锁，空间换时间

  - java.util.concurrent.BlockingQueue

    - 进队列：队列满的时候阻塞

    - 出队列：队列空的时候阻塞

    - ArrayBlockingQueue

    - LinkedBlockingQueue

    - PriorityBlockingQueue：put不阻塞，长度不受限

    - SynchronousQueue: take阻塞

      - e.g. `java.util.concurrent.Executors#newCachedThreadPool(java.util.concurrent.ThreadFactory)`

        ```java
        public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
                return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                              60L, TimeUnit.SECONDS,
                                              new SynchronousQueue<Runnable>(),
                                              threadFactory);
        }
        ```

        