# kafka

#### 1.kafka安装

1.1下载安装包

```shell
wget https://archive.apache.org/dist/kafka/2.4.1/kafka_2.11-2.4.1.tgz
```

1.2配置config目录中的server.properties

```properties
#broker.id属性在kafka集群中必须要是唯一
broker.id=0
#kafka部署的机器ip和提供服务的端口号
advertised.listeners=PLAINTEXT://localhost:9092
#kafka的消息存储文件路径
log.dirs=/usr/local/kafka/data/kafka-logs
#kafka连接zookeeper的地址
zookeeper.connect=localhost:2181

```

1.3进入bin目录，执行命令启动kafka

```shell
#daemon守护进程启动
./bin/kafka-server-start.sh -daemon config/server.properties
```

1.4校验kafka是否启动成功

进入到zk内查看是否有kafka的节点：/brokers/ids/0

#### 2.kafka命令

2.1通过kafka创建topic

```shell
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

2.2查看topic

```shell
./kafka-topics.sh --list --zookeeper localhost:2181
```

2.3查看当前kafka内有那些topic

```shell
./kafka-topics.sh --list --zookeeper 192.168.191.129:2181
```

2.4发送消息

```shell
./kafka-console-producer.sh --broker-list 192.168.191.129:9092 --topic first
```

2.5消费消息

- 从最后一条消息的偏移量+1开始消费

```shell
./kafka-console-consumer.sh --bootstrap-server 192.168.191.129:9092 --topic first
```



- 从头开始消费

```shell
./kafka-console-consumer.sh --bootstrap-server 192.168.191.129:9092 --from-beginning --topic first
```

#### 3.kafka集群操作

​	3.1安装docker-compose

```shell
curl -L http://mirror.azure.cn/docker-toolbox/linux/compose/1.25.4/docker-compose-Linux-x86_64 -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

​	3.2创建docker-compose.yaml文件

```yaml
version: '2'
services:
    zookeeper:
        image: wurstmeister/zookeeper
        restart: always
        ports:
            - 2181:2181
    kafka1:
        image: wurstmeister/kafka
        restart: always
        depends_on:
            - zookeeper
        ports:
            - 9093:9093
        environment:
            KAFKA_ADVERTISED_HOST_NAME: 192.168.191.129
            KAFKA_ZOOKEEPER_CONNECT: 192.168.191.129:2181
            KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9093
            KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.191.129:9093
            KAFKA_BROKER_ID: 1
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
    kafka2:
        image: wurstmeister/kafka
        restart: always
        depends_on:
            - zookeeper
        ports:
            - 9094:9094
        environment:
            KAFKA_ADVERTISED_HOST_NAME: 192.168.191.129
            KAFKA_ZOOKEEPER_CONNECT: 192.168.191.129:2181
            KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9094
            KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.191.129:9094
            KAFKA_BROKER_ID: 2
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
    kafka3:
        image: wurstmeister/kafka
        restart: always
        depends_on:
            - zookeeper
        ports:
            - 9095:9095
        environment:
            KAFKA_ADVERTISED_HOST_NAME: 192.168.191.129
            KAFKA_ZOOKEEPER_CONNECT: 192.168.191.129:2181
            KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9095
            KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.191.129:9095
            KAFKA_BROKER_ID: 3
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock

```

参数说明：

- KAFKA_ZOOKEEPER_CONNECT: zookeeper服务地址
- KAFKA_ADVERTISED_LISTENERS: kafka服务地址
- KAFKA_BROKER_ID: kafka节点ID，不可重复
- KAFKA_LOG_DIRS: kafka数据文件地址(非必选。下面的volumes会将容器内文件挂载到宿主机上)
- KAFKA_LOG_RETENTION_HOURS: 数据文件保留时间(非必选。缺省168小时)


3.3启动命令

```shell
docker-compose up -d
```

3.4查看启动的容器

```shell
docker-compose ps

#检验是否成功
进入到zk中查看/brokers/ids中是否有三个znode(0,1,2)
```

