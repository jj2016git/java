# 分布式缓存redis

## 缓存使用场景

- 多级缓存：内存缓存->分布式缓存->数据库

## 缓存

- 内存缓存：hashmap, ehcache
- 缓存中间件：memcache, redis

## redis

redis命令文档：<http://redisdoc.com/index.html>

redis安装：

- 解压
- make编译
- make test
- make install 安装

redis存储结构

![](E:\images\redis_struct.png)

redis数据类型

- string: 有长度、二进制安全的字符数组

  - String可以表达String, int, double三种类型

  - 二进制安全
    - 编码安全，C字符串是基于ASC编码
    - 不会将'\0'看作字符串的结束符
    - redis将字符串看作二进制字节流处理，保证**存进去什么，取出来什么**

  - sds

    - sds数据结构类型：sdshdr5, 8, 16, ....

      > sdshdrn: n表示字符串长度2^n-1, sdshdr5不使用

    - 根据value长度确定使用的sds数据结构类型

      | **len, alloc区别是什么？**len是字符串长度，alloc相当于capacity

      | **有len，为什么buf[]末尾还需要'\0'**

      - '\0'满足c语言对字符串定义，这样redis可以借用c的一些字符串方法

    - sds扩容

  - 使用场景

- list：双端链表

  - 3.2之前：linkedList, zipList

  - 3.2之后：quickList

    - 双向链表
    - 每个节点都是ziplist（压缩）或QuickListLZF（不压缩）

  - 使用场景

    - 分布式消息队列

      - 生产者lpush
      - 消费者brpop

      > brpop是rpop的阻塞版本，当list没有元素可pop时，会阻塞等待

    - 栈

      - lpush + lpop

    - 队列

      - lpush + rpop

- hash

  - field, value不支持嵌套类型
  - 数据结构：hashtable, ziplist
  - 应用场景：
    - 可以存对象

- set：无序、不重复
  - 数据结构：intset(整数元素按顺序存储，二分查找), hashtable(key, value(null))
  - 使用场景：
    - 用户标签
    - 共同好友：set做集合交并差

- 有序集合sortedSet:

  - 每个元素有一个score，元素按照score排序
  - 数据结构
    - ziplist
    - skiplist+hashtable()
      - skiplist变种二分法
      - skiplist查询、插入时间复杂度为O(logn)

skiplist

- 层高1~32，每层都是一个双向链表
- 每个元素插入前，通过随机算法计算得到层高l
- https://blog.csdn.net/lz710117239/article/details/78408919

```java
// 插入元素

```



### 过期时间

`expire key seconds`

`ttl key`：key的剩余过期时间

`persist key`：取消过期时间设置

`setex(String key, int seconds, String value)`

#### 原理

- 消极方法--惰性删除：访问key时删除过期key
- 积极方法--定期删除：定期随机删除一部分expired key
  - 目的：节省内存

|      | 惰性删除 | 定期删除 |
| ---- | -------- | -------- |
| PRO  | 不耗CPU  | 节省内存 |
| CON  | 浪费内存 | 消耗CPU  |

## 发布订阅

`publish channel message`

`subscribe channel`

- channel
  - 主题
    - 全匹配
    - 正则匹配
  - keyspace

> 消息发送出去不会持久化，如果发送之前没有订阅者，那么后续再有订阅者订阅该频道，之前的消息就收不到了 

### 持久化

#### RDB(快照)

符合save条件时，fork子进程，将内存中数据写入dump.rdb

##### save条件

- 配置规则

  ```
  # redis-server.conf
  # 满足规则时，执行bgsave
  save 900 1
  save 300 10
  save 60 10000
  ```

- save/bgsave

  - save会阻塞客户端请求
  - bgsave在后台异步进行快照操作

- flushall：清空rdb文件

- 执行SYNC复制操作（主从复制）

- shutdown：redis-server收到shutdown命令后，停止进程之前，保存数据

#### AOF（操作日志）

以实时操作日志的方式记录数据变更，redis每发生一次写操作，记录一下写日志

- appendfsync：操作系统何时将aof日志落盘
  - always：效率低
  - **everysec**：推荐，至多损失一秒数据
  - no：1. 数据存在缓冲区中不落盘导致数据丢失；2. 缓冲区满了之后阻塞写操作

- aof重写：解决aof文件不断增大问题
  - 将内存数据再写出来一遍

#### 对比

|      | rdb                                | aof                                                   |
| ---- | ---------------------------------- | ----------------------------------------------------- |
| pro  | 数据同步、故障数据恢复效率高       | 实时性好                                              |
| con  | 故障时，最近一次快照以后的数据丢失 | 1. 冗余操作记录，需要压缩aof；2. 每次操作都有记录消耗 |

