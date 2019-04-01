## nginx反向代理

```
proxy_pass http://ip:port;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For 
```

## 负载均衡

### upstream

```
upstream GroupName {
    server ip1:port1 weight=5 max_fails=2 fails_timeout=600s;
    server ip2:port2;
}
```

- 负载均衡策略
  - 默认轮询，后端服务器宕机后，自动剔除
  - ip_hash
    - 缺点：如果nginx前还有一个前置服务器，ip是前置服务器的ip
  - 权重轮询
  - 。。。
- proxy_next_upstream error timeout http_XXX
- proxy_connect_timeout：nginx与后端服务器连接超时时间
- proxy_send_timeout：
- proxy_read_timeout：

## nginx动静分离

### 缓存

自动加etag, last-modified

### 压缩

gzip

- http meta-data `content-encoding: gzip`

## 防盗链

图片只能在自己网站里访问

## 跨域访问

## nginx进程模型

多进程 + 多路复用

1个master + n个worker

linux下io默认epoll，可设置

> 为什么是多进程，而不是多线程

## nginx高性能、高可用方案

### 负载均衡

- DNS轮询：多个ip对应同一个域名，DNS随机返回一个ip给client
- nginx吞吐量>> tomcat

### nginx单点问题

- 主备：keepalived管理工具
  - 不能扩展nginx吞吐量
- 扩展吞吐量： F5 -> nginx集群

#### keepalived

轻量级的高可用解决方案

lvs四层负载均衡软件

keepalived监控lvs集群中各个节点的存活状态

VRRP（虚拟路由冗余协议）：两台机器对外一个ip，对内多个

> Camel(大众点评软件负载方案)

#### 实践

- 安装keepalived

  - keepalived之间通过组播进行联系

- nginx宕机时，keepalived也需要die

  ```shell
  // keepalived.conf
  global_defs {
      router_id ...
      enable_script_security
  }
  
  vrrp_script_nginx_status_process {
      script "nginx_status_check.sh"
      user root
      interval 3
  }
  
  vrrp_instance_VI_1 {
      ...
      track_script {
          nginx_status_process
      }
  }
  ```

## openresty

nginx + lua

### 网关

#### what?

- 网关 -> 一个网络向另一个网络发送消息
- api gateway -> 
- 开放服务（open api）

#### why?

网关作用

- 鉴权
- 限流
- 灰度
- 日志
- 分流

#### how?

```
# ;;表示默认路径
lua_package_path 
"$prefix/lualib/?.lua;;";
lua_package_cpath "$prefix/lualib/?.so;;";


```