3.5登录到kafka1容器

```shell
docker-compose exec kafka1 bash
```

3.6在/opt/kafka/config下可以看到配置文件（最主要的是server.properties）

3.7创建一个topic:名称为first，3个分区，2个副本

```shell
cd /opt/kafka/bin/

#创建一个主题，2个分区，3个副本
./kafka-topics.sh --create --topic my-replication-factor-topic --zookeeper 192.168.191.129:2181 --partitions 2 --replication-factor 3
#查看topic情况
./kafka-topics.sh --describe --zookeeper 192.168.191.129:2181 --topic my-replication-factor-topic
```

3.8kafka集群消息的发送

```shell
./kafka-console-producer.sh --broker-list 192.168.191.129:9093,192.168.191.129:9094,192.168.191.129:9095 --topic my-replication-factor-topic
```

3.9kafka集群消息的消费

```shell
./kafka-console-consumer.sh --bootstrap-server 192.168.191.129:9093,192.168.191.129:9094,192.168.191.129:9095 --from-beginning --topic my-replication-factor-topic
```

#### 4.kafka的Java客户端-生产者的实现

##### 4.1 生产者的基本实现

- 引入依赖

  ```xml
  <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>2.4.1</version>
  </dependency>
  ```

- 具体实现

```java
public class MyProducer {

    private final static String TOPIC_NAME = "my-replication-factor-topic";

    public static void main(final String[] args) throws ExecutionException, InterruptedException {
        //1.设置参数
        final Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.191.129:9093,192.168.191.129:9094,192.168.191.129:9095");
        //把发送的key从字符串序列化为字节数组
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        //把发送消息value从字符串序列化为字节数组
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        //2.创建生产者客户端，传入参数
        final KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        //3.创建消息
        //key：作用是决定向那个分区上发，value：具体要发送的消息内容
        final ProducerRecord<String, String> producerRecord = new ProducerRecord<>(TOPIC_NAME, 0, "mykey", "hellokafka");
        //4.发送消息,得到消息的元数据并输出--同步发送
//        final RecordMetadata recordMetadata = producer.send(producerRecord).get();
//        System.out.println("同步方式发送消息结果：" + "topic-" + recordMetadata.topic()
//                + "|partition" + recordMetadata.partition() + "|offset-" + recordMetadata.offset());
//        
        //异步消息发送
        producer.send(producerRecord, new Callback() {

            @Override
            public void onCompletion(final RecordMetadata recordMetadata, final Exception e) {

                if (e != null) {
                    System.err.println("发送消息失败" + e.getStackTrace());
                }
                if (recordMetadata != null) {
                    System.out.println("异步方式发送消息结果：" + "topic-" + recordMetadata.topic()
                            + "|partition" + recordMetadata.partition() + "|offset-" + recordMetadata.offset());
                }
            }
        });

        Thread.sleep(1000000000000L);
    }
}
```

##### 4.2发送消息到指定分区上

```java
final ProducerRecord<String, String> producerRecord = new ProducerRecord<>(TOPIC_NAME, 0, "mykey", "hellokafka");
```

##### 4.3未指定分区，则会通过业务key的hash运算，算出消息往哪个分区上发

```java
//未指定发送分区，具体发送的分区计算公式：hash(key)%partitionNum
final ProducerRecord<String, String> producerRecord = new ProducerRecord<>(TOPIC_NAME, 0, "mykey", "hellokafka");
```

##### 4.4生产者的同步发送消息

​	如果生产者发送消息没有收到ack，生产者会阻塞，阻塞到3s的时间，如果还没收到消息，会进行重试。重试的次数3次

```java
// 4.发送消息,得到消息的元数据并输出--同步发送
final RecordMetadata recordMetadata = producer.send(producerRecord).get();
System.out.println("同步方式发送消息结果：" + "topic-" + recordMetadata.topic()
                   + "|partition" + recordMetadata.partition() + "|offset-" + recordMetadata.offset());
```

