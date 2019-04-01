[TOC]

# zookeeper

## what?

### 分布式协调系统

- 使用场景
  - 分布式锁
  - 注册中心

### 特点

#### 存储特点

- 树形数据结构
- znode:
  - 同级唯一性
  - 有序节点
  - 临时节点

#### watch机制

节点发生变化时，触发事件通知

e.g.

- 分布式锁，N个对象竞争同一把锁，锁释放时对象可以获得通知
- 注册中心：服务A调用服务B的某个节点，如果服务B节点宕机，服务A可以获得通知

## why?通过zk实现分布式协调原理剖析

分布式协调服务需求

- 不存在单点问题：有中心集群（leader, follower, observer）

  > 为什么采用有中心集群：降低数据同步等复杂度
  >
  > Redis无中心化集群：各对等节点不会保存同一份数据。对等节点通过Gossip完成节点通信

- 不存在集群中节点数据不一致问题：ZAB协议

### ZAB协议

#### 原子广播：实现数据同步

![](E:\images\zk_request.png)

非事务请求：

client ---> 任意node ---> client

e.g. ZooKeeper#getData()

事务请求：

client ---> 任意node ---> leader

leader ---(proposal)---> participants

participants记录txnLog, 记录成功后，发送ack给leader

leader收到过半数ack, 发送commit给所有participants

#### 源码分析

![](E:\images\zk_request_process.png)

#### 崩溃恢复：leader选举

![](E:\images\zookeeper leader选举.png)

1. 初始选票：<myid, lastLoggedZxid, peerEpoch>
2. 逻辑时钟/electionEpoch：用于leader选举
3. peerEpoch: leader选举出来后，zk集群进入新的epoch
4. 选票优先级：保证候选人是存活最久的节点
   (1) 比较peerEpoch; 
   (2) peerEpoch相同，比较lastLoggedZxid
   (3) lastLoggedZxid相同，比较serverid
5. lastLoggedZxid：最后被记录txnLog的事务id
6. lastProcessedZxid: 最后提交的事务id
   (1) 节点启动，从snapshot文件获取zxid，可能丢失部分已提交未保存快照的事务
   (2) 节点运行中，事务提交并成功执行后，根据执行结果更新

> leader选举原理：
>
> 1. 根据lastLoggedZxid选出活得最久的participant
> 2. leader选举成功后，新leader会复核自己的leader身份，然后向其他成员发起数据同步，一切ok后开始接收新请求
> 3. participant接收到proposal，就会向txnLog写一条记录，不定期将内存中的值写入snapshot。
> 4. participant在commit之后，才会处理数据，并载入内存 ==> snapshot中记录的一定是处理成功的数据

### watch机制

可以注册watcher的方法

- exist
- getData
- getChildren

事件

- NONE
- NodeCreated
- NodeDeleted
- NodeDataChanged
- NodeChildrenChanged

验证什么操作会触发什么方法的watcher, 产生什么事件？

|                      | zk-foo监听          | 方法           | zk-foo/child监听 | 方法           |
| -------------------- | ------------------- | -------------- | ---------------- | -------------- |
| create zk-foo        | NodeCreated         | exist, getData |                  |                |
| setData zk-foo       | NodeDataChanged     | exist, getData |                  |                |
| delete zk-foo        | NodeDeleted         | exist, getData |                  |                |
| create zk-foo/child  | NodeChildrenChanged | getChildren    | NodeCreated      | exist, getData |
| setData zk-foo/child |                     |                | NodeDataChanged  | exist, getData |
| delete zk-foo/child  | NodeChildrenChanged | getChildren    | NodeDeleted      | exist, getData |

#### 原理

1. 注册watcher, e.g. exist()

   ![](E:\images\zookeeper_watcher.png)

   - 服务端watcher-- serverCnxn: 触发后，生成事件，包装成notification发送给client
   - 流程
     - 发起请求，watch=true，服务端看到watch=true后，注册<path, ServerCnxn>
     - 服务端执行完请求，回应客户端
     - 客户端接收到回应后，从等待队列中取出该请求
     - 客户端注册请求的<path, watcher>

2. 触发watcher:

   - 服务端执行事务请求，触发watcher,即ServerCnxn
   - 客户端对收到的notification，取出事件，加入EventThread
   - EventThread取出监听器，触发执行

![](E:\images\zk_watcher.png)

## how?如何使用zk

### CRUD

#### 原生API

#### Curator API

### 分布式锁

#### 实现原理

##### 原生

##### Curator及API源码分析

### 注册中心

#### 实现原理









