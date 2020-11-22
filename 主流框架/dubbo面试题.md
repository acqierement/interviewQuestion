# dubbo面试题

## Dubbo 源码读过吗？没有

## Dubbo的核心组件以及功能

- Provider：服务的提供方
- Consumer：调用远程服务的服务消费方
- Registry：服务注册和发现的注册中心
- Monitor：统计服务调用次数和调用时间的监控中心
- Container：服务运行容器

## 注册中心宕机怎么办

启动dubbo时，消费者会从zk拉取注册的生产者的地址接口等数据，缓存在本地。每次调用时，按照本地存储的地址进行调用。但是在注册中心全部挂掉后增加新的提供者，则不能被消费者发现：

## 服务是怎么注册上去的？

Provider(服务提供者)绑定指定端口并启动服务

Provider 连接注册中心，将本机 IP、端口、应用信息和提供服务信息发送至注册中心存储

## 你们用的什么注册中心？

- Zookeeper 注册中心： 基于分布式协调系统 Zookeeper 实现，采用 Zookeeper 的 watch 机制实现数据变更(官方推荐)

- Multicast 注册中心： 基于网络中组播传输实现，不需要任何中心节点，只要广播地址，就能进行服务注册和发现
- Redis 注册中心： 基于 Redis 实现，采用 key/Map 数据结构存储，主 key 存储服务名和类型，Map 中 key 存储服务 URL，Map 中 value 存储服务过期时间，基于 Redis 的发布/订阅模式通知数据变更
- Simple 注册中心：一个普通的 Dubbo 服务，可以减少第三方依赖，使整体通讯方式一致，不支持集群

## 讲 Dubbo SPI 的源码？

## 讲 Dubbo 服务暴露源码 + Dubbo 服务注册 

## 讲一下 Dubbo 服务引用底层？

开始一直以为是动态***调用 invoker 模型，结果说不够底层原理问 RPC 协议。

## 讲一下 Dubbo RPC 协议调用过程，使用哪些协议？  

## Dubbo 的连接

这里问的贼细，全靠推理推出来的。。

## Dubbo 的负载均衡

## 为什么不选 SpringCloud，而选 Dubbo？

**Dubbo 专注 RPC 和服务治理，Spring Cloud 则是一个微服务架构生态。**

## Dubbo原理

## RPC分为哪几部分

