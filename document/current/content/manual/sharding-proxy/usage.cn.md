+++
pre = "<b>4.2.1. </b>"
toc = true
title = "使用手册"
weight = 1
+++

## Proxy启动

1. 下载Sharding-Proxy的最新发行版。
1. 如果使用docker，可以执行`docker pull shardingsphere/sharding-proxy`获取镜像。详细信息请参考[Docker镜像](/cn/manual/sharding-proxy/docker/)。
1. 解压缩后修改conf/server.yaml和以config-前缀开头的文件，如：conf/config-xxx.yaml文件，进行分片规则、读写分离规则配置. 配置方式请参考[配置手册](/cn/manual/sharding-proxy/configuration/)。
1. Linux操作系统请运行`bin/start.sh`，Windows操作系统请运行`bin/start.bat`启动Sharding-Proxy。如需配置启动端口、配置文件位置，可参考[快速入门](/cn/quick-start/sharding-proxy-quick-start/)
进行启动。
1. 使用任何PostgreSQL的客户端连接。如: `psql -U root -h 127.0.0.1 -p 3307`

## 注册中心使用

若想使用Sharding-Proxy的数据库治理功能，则需要使用注册中心实现实例熔断和从库禁用功能。详情请参考[支持的注册中心](/cn/features/orchestration/supported-registry-repo/)。

### Zookeeper

1. Sharding-Proxy默认提供了Zookeeper的注册中心解决方案。您只需按照[配置规则](/cn/manual/sharding-proxy/configuration/)进行注册中心的配置，即可使用。

### Etcd

1. 将Sharding-Proxy的lib目录下的`sharding-orchestration-reg-zookeeper-curator-${sharding-sphere.version}.jar`文件删除。
1. Maven仓库下载Etcd解决方案的[最新稳定版](http://central.maven.org/maven2/io/shardingsphere/sharding-orchestration-reg-etcd/)jar包。
1. 将下载下来的jar包放到Sharding-Proxy的lib目录下。
1. 按照[配置规则](/cn/manual/sharding-proxy/configuration/)进行注册中心的配置，即可使用。

### 其他第三方注册中心

1. 将Sharding-Proxy的lib目录下的`sharding-orchestration-reg-zookeeper-curator-${sharding-sphere.version}.jar`文件删除。
1. 使用SPI方式实现相关逻辑编码，并将生成的jar包放到Sharding-Proxy的lib目录下。
1. 按照[配置规则](/cn/manual/sharding-proxy/configuration/)进行注册中心的配置，即可使用。

## 使用自定义分片算法

当用户需要使用自定义的分片算法类时，无法再通过简单的inline表达式在yaml文件进行配置。可通过以下方式配置使用自定义分片算法。

1. 实现ShardingAlgorithm接口定义的算法实现类。
1. 将上述java文件打包成jar包。
1. 将上述jar包拷贝至ShardingProxy解压后的conf/lib目录下。
1. 将上述自定义算法实现类的java文件引用配置在yaml文件里tableRule的`algorithmClassName`属性上，具体可参考[配置规则](/cn/manual/sharding-proxy/configuration/)。

## 分布式事务

Sharding-Proxy接入的分布式事务API同Sharding-JDBC保持一致，支持LOCAL，XA，BASE类型的事务。

### XA事务

Sharding-Proxy原生支持XA事务，默认的事务管理器为Atomikos。
可以通过在Sharding-Proxy的conf目录中添加`jta.properties`来定制化Atomikos配置项。
具体的配置规则请参考Atomikos的[官方文档](https://www.atomikos.com/Documentation/JtaProperties)。

### BASE事务

BASE目前没有打包到Sharding-Proxy中，使用时需要将实现了`ShardingTransactionManager`SPI的jar拷贝至conf/lib目录，然后切换事务类型为BASE。

## SCTL (Sharding-Proxy control language)

SCTL为Sharding-Proxy特有的控制语句，可以在运行时修改和查询Sharding-Proxy的状态，目前支持的语法为：

| 语句                         | 说明                                                                           |
|:-----------------------------|:-------------------------------------------------------------------------------|
|sctl:set transaction_type=XX  | 修改当前TCP连接的事务类型, 支持LOCAL，XA，BASE。例：sctl:set transaction_type=XA |
|sctl:show transaction_type    | 查询当前TCP连接的事务类型                                                       |
|sctl:show cached_connections  | 查询当前TCP连接中缓存的物理数据库连接个数                                        |
|sctl:explain SQL语句          | 查看逻辑SQL的执行计划，例：sctl:explain select * from t_order;                  |


## 注意事项

1. Sharding-Proxy默认使用3307端口，可以通过启动脚本追加参数作为启动端口号。如: `bin/start.sh 3308`
1. Sharding-Proxy使用conf/server.yaml配置注册中心、认证信息以及公用属性。
1. Sharding-Proxy支持多逻辑数据源，每个以config-前缀命名的yaml配置文件，即为一个逻辑数据源。
