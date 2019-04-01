# nginx

## nginx高可用

- nginx master && nginx backup
  - lvs, keepalive
  - 对外提供同一个虚拟ip

## what?

- 反向代理web服务器
  - 正向代理：代理客户端，n个client -> proxy -> server
  - 反向代理：代理服务端，client ->proxy->n个server
- web服务器
  - 静态服务器：apache, nginx
  - jsp/servlet服务器：tomcat, jetty

## how?

### 安装

### 启动、停止

### 配置nginx.conf

#### main

#### event

#### http

#### 虚拟主机配置

```
server {
    listen port;
    server_name 
}
```

- 基于ip的

- 基于port

  ```
  server {
      listen ${port}
      .....
  }
  ```

- 基于域名

  ```
  server {
      listen 80;
      server_name ask.gupaoedu.com;
      ....
  }
  ```

  > 测试时，需要修改本地hosts解析文件

#### location

##### 配置语法

`location[= | ~* | ^~]/uri/{...}`

##### 配置规则

location = /uri 精准匹配

location ^~ /uri 前缀匹配

location ~ /uri 正则匹配

location / 通用匹配

##### 规则优先级

```

```

精准匹配 -> 普通匹配（最长的匹配） -> 正则匹配

##### 实际使用建议

```
// 静态匹配主页面
location=/ {
    
}

// 通用匹配
location / {
    
}

// 静态资源匹配
location ~ \.(jpg | jpeg | gif | webp | png) {
    
}
```

#### nginx模块

##### 模块分类

- 核心
- 标准
- 第三方

##### ngx_http_core_module

- server
- location 实现uri到文件系统路径的映射
- error_page

##### nginx_http_access_module

- deny
- allow

##### 添加第三方模块

