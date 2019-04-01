## java函数式编程

### 理解@FunctionalInterface

@FunctionalInterface

- 信息注解
- 接口注解
- 接口仅能有一个抽象方法，加default方法及覆盖Object public方法

接口只要满足函数式接口定义，就会被编译器视为函数式接口，不一定需要@FunctionalInterface注解

why只能有一个抽象方法？实现映射到方法声明上，因此只能有一个

- 类型
  - `Supplier<T>`: 只出不进生产者
  - `Consumer<T>` : 只进不出消费者
    - andThen: consumer -> consumer -> ...
  - `Functional<T>`: 数据转换A->B
    - compose: f.compose(g) = f(g(x))

  - `Predicate<T>`: 过滤器/比较器
  - Action：e.g. Runnable, 

匿名内置类：生成类OutClass$1, 调用时字节码为invokestatic

lambda：不生成类，通过InvokeDynamic指令实现 @since 1.7

- MethodHandler

### 函数式接口设计

### 框架中运用

spring boot Binder类

### stream API

- of
- reduce
- filter
- map
- sorted





假设是一个IntStream pipeline，中间有一环操作是filter，如何操作

```java
Stream filter(Predict<Integer> predict) {
    
}
```



注解：通过动态代理实现AnnotationInvocationHandler

注解是接口，注解的实现就是实现接口的代理类



Stream作为参数时，必须首先判断Stream是并行还是串行，如果操作涉及的数据之间有依赖时，需要改成串行





lambda调试工具：idea

























































栈帧 --> 动态链接：运行时多态，指向真正的实例