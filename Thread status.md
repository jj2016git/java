## Thread status

### interrupt 优雅停止线程

- `thread.interrupt()`：线程A调用线程B的interrupt()方法，即告诉线程B可以终止了，具体是否终止/何时终止，由线程B决定
- `thread.isInterrupted()`：查询线程interrupt状态位
- `thread.interrupted()`：interrupt状态复位
- 抛出`InterruptException`时，状态复位

#### `interrupt()`源码

- 唤醒当前线程，并设置状态位为true

```C++
void os::interrupt(Thread* thread) {
  assert(Thread::current() == thread || Threads_lock->owned_by_self(),
    "possibility of dangling Thread pointer");

  OSThread* osthread = thread->osthread();

  if (!osthread->interrupted()) {
    osthread->set_interrupted(true);
    // More than one thread can get here with the same value of osthread,
    // resulting in multiple notifications.  We do, however, want the store
    // to interrupted() to be visible to other threads before we execute unpark().
    OrderAccess::fence();
    ParkEvent * const slp = thread->_SleepEvent ;
    if (slp != NULL) slp->unpark() ;
  }

  // For JSR166. Unpark even if interrupt status already was set
  if (thread->is_Java_thread())
    ((JavaThread*)thread)->parker()->unpark();

  ParkEvent * ev = thread->_ParkEvent ;
  if (ev != NULL) ev->unpark() ;

}
```



