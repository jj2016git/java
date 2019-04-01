# 分布式协调系统zookeeper

## 一、zookeeper基本使用

### 1. 安装及CRUD

### 2. zookeeper集群搭建

#### 修改zoo.cfg

```
server.id=[ip | hostname]:port1:port2
```

port1: zookeeper client连接端口

port2: zookeeper选举端口

#### 创建${dataDir}/myid文件

>  **myid文件中的id必须和zoo.cfg中保持一致**

#### zookeeper配置解析

##### zoo.cfg

- tickTime: zookeeper时钟单位
- dataDir: 存储zookeeper存储事务日志、选举日志，pid等。。。。
- server.id=ip:通信端口:选举端口

## 二、zk名词

### 1. zookeeper集群及角色

集群数据同步：

- CAP理论（CP）
- 2pc实现最终一致性
- 事务请求必须落在leader上
  - 避免事务请求落到follower/observer上，必须由follower/observer发起数据同步的情况

> zk集群是有中心节点的设计
>
> - leader，follower/observer关系类似于数据库的主库和从库
> - 客户端请求处理类似于数据库读写分离的设计

![](E:\images\zookeeper_cluster.png)

### 2. 数据结构

类似于文件系统的树形数据结构

#### znode节点特性

- 同级节点唯一性
- 临时节点：会话结束后自动删除
  - `create -e`
  - 临时节点下不能存在子节点（防止自动删除临时节点时出问题）
- 有序节点：节点名为自动生成的递增序号
  - `create -s`

#### 节点操作

- 创建节点必须由根到子逐级创建
- 删除节点必须由子到根逐级删除

### 3. 会话

客户端与zookeeper从建立tcp连接到终止连接的过程

- 会话状态变化![](E:\images\zookeeper_session_states.png)

### 4. 节点stat信息

```
[zk: localhost:2181(CONNECTED) 1] get /foo/0000000001
1(value)
cZxid = 0x4(创建事务id)
ctime = Tue Oct 09 04:22:51 EDT 2018
mZxid = 0x4(更新事务id)
mtime = Tue Oct 09 04:22:51 EDT 2018
pZxid = 0x4(子节点变更事务id)
// for 乐观锁
cversion = 0(子节点版本号)
dataVersion = 0(value版本号)
aclVersion = 0(ACL版本号)
ephemeralOwner = 0x0(临时节点session标识)
dataLength = 1
numChildren = 0
```

### 5. watch机制

类似于发布/订阅

- client watch节点，

- 节点数据发生变化时触发事件，通知client

### 6. ACL权限控制

create, delete, read, write, admin

# 三、zk应用场景

### 1. 注册中心

- 服务地址维护
- 负载均衡
- 服务上下线动态感知

![](E:\images\zookeeper_registry.png)

1. 服务注册：
   - 订单服务节点上线时，与zookeeper建立会话，创建协议节点（临时节点）
   - 订单服务节点下线时，删除临时节点

> 利用临时节点会话结束自动删除特性，完成节点上下线自动感知

1. 服务发现：

   - 商品服务从zookeeper获取订单服务协议地址

   - 订单服务协议地址变化通知：商品服务订阅/OrderService，当协议节点下线时，收到zookeeper事件通知

2. 负载均衡？？？可能需要手动实现

### 2. 配置中心

实现动态配置

- watch实现配置变化的动态感知
- 安全机制保证安全
- 性能高

![](E:\images\配置中心.png)

### 3. 负载均衡

机器状态感知

选举master

e.g. kafka集群leader选举

1. kafka节点注册并监控
2. kafka leader选举：按照注册的节点顺序，序号最小的为leader
3. leader下线后，zk通知其他节点重新选举leader

![](E:\images\zk_kafka_epoch.png)

### 4. 分布式锁

协调服务节点访问共享资源的顺序









# 第二次课

## 分布式要求

Q1: 各节点数据一致性

S: 节点创建并监听znode, znode数据变更时zk通知各节点

---

Q2: 任务由一个节点执行

S: zk根据节点注册顺序，选出序号最小的节点作为执行任务的节点

---

Q3: 任务执行过程中，节点宕机，其他节点如何接替执行

<font color="#d0d0d">S: 节点宕机后，zk通知其他节点。其他节点查看任务状态是否需要接替执行，向zk发起请求，zk再次选出执行节点</font>

---

Q4: 共享资源访问互斥

S:  zk根据节点注册顺序决定节点访问共享资源顺序

## zk实现原理

Q1: 数据一致性

Q2: 故障恢复

### ZAB协议

#### 原子广播

#### 故障恢复

