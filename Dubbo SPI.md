## Dubbo SPI

### java SPI

e.g.

SPI:  `java.sql.Driver`

SPI实现类：`com.mysql.cj.jdbc.Driver`

SPI实现类配置文件：`mysql-connector-java-6.0.6.jar\META-INF\services\java.sql.Driver`	

```
// 实现类jar包/META-INF/services/SPI接口名
com.mysql.cj.jdbc.Driver // 实现类全类名
```

### dubbo spi

#### 改进java spi

- 按需扫描
- 提供SPI实现类IoC和AOP

#### 实现例子

```
private static final Protocol protocol = ExtensionLoader.
            getExtensionLoader(Protocol.class).
            getAdaptiveExtension(); //Protocol$Adaptive
```

`com.alibaba.dubbo.common.extension.ExtensionLoader#getExtensionLoader`

`com.alibaba.dubbo.common.extension.ExtensionLoader#getAdaptiveExtension`