##### 4.5生产者的异步发送消息

​	异步发送，生产者发送完消息就可以执行之后的业务，broker在收到消息后异步调用生产者提供的callback回调方法

```java
//异步消息发送
        producer.send(producerRecord, new Callback() {

            @Override
            public void onCompletion(final RecordMetadata recordMetadata, final Exception e) {

                if (e != null) {
                    System.err.println("发送消息失败" + e.getStackTrace());
                }
                if (recordMetadata != null) {
                    System.out.println("异步方式发送消息结果：" + "topic-" + recordMetadata.topic()
                            + "|partition" + recordMetadata.partition() + "|offset-" + recordMetadata.offset());
                }
            }
        });
```

##### 4.6生产者中的ack的配置

在同步发送的前提下，生产者在获得集群返回的ack之前会阻塞。此时ack有三个配置：

- ack=0 kafka-cluster不需要任何的broker收到消息，就立即返回ack给生产者，最容易丢失消息，效率是最高的
- ack=1(默认) 多副本之间的leader已经收到消息，并把消息写入到本地的log中，才会返回ack给生产者，性能和安全性是最均衡的
- ack=-1/all 里面有默认的配置min.insync.replicas=2(默认为1，推荐配置大于等于2)，此时需要leader和一个follower同步完成后，才会返回ack给生产者（此时集群中有2个broker已完成数据接受），这种方式是最安全的，但是性能最差

ack和重试的配置

```java
 //重试次数
/**
         *发送失败会重试，默认重试间隔100ms,重试能保证消息发送的可靠性，但是也可能造成消息重复发送，比如网络抖动，所有需要在接收者
         *那边做好消息接收的幂等性处理
         */
props.put(ProducerConfig.ACKS_CONFIG, "1");
props.put(ProducerConfig.RETRIES_CONFIG, 3);
//重试间隔设置
props.put(ProducerConfig.RETRY_BACKOFF_MS_CONFIG, 300);
```

其他的配置

```java
//设置发送消息的本地缓冲区，如果设置了该缓冲区，消息会先发送到本地缓冲区，可以提高消息发送的性能，默认值是33554432，即32M
props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
//设置本地线程回去缓冲区一次拉取16k的数据，发送到broker
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
//如果线程线程拉不到16k的数据，间隔10ms也会将已拉到的数据发送到broker
props.put(ProducerConfig.LINGER_MS_CONFIG, 10);
```

#### 5.kafka的Java客户端-消费者的实现

##### 5.1基本实现

```java
public class MyConsumer {

    private final static String TOPIC_NAME = "my-replication-factor-topic";

    private final static String CONSUMER_GROUP_CONFIG = "testGroup";

    public static void main(final String[] args) {

        final Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.191.129:9093,192.168.191.129:9094,192.168.191.129:9095");
        //消费组名
        props.put(ConsumerConfig.GROUP_ID_CONFIG, CONSUMER_GROUP_CONFIG);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        //创建一个人消费组客户端
        final KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        //消费者订阅主题
        consumer.subscribe(Arrays.asList(TOPIC_NAME));
        while (true) {
            final ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (final ConsumerRecord<String, String> record : records) {
                System.out.printf("收到消息：partition = %d,offset = %d,key= %s,value =%s%n",
                        record.partition(), record.offset(), record.key(), record.value());
            }
        }
    }
}
```

##### 5.2自动提交和手动提交offset

###### 1）提交内容

消费组无论是自动提交还是手动提交，都需要把所属的消费组+消费主题+消费的某个分区及消费的偏移量等信息提交到集群的_consumer_offsets主题里面

###### 2）自动提交

消费者poll消息下来以后就会提交offset

```java
        //是否自动提交offset。默认为true
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        //自动提交offset的时间间隔
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
```

注意：自动提交会丢消息。因为消费者在消费前提交offset，有可能提交完后还没消费就挂了。新的消费者会从下一个offset开始消费

###### 3）手动提交

需要把自动提交的配置改成false

```java
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
```

手动同步提交

