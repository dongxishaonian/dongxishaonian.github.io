[返回上层](index)

# springboot中实现kafka指定offset消费

kafka消费过程难免会遇到需要重新消费的场景，例如我们消费到kafka数据之后需要进行存库操作，若某一时刻数据库down了，导致kafka消费的数据无法入库，为了弥补数据库down期间的数据损失，有一种做法我们可以指定kafka消费者的offset到之前某一时间的数值，然后重新进行消费。

##首先创建kafka消费服务
```java
@Service
@Slf4j
//实现CommandLineRunner接口，在springboot启动时自动运行其run方法。
public class TspLogbookAnalysisService implements CommandLineRunner {
    @Override
    public void run(String... args) {
        //do something
    }
}
```

##kafka消费模型建立

kafka server中每个主题存在多个分区（partition），每个分区自己维护一个偏移量（offset），我们的目标是实现kafka consumer指定offset消费。

在这里使用consumer-->partition一对一的消费模型，每个consumer各自管理自己的partition。


![kafka consumer partition](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-12-042032.png)



```java
@Service
@Slf4j
public class TspLogbookAnalysisService implements CommandLineRunner {
    //声明kafka分区数相等的消费线程数，一个分区对应一个消费线程
    private  static final int consumeThreadNum = 9;
    //特殊指定每个分区开始消费的offset
    private List<Long> partitionOffsets = Lists.newArrayList(1111,1112,1113,1114,1115,1116,1117,1118,1119);
   
    private ExecutorService executorService = Executors.newFixedThreadPool(consumeThreadNum);

    @Override
    public void run(String... args) {
        //循环遍历创建消费线程
        IntStream.range(0, consumeThreadNum)
                .forEach(partitionIndex -> executorService.submit(() -> startConsume(partitionIndex)));
    }
}
```

##kafka consumer对offset的处理

###声明kafka consumer的配置类
```java
private Properties buildKafkaConfig() {
    Properties kafkaConfiguration = new Properties();
    kafkaConfiguration.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "");
    kafkaConfiguration.put(ConsumerConfig.GROUP_ID_CONFIG, "");
    kafkaConfiguration.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, "");
    kafkaConfiguration.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "");
    kafkaConfiguration.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "");
    kafkaConfiguration.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "");
    kafkaConfiguration.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG,"");
    kafkaConfiguration.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "");
    ...更多配置项

    return kafkaConfiguration;
}
```

###创建kafka consumer，处理offset，开始消费数据任务

```java
private void startConsume(int partitionIndex) {
    //创建kafka consumer
    KafkaConsumer<String, byte[]> consumer = new KafkaConsumer<>(buildKafkaConfig());

    try {
        //指定该consumer对应的消费分区
        TopicPartition partition = new TopicPartition(kafkaProperties.getKafkaTopic(), partitionIndex);
        consumer.assign(Lists.newArrayList(partition));

        //consumer的offset处理
        if (collectionUtils.isNotEmpty(partitionOffsets)  &&  partitionOffsets.size() == consumeThreadNum) {
            Long seekOffset = partitionOffsets.get(partitionIndex);
            log.info("partition:{} , offset seek from {}", partition, seekOffset);
            consumer.seek(partition, seekOffset);
        }
        
        //开始消费数据任务
        kafkaRecordConsume(consumer, partition);
    } catch (Exception e) {
        log.error("kafka consume error:{}", ExceptionUtils.getFullStackTrace(e));
    } finally {
        try {
            consumer.commitSync();
        } finally {
            consumer.close();
        }
    }
}
```

###消费数据逻辑，offset操作

```java
private void kafkaRecordConsume(KafkaConsumer<String, byte[]> consumer, TopicPartition partition) {
    while (true) {
        try {
            ConsumerRecords<String, byte[]> records = consumer.poll(TspLogbookConstants.POLL_TIMEOUT);
            //具体的处理流程
            records.forEach((k) -> handleKafkaInput(k.key(), k.value()));

            //🌿很重要：日志记录当前consumer的offset，partition相关信息(之后如需重新指定offset消费就从这里的日志中获取offset，partition信息)
            if (records.count() > 0) {
                String currentOffset = String.valueOf(consumer.position(partition));
                log.info("current records size is：{}, partition is: {}, offset is:{}", records.count(), consumer.assignment(), currentOffset);
            }
    
            //offset提交        
            consumer.commitAsync();
        } catch (Exception e) {
            log.error("handlerKafkaInput error{}", ExceptionUtils.getFullStackTrace(e));
        }
    }
}
```

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-12-042038.jpg)


---
---
---


## 🤔  💭 👇👇👇

<script src="https://utteranc.es/client.js"
        repo="dongxishaonian/issue-posted"
        issue-term="pathname"
        label="🙂🙃😡🥶😬🤣😄"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>

