# Kafka的学习

##### 环境准备

```java
1.虚拟机安装centos7+
2.使用ssh工具连接至宿主机
```

##### 初识Kafka

```java
1.自我介绍
	A distributed streaming platform 分布式的流平台
	Kafka是基于zookeeper的分布式消息系统
	Kafka具有高吞吐率，高性能，实时及高可靠等特点
```

##### 安装Kafka

```java
1.连接虚拟机
2.安装lrzsz工具  yum install -y lrzsz
3.解压JDK，zookeeper,kafka压缩包
4.安装JDK,修改环境变量
5.配置zookeeper配置文件conf,启动zookeeper,测试连接
6.配置Kafka配置文件
7.启动kafka bin/kafka-server-start.sh config/server.properties &
```

##### Kafka基本概念

```java
1.Topic：一个虚拟的概念，由1到多个Partitions组成
2.Partiton：实际的消息存储单位
3.Producer：消息的生产者
4.Consumer：消息的消费者
```

##### Kafka客户端基本操作（JAVA版）

```java
1.客户端操作的4种API
	a.Producers生产者的API：发布消息到1个或者多个topic
	b.Consumers消费者的API：订阅一个或者多个topic,并处理产生的消息
	c.StreamProcessors流处理的API：高效的将输入流转换为输出流
	d.Connectors连接DB的API：从一些原系统或者应用程序中拉取数据到kafka
	e.AdminClient管理Kafka的API:允许管理和检测Topic、broker以及其他Kafka对象
2.初始化kafka工程
```

##### Kafka AdminClient API

```java
1.AdminClient:AdminClient客户端对象
2.NewTopic:创建Topic
3.CreateTopicsResult:创建Topic的返回结果
4.ListTopicResult:查询Topic列表
5.ListTopicsOptions:查询Topic列表及选项
6.DescribeTopicsResult:查询Topics
7.DescribeConfigsResult:查询Topics配置项
```

##### Kafka AdminClient API instance