在消费完消息后调用同步提交的方法，当集群返回ack之前一直阻塞，返回ack后表示提交成功，执行之后的逻辑

```java
while (true) {
            final ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (final ConsumerRecord<String, String> record : records) {
                System.out.printf("收到消息：partition = %d,offset = %d,key= %s,value =%s%n",
                        record.partition(), record.offset(), record.key(), record.value());
            }
            if (records.count() > 0) {
               //一般使用同步提交，因为提交之后一般没什么逻辑代码执行
               consumer.commitSync();
            }

            
        }
```



手动异步提交

在消息消费完后提交，不需要等到集群ack，直接执行之后的逻辑，可以设置一个回调方法，供集群调用

```java
while (true) {
            final ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (final ConsumerRecord<String, String> record : records) {
                System.out.printf("收到消息：partition = %d,offset = %d,key= %s,value =%s%n",
                        record.partition(), record.offset(), record.key(), record.value());
            }

            if (records.count() > 0) {
                //一般使用同步提交，因为提交之后一般没什么逻辑代码执行
                consumer.commitAsync(new OffsetCommitCallback() {

                    @Override
                    public void onComplete(final Map<TopicPartition, OffsetAndMetadata> map, final Exception e) {

                        if (e != null) {
                            System.out.println(map);
                        }
                    }
                });
            }
        }
```

##### 5.3长轮询poll消息

​	默认一次poll500条消息

```java
  //一次poll拉取消息的最大条数
        props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500);
```

设置长轮询的时间是1000ms

```java
while (true) {
            final ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (final ConsumerRecord<String, String> record : records) {
                System.out.printf("收到消息：partition = %d,offset = %d,key= %s,value =%s%n",
                        record.partition(), record.offset(), record.key(), record.value());
            }
}
```

1）如果poll到500条，直接执行循环

2）如果一次没有poll到500条，且时间在1秒内，长轮询继续执行，要么到500条，要么到1秒

3）如果多次poll没有500条，时间到了1秒，直接执行循环

如果两次的poll的时间间隔超过30秒，集群就会认为该消费者消费能力过弱，该消费组被踢出消费组，触发rebalance机制，rebalance机制会造成性能开销，可以通过设置参数，一次poll消息条数少一点

```java
props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 100);
props.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 30 * 1000);
```

##### 5.4消费者的健康状态检查

消费者每隔1秒向kafka集群发送心跳，集群如果发现10秒没有续约的消费者，将被踢出消费组，触发该消费组的rebalance机制，将该分区交给消费组中的其他消费者消费

```java
//发送心跳时间
props.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 1000);
props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 10 * 1000);
```

5.5指定分区和偏移量，时间消费

```java
 		//消费指定分区
        consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
        //消费回溯消息
        consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
        consumer.seekToBeginning(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));

        //指定offset消费
        consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
        consumer.seek(new TopicPartition(TOPIC_NAME, 0), 10);

        //从指定的时间点开始消费
        final List<PartitionInfo> topicPartition = consumer.partitionsFor(TOPIC_NAME);
        //从一小时前开始消费
        final long l = new Date().getTime() - 1000 * 60 * 60;
        final Map<TopicPartition, Long> map = new HashMap<>();
        for (final PartitionInfo partitionInfo : topicPartition) {
            map.put(new TopicPartition(TOPIC_NAME, partitionInfo.partition()), l);
        }
        final Map<TopicPartition, OffsetAndTimestamp> parMap = consumer.offsetsForTimes(map);
        for (final Map.Entry<TopicPartition, OffsetAndTimestamp> entry : parMap.entrySet()) {
            final TopicPartition key = entry.getKey();
            final OffsetAndTimestamp value = entry.getValue();

            if (key == null || value == null) {
                continue;
            }

            final long offset = value.offset();
            System.out.println("partition:" + key.partition());
            System.out.println("offset:" + offset);
            //根据消费的timestamp确定offset
            if (value != null) {
                consumer.assign(Arrays.asList(key));
                consumer.seek(key, offset);
            }
        }
```

