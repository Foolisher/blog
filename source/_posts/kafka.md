---
title: kafka
date: 2018-01-04 15:34:55
tags:
  - 分布式
  - kafka
---



## 数据可靠性

> 如何保证数据可靠

### 分区冗余

分区冗余解决的是可靠性服务问题，一个Leader提供读写服务，当一个Leader宕机，其他备机可以顶上称为Leader继续提供读写服务

每个Partition有多个followers，每个follower不断从leader拉取已提交的Message，但是如果Follower落后Leader 10 秒钟，那么这个Follower会被剔除出in-sync队列

### 动态选主

#### Leader失效

- 是否允许 out-of-sync 参与选主

- `Leader`失效，如何找到一个最可靠的 `Follower` 作为新 `Leader`？

  - 需要保持完全同步，一条消息 `commit`后，`Followers`们完全同步后才返回成功
- 有一种逻辑控制最小同步replications数量

### 防止消息丢失

- 消息发送，主机commit，Followers 没有同步。可以设置 acks>=3 一部分Followers都同步了消息才返回成功，保证Leader宕机后Follower跟得上最新的消息，但是有个问题是Leader宕机后选主一定要从水位最新的几个Followers里面选
- 不要允许unclean leader election，acks=all, 以为者一条消息必须所有replicas都同步了才返回Producer成功，但此时若消息发送失败将返回“Leader not Available”，若Producer没有重试策略，这条消息将丢失！

### 防止消息重复消费

当网络抖动或服务器故障时消息发送失败，应当妥善处理重发策略，如果是服务器暂时不服务消息落地，应该可以重试，但是如果服务不可达（这个错误不是服务器抛出的），也可以重试，但还是不可避免消息其实已经送达，造成消息重复发送。

要彻底解决消息重复问题，还是需要Broker做幂等处理，或者消费者做幂等消费。

Case：为一个账户 Add:10y，冲发送多次会造成什么影响呢，一个处理办法就是，每次Add指令都有一个唯一的上下文，比如退款单ID，每次退款给用户就会执行 Add:10Y，消费者接收到消息后，判断这个退款单是不是已经处理过了，若已处理认为这是重复消费，弃之



## 消费者



### 如何避免重复消费

重复消费，是指获取到消息并成功消费后但是由于异常导致未成功commit offset导致那阶段($offset-1~$offset)消息重复读取。那么如何避免这种情况呢？



### 如何避免漏消费

如何消费者先commit offset，然后处理业务，但是业务未消费到消息，那么导致该消息未消费，因此先提交消息会导致重复消费；而先提交offset可能导致漏消息。

### enable.auto.commit

消费offset自动提交是指消费完一条消息后自动commi offset，与之对应的一个参数有 auto.commit.interval.ms，时间间隔越小，重复消费消息越少，但是还是无法完全避免重复消费

### Consumer Group 调整时发生什么