```java
package com.luoye.study.kafka.admin;

import org.apache.kafka.clients.admin.*;
import org.apache.kafka.common.config.ConfigResource;

import java.util.*;
import java.util.concurrent.ExecutionException;

/**
 * @program: kafka
 * @Author: yangbai
 * @Date: 2020/10/13 8:38 PM
 * @Version: 1.0
 * @Description: AdminClient 实体类
 */
public class AdminSimple {


    private final static String TOPIC_NAME = "luoye-topic";

    public static void main(String[] args) throws Exception {

        //创建Topic
//        createTopic();
        //获取Topic
//        topicLists();
        //删除Topic
//        deleteTopic();
        //获取一个topic的描述信息
        describeTopic();

        //修改一个Topic的配置信息
//        alterConfigs();
        //获取一个topic的配置信息
//        describeConfig();
        //为一个Topic增加partitions
//        incrPartitions(2);
    }


    /**
     * 为一个Topic增加partitions
     * @param partitions
     * @throws ExecutionException
     * @throws InterruptedException
     */
    public static  void incrPartitions(int partitions) throws ExecutionException, InterruptedException {
        AdminClient adminClient = adminClient();
        Map<String, NewPartitions> newPartitions = new HashMap<>();

        newPartitions.put(TOPIC_NAME, NewPartitions.increaseTo(partitions));
        CreatePartitionsResult createpartitionsresult = adminClient.createPartitions(newPartitions);

        createpartitionsresult.all().get();
    }


    /**
     * 修改一个Topic的配置信息(两种方法)
     *
     * @throws ExecutionException
     * @throws InterruptedException
     */
    public static void alterConfigs() throws ExecutionException, InterruptedException {
        //旧版本的方法
//        AdminClient adminClient = adminClient();
//        Map<ConfigResource, Config> configs = new HashMap<>();
//        ConfigResource configResource = new ConfigResource(ConfigResource.Type.TOPIC, TOPIC_NAME);
//        Config config = new Config(Arrays.asList(new ConfigEntry("preallocate","true")));
//        configs.put(configResource, config);
//
//        AlterConfigsResult alterConfigsResult = adminClient.alterConfigs(configs);
//        alterConfigsResult.all().get();
        //新版本的方法
        AdminClient adminClient = adminClient();

        Map<ConfigResource, Collection<AlterConfigOp>> map = new HashMap<>();
        ConfigResource configResource = new ConfigResource(ConfigResource.Type.TOPIC, TOPIC_NAME);

        AlterConfigOp alterConfigOp = new AlterConfigOp(new ConfigEntry("preallocate", "true"), AlterConfigOp.OpType.SET);
        map.put(configResource, Arrays.asList(alterConfigOp));
        AlterConfigsResult alterConfigsResult = adminClient.incrementalAlterConfigs(map);
        alterConfigsResult.all().get();

    }

    /**
     * 获取一个topic的配置信息
     */
    public static void describeConfig() {
        AdminClient adminClient = adminClient();

        ConfigResource configResource = new ConfigResource(ConfigResource.Type.TOPIC, TOPIC_NAME);
        DescribeConfigsResult describeConfigsResult = adminClient.describeConfigs(Arrays.asList(configResource));
        try {
            Map<ConfigResource, Config> configResourceConfigMap = describeConfigsResult.all().get();
            System.out.println("configResourceConfigMap: " + configResourceConfigMap);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取指定名字Topic的描述信息
     */
    public static void describeTopic() {
        AdminClient adminClient = adminClient();

        DescribeTopicsResult describeTopicsResult = adminClient.describeTopics(Arrays.asList("luoye-topic"));
        try {
            Map<String, TopicDescription> stringTopicDescriptionMap = describeTopicsResult.all().get();
            System.out.println("stringTopicDescriptionMap: " + stringTopicDescriptionMap);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    /**
     * 删除Topic
     *
     * @throws ExecutionException
     * @throws InterruptedException
     */
    public static void deleteTopic() throws ExecutionException, InterruptedException {
        AdminClient adminClient = adminClient();

        DeleteTopicsResult deleteTopicsResult = adminClient.deleteTopics(Arrays.asList(TOPIC_NAME));

        System.out.println("deleteTopicsResult:" + deleteTopicsResult.all().get());
    }


    /**
     * 获取Topics列表
     *
     * @throws ExecutionException
     * @throws InterruptedException
     */
    public static void topicLists() throws ExecutionException, InterruptedException {

        AdminClient adminClient = adminClient();
        ListTopicsOptions listTopicsOptions = new ListTopicsOptions();
        //是否查看internal
        listTopicsOptions.listInternal(true);
        ListTopicsResult listTopicsResult = adminClient.listTopics(listTopicsOptions);
        Set<String> names = listTopicsResult.names().get();
        Collection<TopicListing> topicListings = listTopicsResult.listings().get();
        names.stream().forEach(System.out::println);

        topicListings.stream().forEach((topiclist) -> {
            System.out.println(topiclist);
        });

    }

    /**
     * 创建Topic实例
     *
     * @throws Exception
     */
    public static void createTopic() throws Exception {
        AdminClient adminClient = adminClient();
        Short rs = 1;
        NewTopic newTopic = new NewTopic(TOPIC_NAME, 1, rs);

        CreateTopicsResult topics = adminClient.createTopics(Arrays.asList(newTopic));
        System.out.println(topics);

    }


    /**
     * 设置AdminClient
     *
     * @return
     */
    public static AdminClient adminClient() {

        Properties properties = new Properties();
        properties.setProperty(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "10.211.55.3:9092");

        AdminClient adminClient = AdminClient.create(properties);

        return adminClient;
    }
}
```

##### Kafka Producer APIS

```java
Producer需要掌握的知识：
	1.掌握Producer APIS
	2.了解producer 各项重点配置
	3.熟练Producer负载均衡等高级特性
Producer发送模式：
  1.同步发送
  2.异步发送
  3.异步回调发送
主要构建对象KafkaProducer:
	1.MetricConfig:Kafka监控用提交一些指标。
  2.加载负载均衡器。
  3.初始化Serializer
  4.初始化RecordAccumulator:类似计数器
  5.启动newSender:守护线程
发送消息：producer.send(record)
  1.计算分区：计算发送消息到那个partition
  2.计算批次：accumulator.append()
  3.主要内容：a.创建批次；b.向批次中追加消息。
源码解析：
  1.producer是线程安全的。
  2.producer并不是一条一条发送消息的。
  3.producer是消息逐条累加（append）,达到设置的大小后批量发送的。
Producer发送原理解析：
  1.直接发送
  2.负载均衡(自定义partition)
  3.异步发送
消息传递保障机制（三种）：
  1.最多一次：收到0到1次
  2.至少一次：收到1到多次
  3.正好一次：只收到1次
  4.依赖保障机制依赖于Producer和Consumer共同实现，主要依赖于Producer,受作用于参数（ACKS_CONFIG）
```

##### Kafka Producer APIS instance