根据时间，去所有的partition中确定该时间对应的offset，然后去所有的partition中找到该offset之后开始消费

##### 5.5新消费者消费offset规则

新的消费组中的消费者启动，默认会从当前分区的最后一条消息的offset+1开始消费（消费新的消息），

设置新的消费者从头开始消费，之后开始消费新消息（最后消费的位置的offset+1）

```java
//默认消费新消息，latest
//设置新的消费者从头开始消费，之后开始消费新消息（最后消费的位置的offset+1） earliest
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
```

#### 6.SpringBoot中使用kafka

##### 6.1引入依赖

```java
<dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
```

##### 6.2配置文件

```yaml
server:
  port: 8080

spring:
  kafka:
    bootstrap-servers: 192.168.191.129:9093,192.168.191.129:9094,192.168.191.129:9095
    producer:
      retries: 3
      batch-size: 16384
      buffer-memory: 33554432
      acks: 1
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      bootstrap-servers: 192.168.191.129:9093,192.168.191.129:9094,192.168.191.129:9095
      group-id: default-group
      enable-auto-commit: false
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      max-poll-records: 500
    listener:
      ack-mode: manual_immediate
  


```



##### 6.3生产者

```java

package com.example.kafka.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/msg")
public class kafkaController {

    private final static String TOPIC_NAME = "my-replication-factor-topic";

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @RequestMapping("/send")

    public String sendMessage() {

        this.kafkaTemplate.send(TOPIC_NAME, "key", "message");
        return "success";
    }
    
}

```



##### 6.4消费者

```java

package com.example.kafka.consumer;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.Acknowledgment;
import org.springframework.stereotype.Component;

@Component
public class MyConsumer {

    @KafkaListener(topics = "my-replication-factor-topic", groupId = "testGroup")
    public void listenGroup(final ConsumerRecord<String, String> consumerRecord, final Acknowledgment ack) {

        final String value = consumerRecord.value();
        System.out.println(value);
        System.out.println(consumerRecord);
        //手动提交offset
        ack.acknowledge();
    }
}

```

消费者中配置消费主题，分区和偏移量 

```java
@KafkaListener(groupId = "", topicPartitions = {
            @TopicPartition(topic = "", partitions = {"1", "0"}),
            @TopicPartition(topic = "", partitions = "1",
                    partitionOffsets = @PartitionOffset(partition = "1", initialOffset = "100")),

    }, concurrency = "3")//concurrency就是同组下的消费者个数，就是并发消息数，建议小于等于分区总数
    public void listenGroup(final ConsumerRecord<String, String> consumerRecord, final Acknowledgment ack) {

        final String value = consumerRecord.value();
        System.out.println(value);
        System.out.println(consumerRecord);
        //手动提交offset
        ack.acknowledge();
    }
```

#### 7.kafka集群中的controller，rebalance，HW

##### 	7.1controller

​	1.集群中谁来充当controller

​		每个broker启动时会向zk创建一个临时的序号节点，获得的序号最小的那个broker将会作为集群中的controller，主要功能为

​		1.1当集群中有一个副本的leader挂掉，需要在集群中选举出一个人新的leader，选举的规则是从ISR集合中最右边获取。

​		1.2当集群中有broker新增或者减少，controller会同步信息给其他broker

​		1.3当集群中有分区新增或者减少，controller会同步信息给其他broker

##### 	7.2rebalance机制

​	前提：消费组中的消费者没有指明分区来消费

​	触发条件：当消费组中的消费者和分区的关系发生变化的时候

分区分配的策略：在rebalance之前，分区怎么分配有三种策略

​	1.rang：根据公式计算得到每个消费者消费那几个分区：前面的消费者是分区总数/消费者数量+1(sum/n+1)，之后的消费者是分区总数/消费者数量  

2.轮询

3.sticky：粘合策略，如果需要rebalance，会在之前已分配的基础上调整，不会改变之前的分配情况，如果这个策略没有开，那么就要进行全部的重新分配，会消耗性能，建议开启

