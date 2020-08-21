---
title: Kafka事务性实现
tagline: ""
category : kafka
layout: post
tags : [kafka]
---
[KIP-98 - Exactly Once Delivery and Transactional Messaging](https://cwiki.apache.org/confluence/display/KAFKA/KIP-98+-+Exactly+Once+Delivery+and+Transactional+Messaging)
![Transactional Messaging](https://github.com/2pc/2pc.github.io/tree/master/_posts/images/kafka_tx.png)

事务消息示例：
```
// Init transactions call should always happen first in order to clear zombie transactions from previous generation.
//1. 初始事务
producer.initTransactions();
// Begin a new transaction session.
//2. 开始一个事务操作
producer.beginTransaction();
//3. 发送消息
for (ConsumerRecord<Integer, String> record : records) {
    // Process the record and send to downstream.
    ProducerRecord<Integer, String> customizedRecord = transform(record);
    producer.send(customizedRecord);
}
Map<TopicPartition, OffsetAndMetadata> offsets = consumerOffsets();

// Checkpoint the progress by sending offsets to group coordinator broker.
// Note that this API is only available for broker >= 2.5.
producer.sendOffsetsToTransaction(offsets, consumer.groupMetadata());

// Finish the transaction. All sent records should be visible for consumption now.
//commit事务
producer.commitTransaction();
```

initTransactions 依据transactionId 获取PID与epoch

```
public void initTransactions() {
    throwIfNoTransactionManager();
    throwIfProducerClosed();
    TransactionalRequestResult result = transactionManager.initializeTransactions();
    sender.wakeup();
    result.await(maxBlockTimeMs, TimeUnit.MILLISECONDS);
}
//transactionManager
synchronized TransactionalRequestResult initializeTransactions(ProducerIdAndEpoch producerIdAndEpoch) {
    boolean isEpochBump = producerIdAndEpoch != ProducerIdAndEpoch.NONE;
    return handleCachedTransactionRequestResult(() -> {
        // If this is an epoch bump, we will transition the state as part of handling the EndTxnRequest
        if (!isEpochBump) {
            transitionTo(State.INITIALIZING);
            log.info("Invoking InitProducerId for the first time in order to acquire a producer ID");
        } else {
            log.info("Invoking InitProducerId with current producer ID and epoch {} in order to bump the epoch", producerIdAndEpoch);
        }
        //发送InitProducerIdRequest
        InitProducerIdRequestData requestData = new InitProducerIdRequestData()
                .setTransactionalId(transactionalId)
                .setTransactionTimeoutMs(transactionTimeoutMs)
                .setProducerId(producerIdAndEpoch.producerId)
                .setProducerEpoch(producerIdAndEpoch.epoch);
        InitProducerIdHandler handler = new InitProducerIdHandler(new InitProducerIdRequest.Builder(requestData),
                isEpochBump);
        enqueueRequest(handler);
        return handler.result;
    }, State.INITIALIZING);
    }

```

producer(transactionalId-->PID)

AddPartitionsToTxnRequest

TxnOffsetCommitRequset

