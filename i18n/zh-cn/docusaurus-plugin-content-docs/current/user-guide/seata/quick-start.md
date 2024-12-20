---
title: 快速开始
keywords: [Spring Cloud Alibaba]
description: Quick Start.
---

### 如何使用seata

引入依赖：
```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

项目提供了相关实例： [Example](https://github.com/alibaba/spring-cloud-alibaba/tree/2.2.x/spring-cloud-alibaba-examples/seata-example)

1. 保证本地已经运行 `nacos-server` 在 `8848` 端口。
2. 配置数据库，将 `account-server`、`order-service`、`storage-service` 这三个应用中的 resources 目录下的 `application.yml` 文件中的如下配置修改成你运行环境中的实际配置：

```properties
base:
  config:
    mdb:
      hostname: 127.0.0.1 #your mysql server ip address
      dbname: seata #your database name for test
      port: 3306 #your mysql server listening port
      username: root #your mysql server username
      password: root #your mysql server password
```
3. 创建数据库表，可根据 `spring-cloud-alibaba-examples/seata-example` 下的 `all.sql` 快速操作：
    - 根据选择的事务模式，创建 [事务日志表](https://github.com/seata/seata/tree/develop/script/client)。比如默认为AT模式，需要使用到 undo_log 表，则进入 at/db 下选择对应的数据库脚本执行。
    - 创建 seata-server db模式所需要的 [状态记录表](https://github.com/seata/seata/tree/develop/script/server/db) ，包括 `global_table` 、`branch_table` 、`lock_table` 、`distributed_lock`。
    - 创建 `spring-cloud-alibaba-examples/seata-example` 示例中所需的数据库表。
4. 创建 `spring-cloud-alibaba-examples/seata-example` 所需要的Nacos配置。  
DataId: seata.properties，Group: SEATA_ Group(Seata 1.5.1 默认组)  
配置内容：
```properties
service.vgroupMapping.order-service-tx-group=default
service.vgroupMapping.account-service-tx-group=default
service.vgroupMapping.business-service-tx-group=default
service.vgroupMapping.storage-service-tx-group=default
```

5. 启动 Seata Server，有SpringBoot 和下载server两种方式：
    - 运行 `spring-cloud-alibaba-examples/seata-example` 下的seata-server， 启动Seata server。
    - 根据seata官方提供的 [seata-server.jar](https://seata.io/zh-cn/docs/ops/deploy-guide-beginner.html)启动Seata Server。
6. 在本地启动 `spring-cloud-alibaba-examples/seata-example` 文件夹下的子服务 `account-service` ， `order-service` ， `storage-service` ，最后启动全局事务控制服务 `business-service`。
7. 启动示例后，通过 HTTP 的 GET 方法访问如下 URL，可以分别验证在 `business-service` 中 通过 RestTemplate 和 FeignClient 调用其他服务的场景：

   http://127.0.0.1:18081/seata/feign  
   http://127.0.0.1:18081/seata/rest

### Seata Dashboard

- Seata 1.5.1支持Seata控制台本地访问控制台地址：`http://127.0.0.1:7091`。
- 通过Seata控制台可以观察到正在执行的事务信息和全局锁信息，并且在事务完成时删除相关信息。


### 如何验证分布式事务的成功？
#### Xid 信息是否成功传递

在 `account-server`、`order-service` 和 `storage-service` 三个 服务的 Controller 中，第一个执行的逻辑都是输出 RootContext 中的 Xid 信息，如果看到都输出了正确的 Xid 信息，即每次都发生变化，且同一次调用中所有服务的 Xid 都一致。则表明 Seata 的 Xid 的传递和还原是正常的。

#### 数据库中数据的一致性

在这个例子中，我们模拟一个用户购买商品的场景，StorageService 负责扣除库存数量，OrderService 负责保存订单，BusinessService 负责扣除用户账户余额，AccountService 负责更新账号的余额，并作为全局事务控制入口。
为了演示示例，我们在 OrderService 和 AccountService 中使用了 Random。NextBoolean() 随机抛出异常，模拟调用服务时随机发生异常的场景。

如果分布式事务是有效的，那么下面的等式应该是正确的：
- 用户原始金额（1000）=用户现有金额+商品单价（2）*订单数量*每个订单的商品数量（2）
- 初始商品数量 (100) = 现有商品数量 + 订单数量 * 每个订单的商品数量 (2)

### Spring Cloud 支持点
- 通过 Spring MVC 提供服务的服务提供者在收到 HTTP 请求时，可以自动恢复 Seata 上下文，该请求在 header 中包含 Seata 信息。
- 当服务调用者通过 RestTemplate 调用时，支持自动传递 Seata 上下文。
- 当服务调用者通过 FeignClient 调用时，支持自动传递 Seata 上下文。
- 支持同时使用 SeataClient 和 Hystrix 的场景。
- 支持 SeataClient 和 Sentinel 使用的场景。