## java并发理论基础

### 线程安全实现手段

- 可重入
- thread-local storage: java ThreadLocal
  - 线程内部有一份copy
- 不可变对象
- 互斥：可能导致死锁、活锁、饿死
- 原子操作
- 同步

















BeanDefinitionReader是用来解析xml bean，注解的话直接通过反射就能生成。xml定位，加载，注册





任意ApplicationContext调用refresh，最终会调用org.springframework.context.support.AbstractApplicationContext#refresh()，完成IOC + DI

