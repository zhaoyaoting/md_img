# Kafka

kafa安装配置



![image-20210928172610700](D:\document\img\image-20210928172610700.png)

通过kafka创建topic

```shell
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```



查看topic

```shell
./kafka-topics.sh --list --zookeeper localhost:2181
```



![image-20210929145452792](D:\document\img\image-20210929145452792.png)

![image-20210929150854377](D:\document\img\image-20210929150854377.png)

![image-20210929151534575](D:\document\img\image-20210929151534575.png)

![image-20210929152208225](D:\document\img\image-20210929152208225.png)

![image-20210929152633204](D:\document\img\image-20210929152633204.png)

![image-20210929153810319](D:\document\img\image-20210929153810319.png)

![image-20211008160133116](D:\document\img\image-20211008160133116.png)

```shell
#创建一个主题，2个分区，3个副本
./kafka-topics.sh --create --topic my-replication-factor-topic --zookeeper 192.168.191.129:2181 --partitions 2 --replication-factor 3
#查看topic情况
./kafka-topics.sh --describe --zookeeper 192.168.191.129:2181 --topic my-replication-factor-topic
```

![image-20211008174556419](D:\document\img\image-20211008174556419.png)

![image-20211008174320756](D:\document\img\image-20211008174320756.png)



![image-20211008175209976](D:\document\img\image-20211008175209976.png)

kafka集群消息的发送

```shell
./kafka-console-producer.sh --broker-list 192.168.191.129:9093,192.168.191.129:9094,192.168.191.129:9095 --topic my-replication-factor-topic
```

kafka集群消息的消费

```shell
./kafka-console-consumer.sh --bootstrap-server 192.168.191.129:9093,192.168.191.129:9094,192.168.191.129:9095 --from-beginning --topic my-replication-factor-topic
```

#### 关于集群消费

![image-20211011170500218](D:\document\img\image-20211011170500218.png)

![image-20211011174112218](D:\document\img\image-20211011174112218.png)

![image-20211011175937849](C:\Users\zhoayaoting\AppData\Roaming\Typora\typora-user-images\image-20211011175937849.png)

![image-20211011180011368](D:\document\img\image-20211011180011368.png)

![image-20211012162418799](D:\document\img\image-20211012162418799.png)

 ![image-20211012164750105](D:\document\img\image-20211012164750105.png)

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

![image-20211012165048481](D:\document\img\image-20211012165048481.png)

![image-20211012174831970](D:\document\img\image-20211012174831970.png)

 ![image-20211012184454195](D:\document\img\image-20211012184454195.png)

![image-20211012184752975](D:\document\img\image-20211012184752975.png)

![image-20211013144225921](D:\document\img\image-20211013144225921.png)

![image-20211013162052302](D:\document\img\image-20211013162052302.png)

![image-20211013170605464](D:\document\img\image-20211013170605464.png)

![image-20211013165000525](D:\document\img\image-20211013165000525.png)

![image-20211013172258326](D:\document\img\image-20211013172258326.png)

![image-20211013172336940](D:\document\img\image-20211013172336940.png)

![image-20211013174145414](D:\document\img\image-20211013174145414.png)

![image-20211013174904117](D:\document\img\image-20211013174904117.png)



![image-20211013174224684](D:\document\img\image-20211013174224684.png)

![image-20211013175150976](D:\document\img\image-20211013175150976.png)

![image-20211013175227230](D:\document\img\image-20211013175227230.png)

![image-20211013175312785](D:\document\img\image-20211013175312785.png)

