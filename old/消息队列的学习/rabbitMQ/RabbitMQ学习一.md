# RabbitMQ学习

##### 一.RabbitMQ的基础

```java
1.未使用消息中间件可能存在的问题：
	a:业务调用链过长，用户等待时间长
	b:部分组件故障会瘫痪整个业务
	c:业务高峰期没有缓冲
  d:业务高峰期时产生大量的异步线程，造成线程池不够用或者内存爆满
2.使用消息中间件的优势：
  a:业务调用链短，用户等待时间短
  b:部分组件故障不会瘫痪整个业务
  c:业务高峰期有缓冲
  d:业务高峰期时不会产生大量的异步线程
3.使用消息中间件的作用
  a:异步处理
  b:系统解耦
  c:流量的削峰和流控
  d:消息广播
  e:消息收集
  f:最终一致性
```

##### 二.主流消息中间件技术

```java
1.ActiveMQ
  a:apache出品，java开发，支持JMS1.1协议和J2EE1.4规范。
  b:支持广泛的连接协议（OpenWrie/STOMP/REST/XMPP/AMQP）
  C:支持多种语言的客户端，支持插件
  d:优点（基于JAVA,跨平台运行；可以用JDBC连接多种数据库；有完善的界面、监控、安全机制；自动重连和错误重试）缺点（社区活跃度不及rabbitMQ；重心在6.0产品Apollo,对5的Bug维护较少；不适合用于上千个队列的应用场景）
2.RabbitMQ
  a:当前最主流的消息中间件
  b:高可靠性，支持发送确认，投递确认等特性
  c:高可用，支持镜像队列
  d:优点（基于Erlang，支持高并发；支持多种平台，多种客户端，文档齐全；可靠性高；在互联网公司有较大规模的应用，社区活跃度高）缺点（Erlang语言较为小众，不利于二次开发；代理架构下，中央节点增加了延迟，影响性能；使用AMQP协议，使用起来有学习成本）
3.RocketMQ
  a:阿里巴巴团队开发，经受过双十一的考验
  b:能够保证严格的消息顺序
  c:亿级消息堆积能力
  d:丰富的消息拉取模式
  e:优点（基于java,方便二次开发；单机支持1万以上持久化队列；内存和磁盘都有一份数据，保证性能+高可用；开发度较活跃，版本更新更快）缺点（客户端种类不多，较成熟的是java及C++；没有web管理界面，提供一个CLI命令行界面；社区关注度及成熟度不如RabbitMQ）
4.KafkaMQ
  a:LinkedIn开发的分布式的日志提交系统
  b:独特的分区特性，适用于大数据系统
  c:性能高效，可扩展性良好
  d:可复制，可容错
  e:优点（原生的分布式系统；零拷贝技术，减少IO操作步骤，提高系统吞吐量；快速持久化，可以再o(1)的系统开销下进行消息持久化；支持数据批量发送和拉取）缺点（单机超过64个队列/分区时，性能明显劣化；使用短轮询方式，实时性取决于轮询间隔时间；消费失败时不支持重试；可靠性比较差）
5.总结：
  a:ActiveMQ最”老“，老牌，但维护较慢
  b:RabbitMQ最”火“，适合大小公司，各种场景通杀
  c:RocketMQ最“猛”，功能强，但考验公司运维能力
  d:KafkaMQ最“最强”，支持超大量数据，但消息可靠性弱
```

##### 三.RabbitMQ高性能的原因

```java
1.编写语言Erlang
  a:通用的面向并发的变成语言，适用于分布式系统
  b:基于虚拟机解释运行，跨平台部署
  c:进程间上下文切换效率远高于C语言
  d:网络性能有着和原生Socket一样的延迟
2.总结
  a:RabbitMQ底层使用Erlang实现，天然具有高性能基因
  b:RabbitMQ在互联网和金融领域都有广泛的应用
```

##### 四.AMQP协议

![AMQP协议.png](https://i.loli.net/2021/02/07/geXjKFmtnE8TU3Y.png)

```java
1.Broker:接受和分发消息的应用，RabbitMQ就是MessageBroker.
2.Virtual Host：虚拟Broker,将多个单元隔离开.
3.Connection:publisher/consumer和broker之间的TCP连接
4.Channel:connection内部建立的逻辑连接，通常每个线程创建单独的channel
5.Routing Key:路由键，用来指示消息的路由转发，相当于快递的地址
6.Exchange:交换机，相当于快递的分拨中心
7.Queue:消息队列，消息最终被送到这里等待consumer取走
8.Binding:exchange和queue之间的虚拟连接，用于message的分发依据
9.AMQP协议或者RabbitaMQ实现中，最核心的组件是Exchange
10.Exchange承担RabbitMQ的核心功能===路由转发
11.Exchange有多个种类，配置多变，需要详细讲解
```

##### 五.RabbitMQ的心脏--Exchange

```java
1.Exchange的基本概念
  a:Exchange是AMQP协议和RabbitMQ的核心组件
  b:Exchange的功能是根据绑定关系和路由键为消息提供路由，将消息转发至相应的队列
  c:Exchange有4种类型：Direct/Topic/Fanout/Headers,其中Headers使用很少，以前三种为主
2.Direct Exchange
  a:Message中的Routing Key如果和Binding Key一致，Direct Exchange则将message发到对应的queue中
3.Fanout Exchange
  a:每个发到Fanout Exchange的message都会分发到所有绑定的queue上去
4.Topit Exchange
  a:根据Routing Key以及通配规则，Topic Exchange将消息分发到目标Queue中
  b:如果为全匹配，则和Direct类似
  c:Binding Key中的#:匹配任意个数的word
  d:Binding Key中的*:匹配任意1个word
```

##### 六.RabbitMQ的快速安装

```java
1.docker安装
	docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
2.网页端管理工具
  localhost:15672   guest/guest
3.总结
  a:管控台是RabbitMQ开发和管理的必备工具
  b:管控台有配置，验证，监控等多种功能
  c:在业务开发中，要注意利用管控台，有效提升开发效率
4.命令行工具
  a:想看什么就用list_什么
  b:想删除什么就用delete_什么
  c:想清空什么就用purge_什么
  d:一切问题就看--help
```