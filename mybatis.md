# mybatis

## what

### a：定义

- 持久化框架
- 优势：无需手工处理参数及返回结果
- 怎么做：xml，注解的方式实现primitives, Mapper interface, POJO到数据库记录的映射##

### b：JDBC与mybatis对比

## how

### 使用方式

#### 编程式

- - sqlsessionFactory -> SqlSession（与数据库一次交互就是一次会话）->

  > COC conversation over configuration约定优于配置
  >
  > 框架根据Configuration生成SqlSessionFactory
  >
  > 每个框架xml标签都对应了一个类

#### 集成式

#### 工作中的使用方式

#### generator

- lombok

#### scope

- Mapper在spring管理下是singleton, application级别
- 在编程式里面是method级别

#### Mapper的xml和annotation

- 兼容么？兼容的形式
  - 可以都用，一个Mapper method只能用xml或annotation
- pros and cons
  - xml集中管理sql，sql可以很复杂
  - annotation可以直接看到方法的sql，复杂sql影响可读性
- mybatis + spring没有xml文件会不会报错？why?

## configuration

#### environment

#### typeHandler

#### plugin

- interceptor：`@Signature`配置切入方法

> Q：mybatis为什么不用拦截器实现日志？
>
> A： 

#### mapper

- namespace：关联sql，sql重名时，区分sql作用域，namespace#sql作为sql的全限名

- resultType/resultMap

- sql

- CRUD

  - insert自动生成id，

    `useGenerateKeys="true" keyProperty="..."`

- 动态sql：官网介绍

> sql判断字符串时，需要判断字符串!=null 且不为空

- 缓存

  - 一级缓存：基于sqlsession，默认打开
    - 清除策略：
      - Executor.update()清除
      - sql手工设置flush清除

  > 1. 为什么要一级缓存？减少数据库压力
  > 2. 如何验证一级缓存？从结果展示？
  > 3. 一级缓存的问题？
  >
  > ```java
  > // thread 1
  > public void query() {
  >     sqlsession.selectOne();
  >     sqlsession.selectOne();
  > }
  > 
  > // thread 2在thrad 1第一次查询后，更新数据，thread 1的缓存未更新，导致脏数据
  > ```
  >
  > 4. 为什么有问题，还这样设计？一个session里面连续两次查询相同数据的概率很低。优点大于缺点

  - 二级缓存：基于namespace，默认关闭，不建议使用，一般用redis替代
    - 缓存基于namespace，只要访问同一个namespace的同一个方法，就会命中二级缓存

  > 1. scope? namespace
  >
  > 2. 怎么验证二级缓存？多个sqlsession发起相同查询，统计每次查询时间
  >
  > 3. 二级缓存问题？
  >
  >    脏数据：e.g. Blog关联Author，Blog中缓存了queryBlogWithAuthor结果，Author update，queryBlogWithAuthor出现问题
  >
  >    缓存全部失效：更新操作后，会全部失效namespace的缓存


## best pratice

### 分页

- mybatis自带分页：逻辑分页
- 物理分页：PageHelper
  - mysql: limit

### 批量操作

|                           | 性能            | 缺点        |
| ------------------------- | --------------- | ----------- |
| for循环一个一个插入       | n sql, n commit | 效率低      |
| foreach（性能最高，推荐） | 1sql, 1 commit  | sql长度限制 |
| batch executor            | n sql, 1 commit | 使用复杂    |

### 联合查询

#### 嵌套结果

- join

#### 嵌套查询

Blog, Author, Post

- 1：1 查询Blog with Author

```xml
// sql1查询blog, sql2根据sql1 author_id查询author
```

- 1：N查询blog with多个posts

```xml
// sql1查询blog, sql2根据blog_id查询post列表
```

- N+1问题

  1个Blog查询 + 1个Post批量查询 + 。。。。。。

  如果仅需要Blog信息，后面的N次查询没有意义

  启动懒加载，可以避免后面N次查询