```java
package com.luoye.study.kafka.producer;

import org.apache.kafka.clients.producer.*;

import java.util.Properties;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

/**
 * @program: kafka
 * @Author: yangbai
 * @Date: 2020/10/15 11:02 AM
 * @Version: 1.0
 * @Description: 生产者实例
 */
public class ProducerSample {

    private final static String TOPIC_NAME = "luoye-topic";

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        //Producer异步发送演示
//        producerSend();
        //Producer异步阻塞发送演示
//        producerSycSend();
        //Producer异步发送回调演示
//        producerSendCallBack();
        //Producer异步发送回调自定义Partition演示
        producerSendCallBackPartition();
    }

    /**
     * Producer异步发送演示
     */
    public static void producerSend() {
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "10.211.55.3:9092");
        properties.put(ProducerConfig.ACKS_CONFIG, "all");
        properties.put(ProducerConfig.RETRIES_CONFIG, "0");
        properties.put(ProducerConfig.BATCH_SIZE_CONFIG, "16384");
        properties.put(ProducerConfig.LINGER_MS_CONFIG, "1");
        properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG, "33554432");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");

        //Producer的主对象
        Producer<String, String> producer = new KafkaProducer<String, String>(properties);

        //消息对象ProducerRecord
        for (int i = 0; i < 10; i++) {
            ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>(TOPIC_NAME, "key-" + i, "value-" + i);
            producer.send(producerRecord);

        }
       //所有打开的通道都要关闭
        producer.close();
    }


    /**
     * Producer异步阻塞发送演示
     */
    public static void producerSycSend() throws ExecutionException, InterruptedException {
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "10.211.55.3:9092");
        properties.put(ProducerConfig.ACKS_CONFIG, "all");
        properties.put(ProducerConfig.RETRIES_CONFIG, "0");
        properties.put(ProducerConfig.BATCH_SIZE_CONFIG, "16384");
        properties.put(ProducerConfig.LINGER_MS_CONFIG, "1");
        properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG, "33554432");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");

        //Producer的主对象
        Producer<String, String> producer = new KafkaProducer<String, String>(properties);

        //消息对象ProducerRecord
        for (int i = 0; i < 10; i++) {
            String key = "key-" + i;
            ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>(TOPIC_NAME, key, "value-" + i);
            Future<RecordMetadata> send = producer.send(producerRecord);
            RecordMetadata recordMetadata = send.get();
            System.out.println(key + "partition: " + recordMetadata.partition() + ", offset: " + recordMetadata.offset() );
        }

        //所有打开的通道都要关闭
        producer.close();
    }


    /**
     * Producer异步发送回调演示
     */
    public static void producerSendCallBack() {
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "10.211.55.3:9092");
        properties.put(ProducerConfig.ACKS_CONFIG, "all");
        properties.put(ProducerConfig.RETRIES_CONFIG, "0");
        properties.put(ProducerConfig.BATCH_SIZE_CONFIG, "16384");
        properties.put(ProducerConfig.LINGER_MS_CONFIG, "1");
        properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG, "33554432");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");

        //Producer的主对象
        Producer<String, String> producer = new KafkaProducer<String, String>(properties);

        //消息对象ProducerRecord
        for (int i = 0; i < 10; i++) {
            ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>(TOPIC_NAME, "key-" + i, "value-" + i);
            producer.send(producerRecord, new Callback() {
                @Override
                public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                    System.out.println("partition: " + recordMetadata.partition() + ", offset: " + recordMetadata.offset() );
                }
            });

        }
        //所有打开的通道都要关闭
        producer.close();
    }

    /**
     * Producer异步发送回调自定义Partition演示
     */
    public static void producerSendCallBackPartition() {
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "10.211.55.3:9092");
        properties.put(ProducerConfig.ACKS_CONFIG, "all");
        properties.put(ProducerConfig.RETRIES_CONFIG, "0");
        properties.put(ProducerConfig.BATCH_SIZE_CONFIG, "16384");
        properties.put(ProducerConfig.LINGER_MS_CONFIG, "1");
        properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG, "33554432");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, "com.luoye.study.kafka.producer.SamplePartition");
        //Producer的主对象
        Producer<String, String> producer = new KafkaProducer<String, String>(properties);

        //消息对象ProducerRecord
        for (int i = 0; i < 10; i++) {
            ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>(TOPIC_NAME, "key-" + i, "value-" + i);
            producer.send(producerRecord, new Callback() {
                @Override
                public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                    System.out.println("partition: " + recordMetadata.partition() + ", offset: " + recordMetadata.offset() );
                }
            });

        }
        //所有打开的通道都要关闭
        producer.close();
    }

}
```

