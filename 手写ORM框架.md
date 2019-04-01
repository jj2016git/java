# 手写ORM框架

## 需求

- 连接多种数据库进行CRUD，实现数据库查询结果与POJO的映射
- 单表查询不写sql
- 支持jdbc，redis，Mongodb数据查询
- extension:
  - 支持动态切换数据源，做到读写分离等

## ORM框架 object relation mapping

- 定义：将sql结果映射成java object
- 例子：
  - HIbernete：全自动，自动拼接sql，映射结果
  - mybatis：半自动，手写sql，自动映射结果
  - spring jdbc template：全手动，手写sql，手工映射结果

## 约定

- 统一DAO方法名
  - select
  - insert
  - delete
  - update
- delete, update以id为唯一检索条件
- 统一参数
  - 条件查询，将查询条件封装成QueryRule对象
  - 批量更新、插入，方法名以All结尾，参数为list
  - 删除、插入、修改数据，参数为T
- 统一返回值
  - 分页操作返回Page
  - 集合查询返回List
  - 单条查询返回T
  - id采用long类型
  - 删除、修改、增加返回boolean

## ORM框架设计

- 各种数据库连接配置
- 生成查询语句
  - 初始化查询规则
  - 根据查询规则生成查询语句
- 实现不同数据库CRUD操作 + 查询结果集映射成POJO

- 结果集映射

  > 约定：java entity属性与表字段同名

  - springboot如何配置datasource及连接池
  - spring jdbc template实现简单的单表查询
  - 利用反射实现mapping

- 复杂条件查询

  - where
    - prop operator value       
      - AndEq(prop, val)
      - AndNotEq(prop, val)
      - AndGt(prop, val)
      - AndLt(prop, val)
      - AndLe(prop, val)
      - AndGe(prop, val)
    - between val1 and val2
      - AndBetween(prop, val1, val2)
    - in (val1, val2, val3)
      - AndIn(prop, Object... args)
    - like
      - AndLike(prop, val)
    - and/or组合以上单条规则
      - AndRule(rule1, rule2)
      - OrRule(rule1, rule2)
  - order
    - order by col1 asc/desc/(asc), col2 ......
      - Order(prop, asc/desc)
  - 如何将以上规则封装成QueryRule
  - 如何利用`select * from table_name` +QueryRule生成查询语句 

- 多数据源

- 多种类型数据库统一API

