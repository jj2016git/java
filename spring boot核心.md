# spring boot核心

## spring framework手动装配

### spring模式注解(stereotype annotation)

@Component及其派生的注解(meta-annotated)

> meta-annotated: 被注解标注的注解，e.g. @Service

- @Service
- @Controller
- @Configuration
- @Repository

装配

- xml方式
  - `<context:component-scan>`配置扫包路径
- `@ComponentScan`

#### 自定义模式注解

- @Component的“派生性”
  - 被父注解标注
  - 与父注解方法签名相同
- @Component“层次性”

@Enable基于注解驱动方式

- Configuration bean

```java
@Documented
@Import(DelegatingWebMvcConfiguration.class) 
// 将配置类中@Bean标注的方法注册为bean
public @interface EnableWebMvc {
}
```

- selector

```java

```



条件装配

- Bean装配的前置判断
- @Profile
  - 指定bean装配的profile
  - 根据spring运行时的profile，装载profile对应的bean
- @Conditional



## spring boot自动装配

> 约定大于配置

- 装配方式
  - 模式注解
  - @Enable
  - 条件装配
  - 工厂加载模式
    - 配置文件spring.factories + SpringFactoriesLoader