#####  Kafka Consumer Apis

```java
1.Kafka Consumer客户端的使用
  a.kafka Consumer group注意事项
  	1).单个分区的消息只能由ConsumerGroup中的某个Consumer消费。
    2).Consumer从Partition中消费消息是顺序的，默认从头开始消费。
    3).单个ConsumerGroup会消费所有Partition中的消息。
2.Kafka Consumer客户端的配置
3.Kafka Consumer的高级特性
  a.多线程下的Consumer的两种实现
  	1).多个Consumer
    2).多个Handler
	b.手动指定Partition
  c.手动指定offset
4.Consumer端的限流处理
  a.consumer.pause();
  b.consumer.resume();
5.Consumer端的rebalance
  a.member join 新组员加入组
  b.member failure  组成员崩溃（非正常退出）
  c.member leave group 组成员主动离组
  d.member commit offset 提交位移
  
```

##### Kafka Consumer Apis instance

```java
	package com.luoye.study.kafka.consumer;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.OffsetAndMetadata;
import org.apache.kafka.common.TopicPartition;

import java.time.Duration;
import java.util.*;

/**
 * @program: kafka
 * @Author: yangbai
 * @Date: 2020/10/20 8:07 PM
 * @Version: 1.0
 * @Description: Consumer例子
 */
public class ConsumerSample {

    private final static String TOPIC_NAME = "luoye_topic";

    public static void main(String[] args) {

    }

    /**
     * 不推荐使用
     */
    private static void helloWord(){

        Properties properties = new Properties();
        properties.setProperty("bootstrap.server","10.211.55.3:9092");
        properties.setProperty("group.id","test");
        properties.setProperty("enable.auto.commit","true");
        properties.setProperty("auto.commit.interval.ms","1000");
        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");

        KafkaConsumer<String, String> objectObjectKafkaConsumer = new KafkaConsumer<>(properties);
        //消费订阅的但一个Topic或者几个Topic
        objectObjectKafkaConsumer.subscribe(Arrays.asList(TOPIC_NAME));
        while(true){
            ConsumerRecords<String, String> records = objectObjectKafkaConsumer.poll(Duration.ofMillis(10000));
            for (ConsumerRecord<String, String> record: records) {
                System.out.printf("partition = %d, offset = %d, key = %s, value = %s%n", record.partition(),record.offset(),record.key(),record.value());
            }
        }


    }

    /**
     * 手动提交offset
     */
    private static void commitdOffset(){

        Properties properties = new Properties();
        properties.setProperty("bootstrap.server","10.211.55.3:9092");
        properties.setProperty("group.id","test");
        properties.setProperty("enable.auto.commit","false");
        properties.setProperty("auto.commit.interval.ms","1000");
        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);
        //消费订阅的但一个Topic或者几个Topic
        consumer.subscribe(Arrays.asList(TOPIC_NAME));
        while(true){
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(10000));
            for (ConsumerRecord<String, String> record: records) {
                //想把数据保存到数据库，成功就成功。不成功。。。
                //TODO record 2 db
                System.out.printf("partition = %d, offset = %d, key = %s, value = %s%n", record.partition(),record.offset(),record.key(),record.value());
                //如果失败，则回滚，不提交offset
            }
            //手动通知offset提交
            consumer.commitAsync();

        }


    }

    /**
     * 手动提交offset,指定Partition
     * 手动对每个partition进行提交
     */
    private static void commitdOffsetWithPartition(){

        Properties properties = new Properties();
        properties.setProperty("bootstrap.server","10.211.55.3:9092");
        properties.setProperty("group.id","test");
        properties.setProperty("enable.auto.commit","false");
        properties.setProperty("auto.commit.interval.ms","1000");
        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);
        //消费订阅的但一个Topic或者几个Topic
        consumer.subscribe(Arrays.asList(TOPIC_NAME));
        while(true){
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(10000));
            //每个partition单独处理
            for (TopicPartition partition: records.partitions()) {
                List<ConsumerRecord<String, String>> pRecords = records.records(partition);
                for (ConsumerRecord<String, String> pRecord: pRecords) {
                    System.out.printf("partition = %d, offset = %d, key = %s, value = %s%n", pRecord.partition(),pRecord.offset(),pRecord.key(),pRecord.value());
                }
                //手动通知offset提交
                Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
                long lastOffset = pRecords.get(pRecords.size() - 1).offset();
                offsets.put(partition,new OffsetAndMetadata(lastOffset + 1 ));
                consumer.commitSync(offsets);
                System.out.println("==================partition-- " + partition + "end ======================");
            }
        }
    }

    /**
     * 手动提交offset,指定Partition
     * 手动对某几个partition（不是所有partition）进行提交
     */
    private static void commitdOffsetWithOnePartition(){

        Properties properties = new Properties();
        properties.setProperty("bootstrap.server","10.211.55.3:9092");
        properties.setProperty("group.id","test");
        properties.setProperty("enable.auto.commit","false");
        properties.setProperty("auto.commit.interval.ms","1000");
        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);
        //消费订阅的一个Partition或者几个Partition
        TopicPartition p0 = new TopicPartition(TOPIC_NAME, 0);
        consumer.assign(Arrays.asList(p0));
        while(true){
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(10000));
            //对选择的某个或某几个partition单独处理
            for (TopicPartition partition: records.partitions()) {
                List<ConsumerRecord<String, String>> pRecords = records.records(partition);
                for (ConsumerRecord<String, String> pRecord: pRecords) {
                    System.out.printf("partition = %d, offset = %d, key = %s, value = %s%n", pRecord.partition(),pRecord.offset(),pRecord.key(),pRecord.value());
                }
                //手动通知offset提交
                Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
                long lastOffset = pRecords.get(pRecords.size() - 1).offset();
                offsets.put(partition,new OffsetAndMetadata(lastOffset + 1 ));
                consumer.commitSync(offsets);
                System.out.println("==================partition-- " + partition + "end ======================");
            }
        }
    }
```