> redis-server启动时载入数据文件
>
> 由于aof时效性比rdb要好，aof打开时，仅载入aof文件，aof关闭才会载入rdb

### 内存回收策略

- 默认不回收，内存溢出时直接报错
- allkeys-random/lru：lru适用于热点数据，randoms适用于数据频次比较平均
- volatile-random/lru/ttl: 针对带过期时间的key

### 单线程

- Reactor模式多路复用：单线程处理客户端请求

> 多路复用
>
> 1. 同步 && 异步：A调用B，同步就是需要B回应，异步就是不需要B回应
> 2. 阻塞 && 非阻塞：A调用B，阻塞就是A等待B干活，非阻塞就是A不等待B干活
> 3. 同步阻塞：在一次调用内，A一直等到B干完活给了回应
> 4. 同步非阻塞：在一次调用内，A不等待B干完，A一直轮询B是否完事儿
> 5. 异步阻塞IO/IO多路复用：A无需B回应，B干完活后通知A
>
> 用户进程空间：传输层之上
>
> 内核空间：传输层之下

### redis单线程原因

- redis性能瓶颈：内存、网络带宽

## redis线程安全问题

- 多个客户端同时修改同一个数据，最终结果无法保证

### pipeline

- 一次请求多个命令
- 局限性：多个命令之间不能存在依赖

### lua

- lua脚本中编写redis命令
  - `redis.call(command)`
- redis中执行lua脚本
  - `eval "return redis.call('get', ...)" ` 
  - `redis-cli --eval *.lua`
- lua保证原子性
  - 原理：
    - redis执行lua脚本时，会阻塞其他请求
    - script kill可以终止脚本执行
    - 如果脚本更新了数据，script kill无法终止，通过`shutdown nosave`回滚操作

## 分布式

> redis.conf bind <ip> 绑定本机可以接受访问的IP

### 主从

#### 配置

- slaveof

#### 数据同步

##### 原理

###### 全量复制（初始化）

- slave -> master: sync
- master: bgsave
- master->slave: send dump.rdb
- slave: load dump.rdb

###### 增量复制

- master->slave: 发送命令

```
replconf listening-port 6379
sync
```

- 数据一致性保证
  - master保证同步N个slave后，才能接收新的请求
    - min-slaves-to-write 每条写命令至少同步的slave个数
    - min-slaves-max-lag 允许slave最长连接丢失时间
  - slave重连后，如何增量同步master
    - backlog && offset

###### 无磁盘复制

- master不做磁盘复制：禁用rdb快照

- repl-diskless-sync 

#### 选主

- 哨兵
  - 监控master和slave是否正常运行
  - master down时，选举master

##### 选举过程

- 建立哨兵集群

sentinel sub(sentinel: hello)

新的sentinel pub(sentinel: hello, ....)

- raft协议，选举主sentinel

  - 一个节点有三种状态：
    - follower
    - leader
    - candicate
  - follower失去对leader的连接，变成candicate，向集群发起vote，集群中过半数节点同意，candidate-> leader
  - epoch 投票纪元

- sentinel配置

  sentinel monitor <master-name> <master-ip> <master-port> quorum

  - quorum: master主观下线 -> 客观下线，至少quorum个sentinel监控到master主观下线

### 数据分片

数据分片：一致性hash 客户端->访问代理->redis-server

- codis：
  - 分片路由
  - 动态扩容

#### redis-cluster

基于gossip协议的无中心化集群

redis cluster bus完成节点通信

- 数据分片--slot 0~16383
- 每个master指派slot，如M1 0~5000, M2 5001~10000，M3 10001~16383
- 数据路由：CRC16(key)%16383 = slot

> Q：希望数据落在同一台机器上，怎么处理？e.g. MSET key1, key2, key3
>
> A：HashTag？对key1, key2, key3相同的部分做hash
>
> Q：访问的key不在当前节点上，重定向
>
> A：返回MOVED <node_ip:port>

- 分片迁移 slot分配
  - 新增节点：从每个节点上取部分slot
    - 数据迁移
    - slot迁移
  - e.g. masterA 1,2,3 -> masterB 1,2,3
    - masterA状态-> migrating，masterB状态-> importing
    - masterA数据
      - 客户端访问的key未迁移，正常处理
      - key已迁移或不存在，回复ask，跳转到masterB
    - masterB数据
      - 客户端访问不是从ask跳转，不允许修改，返回moved

## 企业级集群方案

- codis
- twemproxy
- redis-cluster