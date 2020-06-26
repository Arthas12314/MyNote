## Kafka

### 概述

#### 特性

* 高吞吐量、低延迟

  每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个topic可以分多个partition, consumer group 对partition进行consume操作。

* 可扩展性

  kafka集群支持热扩展

* 持久性、可靠性

  消息被持久化到本地磁盘，并且支持数据备份防止数据丢失

* 容错性

  允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）

* 高并发

  支持数千个客户端同时读写

#### 使用场景

* 日志收集

  一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、Hbase、Solr等。

* 消息系统

  解耦和生产者和消费者、缓存消息等。

* 用户活动跟踪

  Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘。

* 运营指标

  Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。

* 流式处理

  比如spark streaming和storm

* 事件源

#### 技术优势

* 可伸缩性

  1. Kafka集群在运行期间可以轻松的扩展和收缩
  2. 可以扩展一个Kafka主题来包含更多的分区，由于一个分区无法扩展到多个代理，所以它的容量收到代理磁盘空间的限制，能够增加分区和代理的数量意味着单个主题可以存储的数据量是没有限制的

* 容错性和可靠性

  Kafka的设计方式是某个代理的故障能够被集群中的其他代理检测到，由于每个主题都可以在多个代理上复制，所以集群可以在不中断服务的情况下从此类故障中恢复并继续运行

* 吞吐量

  代理能够以超快的速度有效地存储和检索数据

### 生产者

> producer封装数据封装成一个ProducerRecord对象

#### 发送类型

* 发送即忘记

  ```java
  producer.send(record);
  ```

* 同步发送

  ```java
  producer.send(record).get();
  ```

* 异步发送

  ```java
  producer.send(record,new Callback(){
      public void onCompletion(RecordMetadata metadata,Exception exception){
          
      }
  });
  ```

#### 序列化器

* Serializer接口

  提供了StringSerializer, IntegerSerializer, BytesSerializer

* 自定义序列化器

#### 分区器

* 分区策略

  1. 指定partition时，直接按指定

  2. 没有指定但包含key，则按key的hash与topic的partition数取余的值

  3. 均未指定则第一次调用随机生成值，将值按topic的partition总数取

     轮询调度算法的原理是每一次把来自用户的请求轮流分配给内部中的服务器，从1开始，直到N(内部服务器个数)，然后重新开始循环。算法的优点是其简洁性，它无需记录当前所有连接的状态，所以它是一种无状态调度。

* 自定义分区器

#### 拦截器

* 使用场景

  1. 按照某个规则过滤掉不符合要求的消息
  2. 修改消息的内容
  3. 统计类需求

* 自定义拦截器

  ```java
  public class ProducerInterceptor implements ProducerInterceptor<String,String>{
      private volatile long sendSuccess = 0;
      private volatile long sendFailure = 0;
      
      @Override
      public P onSend(){
          return new ProducerRecord<>(
      record.topic(), record.partition(), record.timestamp(),record.key(),"prefix-"+record.value(),record.headers());
      }
      
      @Override
      public void onAcknowledgement(RecordMetadata recordMetadata, Exception e){
          if(e == null){
              sendSuccess++;
          } else{
              sendFailure++;
          }
      }
      
      @Override
      public void close(){
          
      }
      
      @Override
      public void configure(){
          
      }
  }
  
  ```

#### 生产者参数

* ACK

  Kafka 为用户提供了三种可靠性级别，用户根据可靠性和延迟的要求进行权衡，选择以下配置

  - 0表示producer 不等待 broker 的 ack，这提供了最低延迟，broker 一收到数据还没有写入磁盘就已经返回，当 broker 故障时有可能丢失数据
  - 1表示producer 等待 broker 的 ack，partition 的 leader 落盘成功后返回 ack，如果在 follower 同步成功之前 leader 故障，那么将会丢失数据
  - -1表示producer 等待 broker 的 ack，partition 的 leader 和 follower 全部落盘成功后才返回 ack。但是在 broker 发送 ack 时，leader 发生故障，则会造成数据重复

### 消费者

#### 订阅分区和主题

```java
KafkaConsumer<String,String> consumer = new KafkaConsumer<>(properties);
consumer.subscribe(Collections.singletonList(topic));
consumer.assign(Arrays.asList(new TopicPartition(topic,0)));
```

#### 消费消息

对于Kafka的分区而言，它的每条消息都有唯一的offset，用来表示消息在分区中的位置

当我们调用poll()，消息从broker返回消费者，broker并不跟踪这些消息是否被消费者接收，Kafka让消费者自身来管理消费的位移，并向消费者提供更新位移的接口，此种方式称为提交

* 重复消费

  多个消费者，某个消费者消息唯一还未提交，另一个消费者可能消费到还未提交但已经被消费的消息

* 消息丢失

  消费者消费一部分后宕机，则重启后未消费的部分丢失

* 自动提交

  让消费者管理位移，应用本身不需要显式操作，将enable.auto.commit设置为true，消费者调用poll后每隔五秒提交一次offset，此种方式可能导致重复消费

* 同步提交

  consumer.commitSync();

* 异步提交

  异步提交可能在提交失败后不能进行重试

#### 指定位移消费

consumer.seek(topic,offset)

#### 重平衡监听器

指分区的所属从一个消费者转移到另一个消费者的行为，期间消费者无法拉取消息

#### 消费者拦截器

应用于消费到消息或者提交消费位移时进行的一些定制化操作，如设置消息有效期的属性

#### 其他参数

* fetch.min.bytes

  允许消费者指定从broker读取消息时最小的数据量，当小欸这从broker读取消息时，如果数据量小于这个阈值，broker会等待有足够的数据后才会批量返回给消费者。减少broker和消费者的压力

* fetch.max.wait.ms

  指定了消费者读取时的最长等待时间，避免长时间阻塞，默认500ms

* max.poll.records

  制定了每个分区返回的最多字节数，默认1M

### 主题

#### 创建主题

bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test --replication-factor 1

参数依次表示为

* zookeeper所在IP，多个使用,分隔
* 主题分区数，每个线程处理一个分区数据
* 设置主题副本数，每个副本分布在不同节点，不能吃超过总结点数，否则将报错。

