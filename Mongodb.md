# Mongodb

[TOC]

## Mongodb应用场景及实现原理

### 应用场景

- 适用

  - 网站实时数据：日志、timeline, 用户行为（代替方案：日志）
  - 数据缓存：缓存的数据必须在数据库持久化
  - 图片、视频等文件存储
  - 高伸缩性：可随意增减机器
  - 对象或json数据存储：redis更合适
- 不适用
  - 高度事务性
  - 结构化查询要求高

- 如何存储日志：https://blog.csdn.net/huyangg/article/details/78772406

    > 如何用Mongodb存储日志？
    >
    > 1. document pattern设计：
    >
    >    最好从日志中提取出价值字段，按照json格式存储，而不是将一条日志作为一个文本存入
    >
    >    如：
    >
    >    ```java
    >    // 按照文本方式存储
    >    {
    >          _id: ObjectId('4f442120eb03305789000000'),
    >         line: '127.0.0.1 - frank [10/Oct/2000:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326 "[http://www.example.com/start.html](http://www.example.com/start.html)" "Mozilla/4.08 [en] (Win98; I ;Nav)"'
    >    }
    >    
    >    // 按照json格式存储价值字段
    >    {
    >         _id: ObjectId('4f442120eb03305789000000'),
    >         host: "127.0.0.1",
    >         time: ISODate("2000-10-10T20:55:36Z"),
    >         path: "/apache_pb.gif",
    >         referer: "[http://www.example.com/start.html](http://www.example.com/start.html)",
    >         user_agent: "Mozilla/4.08 [en] (Win98; I ;Nav)"
    >    }
    >    ```
    >
    > 2. client端如何写日志：`writeConcern`
    >
    >    https://yq.aliyun.com/articles/54367?spm=5176.100239.blogcont68353.18.kIbqJ4
    >
    > 3. 查询日志：对关键字段建立索引
    >
    > 4. 数据分片：如何分片，保证数据均匀落到shard上，且具有较高的查询效率
    >
    >    1. 时间戳分片：
    >       1. 新的写入都会分到同一个shard，并不能扩展日志写入能力，
    >       2. 很多日志查询是针对最新的数据，而最新的数据通常只分散在部分shard上，这样导致查询也只会落到部分shard
    >    2. 随机字段分片
    >       1. 分片均匀
    >       2. 范围查询效率低
    >    3. 均匀分布的key分片
    >       1. key的设计及选取较困难
    >
    > 5. 清理过期日志
    >
    >    1. TTL索引
    >    2. Capped集合：固定大小集合，自动删除最老的document
    >    3. 定期按集合或DB归档


### Mongodb集群架构

![](C:\Users\lacri\Downloads\mongodb_cluster.png)
- config server: 记录数据存储在哪个shard server上
  - 数据分片的key与chunk的映射关系
  - chunk与shard server映射关系
- mongos route server根据key查找到shard server, 将client的读写请求发送到对应的shard server上执行
- replica set1与replica set2是主从备份

> mongodb数据存储方式
>
> chunk --> shard --> replica set --> database

### 基本概念

- database：数据库
- collection：table
- document：row
- field：column
- objectId: 主键

## 常用命令及配置

### linux下mongodb安装及配置

- 安装
  - 解压
  - PATH配置：`vim /etc/profile; export PATH=$PATH:mongo安装目录/bin`
- 命令行配置
  - 配置`logpath`，`dbpath`
  - 后台启动`--fork`
- 配置文件配置：`mongod -f 配置文件`

> linux下mongodb学习方法
>
> - mongodb帮助：`mongod -h`
> - mongo命令帮助：进入mongo client命令行后，输入help

### CRUD

> mongodb中任何对象都不需要预定义，如database, collection等

> mongodb支持js

> 官方文档" SQL to Mongodb Mapping chart"

### 课后答疑
mongodb使用中的坑

- 数据量大时，数据迁移有坑
- 数据恢复能力差
- mongodb优点：最像关系型数据库的nosql

elasticSearch && mongodb对比
- elasticSearch value是json字符串
- mongo优势：可以存复杂的数据

用户手机注册验证码存取？

- 验证码用一次就丢，不建议持久化

- 建议session，或redis(设置过期时间)

mongo && redis对比

- mongo优势
  - 日志
    - 不规则日志：es, elk，如log4j打印的info，warn，error等
      - elk pattern，用多个正则去解析字符串，每种日志格式都需要编写正则去匹配
      - 用elk原因：非侵入式（不需要更改应用），跨平台、跨语言。在系统外围加一层将日志进行结构化存储
    - 规则日志：mongodb，日志持续增量(id + updatetime)，
      - 规则日志：结构化的日志，如用户行为日志，timeline，调用链路
  - 文件存储：GridFS（文件存储系统）
- redis优势：数据可以设置时效，适合分布式锁，用户登录token，数据库缓存中间件

mongo, hadoop, hbase定位

- mongo：结构化缓存，存储数据量有瓶颈
  - 日志数据库，文件存储系统
- hbase：基于列簇设计，扩展性，高可用性更好，数据存储数量级>>mongo
  - 通常用于大数据

mongo持久化

- 任何数据库都用文件实现持久化
- mongo数据文件存放在dbpath下

### 复习

- spring手写ORM框架

## 手写mongodb ORM框架

- `abstract BaseDaoSupport<T, PK>`模板模式
- `QueryRule`
- `QueryRuleBuilder`需要根据底层数据库进行不同实现

## 基于mongodb实现网络云盘

## mongodb高级应用

### 高可用

#### cluster

##### （Deprecated）M-S架构

Master <------------Client ----------------> Slave

##### M-A-S架构

Master <------------Client -----------------> slave

master <-----------Arbiter仲裁者 ----------------> slave

arbiter不存储数据，负责调度（**调度什么？**）

- 主从切换时，仲裁节点可以帮助选出新的master

###### 创建集群

- 创建master, slave, arbiter节点

- 配置集群

  > 必须先将数据库切换至admin库

- slave需要slaveok()

###### 集群故障

- arbiter故障：主从都可以承担artiber职能
- master故障：主从切换

##### 混合型

###### 节点

- route server
- config server
- shard server

###### 搭建集群

- 搭建shard集群

- - mongo连接任意一台服务器

  - 配置集群

    ```
    use admin;
    cfg={}
    rs.initiate(cfg);
    rs.status(); // 查看集群里机器状态，health=1表示node ok
    ```

    > shard server都是slave

- 搭建config集群

- 搭建路由服务器：关联shard集群及config集群
  - 关联config servers: `configdb=配置副本集/ip:port,ip:port...`
  - 启动mongos
  - 关联shard servers: 
    - `sh.addShard(数据副本集/任意ip:port)`
    - 激活shard
    - 数据分片`sh.shardCollection`

### 4.0新特性

