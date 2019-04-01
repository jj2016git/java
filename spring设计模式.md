# spring设计模式

## 工厂模式

- 简单工厂
  - 工厂类根据用户指定的参数创建产品
  - 小作坊式工厂，由一个工厂类创建所有产品
  - 问题：
    - 用户指定不合理参数
    - 增加产品时，需要修改工厂类
- 工厂方法
  - 一个工厂创建一类产品
  - 改进点：
    - 用户不需要知道指定什么参数才能生成他想要的产品
    - 封装了产品创建细节
    - 增加产品时，增加新的工厂类
  - 问题：
    - 用户必须指定由哪个工厂创建哪个产品
- 抽象工厂
  - A工厂生成产品A, B工厂生成产品B，用户需要配套使用产品A和产品B
  - e.g. 用户需要包装后的牛奶，A工厂生产各类牛奶，B工厂生产各类包装，用户需要纸盒特仑苏，铁罐旺仔，塑料盒伊利等
  - 问题
    - 产品A和产品B的组合难以扩展

## 单例模式

- 例子：工厂，配置文件，日历，上下文

- 解决并发访问时线程安全问题

- 实现方案

  - 饿汉：类创建的时候初始化

    ```java
    private static final HungrySingleton hungrySingleton = new HungrySingleton();
    private HungrySingleton() {
    
    }
    
    public static HungrySingleton getInstance() {
        return hungrySingleton;
    }
    ```

    - JVM在初始化一个类的时候（即调用类构造函数<clinit>()）会自动同步
    - 线程访问对象晚于JVM创建static对象，因此线程安全

  - 懒汉：使用时初始化，因此有线程安全问题

    ```java
    public class LazySingleton {
        private static LazySingleton lazySingleton = null;
    
        private LazySingleton() {
    
        }
    
        // 线程不安全
        public static LazySingleton getInstance() {
            if (lazySingleton == null) {
                lazySingleton  = new LazySingleton();
            }
    
            return lazySingleton;
        }
    
        public  static synchronized LazySingleton syncGetInstance(){
            if (lazySingleton == null) {
                lazySingleton  = new LazySingleton();
            }
    
            return lazySingleton;
        }
    
        public static LazySingleton syncGetInstance2() {
            if (lazySingleton == null) {
                synchronized (LazySingleton.class) {
                    if (lazySingleton == null) {
                        lazySingleton = new LazySingleton();
                    }
                }
            }
    
            return lazySingleton;
        }
    }
    ```

  - 静态内部类

    ```java
    public class LazyInnerSingleton {
        private LazyInnerSingleton() {
    
        }
    
        public static LazyInnerSingleton getInstance() {
            return LazyHolder.LAZY;
        }
    
        private static class LazyHolder {
            public static final LazyInnerSingleton LAZY = new LazyInnerSingleton();
        }
    }
    ```

    - 静态内部类在`getInstance()`方法内被使用的时候才会被初始化，实现懒加载

  - 注册登记：如spring IOC容器

    - 将对象放在容器中，如果容器中不存在，就new一个，否则直接返回对象

  - 枚举

    ```java
    public enum EnumSingleton {
        INSTANCE("a", "b");
    
        private String token;
        private String desc;
    
        EnumSingleton(String token, String desc) {
            this.token = token;
            this.desc = desc;
        }
    }
    ```

    - 利用枚举自身特性实现**序列化安全**的饿汉式单例
      - Enum构造函数为private, enumClass为final class

  - 序列化与反序列化时如何保证单例

    ```java
    // 饿汉式序列化不安全例子
    public class HungrySingleton implements Serializable {
        private static final HungrySingleton hungrySingleton = new HungrySingleton();
        private HungrySingleton() {
    
        }
    
        public static HungrySingleton getInstance() {
            return hungrySingleton;
        }
    }
    
    public static void main(String[] args) {
            HungrySingleton s1 = null;
            HungrySingleton s2 = HungrySingleton.getInstance();
    
            try {
                FileOutputStream fos = new FileOutputStream("HungrySingleton.obj");
                ObjectOutputStream oos = new ObjectOutputStream(fos);
                oos.writeObject(s2);
    
                FileInputStream fis = new FileInputStream("HungrySingleton.obj");
                ObjectInputStream ois = new ObjectInputStream(fis);
                s1 = (HungrySingleton) ois.readObject();
                System.out.println(s1 == s2);
            } catch (Exception e) {
                e.printStackTrace();
            }
    
        }
    ```

    ```java
    // 解决
    package com.spring.boot.webdemo.singleton;
    
    import java.io.Serializable;
    import java.util.concurrent.CountDownLatch;
    
    public class HungrySingleton implements Serializable {
        private static final HungrySingleton hungrySingleton = new HungrySingleton();
        private HungrySingleton() {
    
        }
    
        public static HungrySingleton getInstance() {
            return hungrySingleton;
        }
    
        public Object readResolve() {
            return hungrySingleton;
        }
    }
    
    ```

### 原型模式prototype

e.g. 数据库DTO转VO,VO转model, 对象属性名称、类型均相同，实现对象复制

- 浅拷贝
- 如何实现深拷贝
  - java序列化

```java
public class A implements Cloneable, Serializable {
    public B b;

    public A() {
        this.b = new B();
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        // return super.clone(); // 浅拷贝，a1!=a2 a1.b==a2.b
        return this.deepClone(); // 深拷贝，a1!=a2 && a1.b!=a2.b
    }


    private Object deepClone() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(bos);
            oos.writeObject(this);

            ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bis);
            return ois.readObject();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}

public static void main(String[] args) {
    A a1 = new A();
    try {
        A a2 = (A)a1.clone();
        System.out.println(a1 == a2);
        System.out.println(a1.b == a2.b);
    } catch (CloneNotSupportedException e) {
        e.printStackTrace();
    }
}
```