##### 	7.3HW和LEO

LEO是某个副本最后消息的位置（log-end-offset）

HW是已完成同步的位置，消息在写入broker时，且每个broker完成这条消息的同步后，hw才会变化，在这之前消费者是消费不到这条消息的。在完成同步之后，HW更新之后，消费者才能消费到这条消息，这样的目的是为了防止消息丢失



# spark

##### 1.Spark 是什么

- Spark 是一种基于内存的快速、通用、可扩展的大数据分析计算引擎。
- Spark 和Hadoop 的根本差异是多个作业之间的数据通信问题 : Spark 多个作业之间数据 通信是基于内存，而 Hadoop 是基于磁盘。
- Spark Task 的启动时间快。Spark 采用 fork 线程的方式，而 Hadoop 采用创建新的进程 的方式。
- Spark 只有在 shuffle 的时候将数据写入磁盘，而 Hadoop 中多个 MR 作业之间的数据交 互都要依赖于磁盘交互

##### 2.构建WordCount

######  2.1.1 增加 Scala 插件

​		Spark 由 Scala 语言开发的，当 前使用的 Spark 版本为 3.0.0，默认采用的 Scala 编译版本为 2.12，

![image-20211015102445333](D:\document\img\image-20211015102445333.png)

###### 2.1.2 增加依赖关系 

```xml
<dependencies>
 <dependency>
 <groupId>org.apache.spark</groupId>
 <artifactId>spark-core_2.12</artifactId>
 <version>3.0.0</version>
 </dependency>
</dependencies>
```

###### 2.1.3 WordCount

```scala
object Spark01_WordCount {
  def main(args: Array[String]): Unit = {
    //建立与spark框架的连接
    val sparkConf = new SparkConf().setMaster("local").setAppName("WordCount")
    val sc = new SparkContext(sparkConf)
    //执行业务操作
    //1.读取文件，获取一行一行的数据
    val lines: RDD[String] = sc.textFile("datas")

    //2.将一行数据进行拆分，形成一个个的单词
    val words: RDD[String] = lines.flatMap(_.split(""))
    //3.将数据根据单词进行分组，便于统计
    val wordGroup: RDD[(String, Iterable[String])] = words.groupBy(word => word)

    //4.对分组后的数据进行转换
    val wordToCount = wordGroup.map {
      case (word, list) => {
        (word, list.size)
      }
    }
    val array: Array[(String, Int)] = wordToCount.collect()
    array.foreach(println)
    //关闭连接
    sc.stop()
  }
}
```

###### 2.1.4 异常处理

​	本机操作系统是 Windows，在程序中使用了 Hadoop 相关的东西，比如写入文件到 HDFS，则会遇到如下异常：

![image-20211015102329445](D:\document\img\image-20211015102329445.png)

出现这个问题的原因，并不是程序的错误，而是 windows 系统用到了 hadoop 相关的服 务，解决办法是通过配置关联到 windows 的系统依赖就可以了

##### 3.Spark 运行环境

###### 3.1 Local 模式

###### 3.1.1 解压缩文件 

将下载的文件上传到虚拟机中

```shell
tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module
cd /opt/module
mv spark-3.0.0-bin-hadoop3.2 spark-local
```

###### 3.1.2 启动 Local 环境 

​	1.进入解压缩后的路径，执行如下指令

```shell
bin/spark-shell
```

​    2.启动成功后，可以输入网址进行 Web UI 监控页面访问

​	http://192.168.191.129:4040

###### 3.1.3 退出本地模式 

按键 Ctrl+C 或输入 Scala 指令

```shell
:quit
```

###### 3.1.4 提交应用 

```shell
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master local[2] \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10
```

1) --class 表示要执行程序的主类，此处可以更换为咱们自己写的应用程序 
2) --master local[2] 部署模式，默认为本地模式，数字表示分配的虚拟 CPU 核数量 
3) spark-examples_2.12-3.0.0.jar 运行的应用类所在的 jar 包，实际使用时，可以设定为自己打的 jar 包 
4) 数字 10 表示程序的入口参数，用于设定当前应用的任务数量