##### Kafka Stream Apis

```java
1.基本概念
  a.Kafka Stream是处理分析存储在Kafka数据的客户端程序
  b.Kafka Stream是通过state store可以实现高效状态操作
  c.支持原语Processor和高层抽象DSL语法
```

##### Kafka Connect Apis

```java
1.基本概念
  a.Kafka Connect是Kafka流式计算的一部分
  b.Kafka Connect主要用来与其他中间件建立流式通道
  c.Kafka Connect支持流式和批量处理集成
2.
```

##### Kafka 集群部署

```java
1.copy三份kafka压缩包，配置kafka的配置文件
2.修改/config/server.propertity文件
 a.broke.id的值
 b.端口号
 c.日志存储地址路径
3.启动kafka
```

##### Kafka 集群总览

```java
Kafka副本集：
	1.Kafka副本集是指将日志复制多份
	2.Kafka可以为每个Topic设置副本集
  3.Kafka可以通过配置设置默认副本集数量
kafak核心概念：
  1.Broker:一般指Kafka的部署节点
  2.Leader:用于处理消息的接受和消费等请求
  3.Follower:主要用于备份消息数据
kafka节点故障：
  1.kafka与zookeeper心跳未保持视为节点故障
  2.follower消息落后leader太多也视为节点故障
  3.kafak会对故障节点进行移除
kafka节点故障处理
  1.kafka基本不会因为节点故障而丢失数据
  2.kafka的语义担保也很大程度上避免数据丢失
  3.kafka会对消息进行集群内的平衡，减少消息在某些节点热度过高
kafka集群的Leader选举
  1.Kafka并没用采用多数投票来选举leader
  2.Kafka会动态维护一组Leader数据的副本（ISR）
  3.Kafka会在ISR中选择一个速度比较快的设为Leader
kafka集群的一种特殊情况
  1.ISR中副本全部宕机
  2.对于上数情况，Kafka会进行unclean leader选举
  3.Kafka提供了两种不同的选择处理改部分内容
Kafka生产环境的选举配置优化：
  1.禁用“unclean Leader”选举
  2.手动指定最小ISR
```

##### Kafka 集群监控和安全机制

```java
Kafka常用的监控插件：
  1.Kafka manager
kafka manager的安装：
  1.解压缩kafka-manager的压缩包。
  2.修改Kafka-manager的conf的配置，连接ZK的链接地址
  3.启动kafka
  4.启动kafk集群
  5.在默认的9000端口查看集群状态
Kafka的安全机制：
  1.kafka提供额SSL和SASL证书验证机制
  2.Kafka提供了Broker到zk连接的安全机制
  3.Kakfa支持Client的读写验证
Kafka的SSL安全机制配置：
  1.生成SSL证书
  2.配置SSL服务端
  3.配置SSL客户端
```

##### Kafka 最佳实践

```

```

##### Kafka springcloud config集成

```java
集成步骤：
  1.创建spring-cloud-config-server服务
  2.创建spring-cloud-config-client服务
  3.引入依赖spring-cloud-config-kafka-bus依赖
```

##### spring-cloud-config-server服务创建

```

```