##### 4.SparkSQL 是什么

Spark SQL 是 Spark 用于结构化数据(structured data)处理的 Spark 模块

###### 4.1开发SparkSQL

######  1.引入依赖

```java
<dependency>
 <groupId>org.apache.spark</groupId>
 <artifactId>spark-sql_2.12</artifactId>
 <version>3.0.0</version>
</dependency>
```

###### 2.测试代码

```scala
object SparkSQL01_Demo {
 def main(args: Array[String]): Unit = {
 //创建上下文环境配置对象
 val conf: SparkConf = new
SparkConf().setMaster("local[*]").setAppName("SparkSQL01_Demo")
 //创建 SparkSession 对象
 val spark: SparkSession = SparkSession.builder().config(conf).getOrCreate()
 //RDD=>DataFrame=>DataSet 转换需要引入隐式转换规则，否则无法转换
 //spark 不是包名，是上下文环境对象名
 import spark.implicits._
 //读取 json 文件 创建 DataFrame {"username": "lisi","age": 18}
 val df: DataFrame = spark.read.json("input/test.json")
 //df.show()
 //SQL 风格语法
 df.createOrReplaceTempView("user")
 //spark.sql("select avg(age) from user").show
 //DSL 风格语法
 //df.select("username","age").show()

 //RDD
 val rdd1: RDD[(Int, String, Int)] =
spark.sparkContext.makeRDD(List((1,"zhangsan",30),(2,"lisi",28),(3,"wangwu",
20)))
 //DataFrame
 val df1: DataFrame = rdd1.toDF("id","name","age")
 //df1.show()
 //DateSet
 val ds1: Dataset[User] = df1.as[User]
 //ds1.show()
 
 //DataFrame
 val df2: DataFrame = ds1.toDF()
 //RDD 返回的 RDD 类型为 Row，里面提供的 getXXX 方法可以获取字段值，类似 jdbc 处理结果集，
但是索引从 0 开始
 val rdd2: RDD[Row] = df2.rdd
 //rdd2.foreach(a=>println(a.getString(1)))
 
 rdd1.map{
 	case (id,name,age)=>User(id,name,age)
 }.toDS()
 
 ds1.rdd
 //释放资源
 spark.stop()
 }
}
case class User(id:Int,name:String,age:Int)
```

###### 3.连接mysql

​	引入依赖

```xml
<dependency>
 <groupId>mysql</groupId>
 <artifactId>mysql-connector-java</artifactId>
 <version>5.1.27</version>
</dependency>
```

​	读取数据

```scala
val conf: SparkConf = new
SparkConf().setMaster("local[*]").setAppName("SparkSQL")
//创建 SparkSession 对象
val spark: SparkSession = SparkSession.builder().config(conf).getOrCreate()
import spark.implicits._

val props: Properties = new Properties()
props.setProperty("user", "root")
props.setProperty("password", "123123")
val df: DataFrame = spark.read.jdbc("jdbc:mysql://192.168.191.129:3306/spark-sql",
"user", props)
df.show
//释放资源
spark.stop()
```

​	写入数据

```scala
val conf: SparkConf = new
SparkConf().setMaster("local[*]").setAppName("SparkSQL")
//创建 SparkSession 对象
val spark: SparkSession = SparkSession.builder().config(conf).getOrCreate()
import spark.implicits._
val rdd: RDD[User2] = spark.sparkContext.makeRDD(List(User2("lisi", 20),
User2("zs", 30)))
val ds: Dataset[User2] = rdd.toDS
val props: Properties = new Properties()
props.setProperty("user", "root")
props.setProperty("password", "123123")
ds.write.mode(SaveMode.Append).jdbc("jdbc:mysql://192.168.191.129:3306/spark-sql",
"user", props)
//释放资源
spark.stop()
```



