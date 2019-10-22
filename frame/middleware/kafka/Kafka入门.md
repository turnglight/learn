
>Kafka是最初由Linkedin公司开发，是一个分布式、分区的、多副本的、多订阅者，基于zookeeper协调的分布式日志系统（也可以当做MQ系统），常见可以用于web/nginx日志、访问日志，消息服务等等，Linkedin于2010年贡献给了Apache基金会并成为顶级开源项目。

## 消息传递模式
+ 点对点消息传递模式
> 生产者发送一条消息到queue，只有一个消费者能收到。
<img src="https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180507190326476-771565746.png">
+ 发布-订阅消息传递模式
> 发布者发送到topic的消息，只有订阅了topic的订阅者才会收到消息。
><img src="https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180507190443404-1266011458.png">


## 术语解释
+ 术语关系图
<img src="https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180507190731172-1317551019.png">
> + broker(kafka集群包含多个kafka服务，每个服务节点都有唯一的一个brokerid)
> + topic(消息主题类别，将不同的消息用topic进行主题区分)
> + partition(每个topic至少有一个partition，在集群部署条件下，一个topic的多个partition会被存储在不同的broker节点中，尽量避免broker节点数少于partition数，防止数据存储不均的问题)
> + producer(消息生产者，指外部调用kafka服务的应用)
> + consumer(消费者)
> + consumer group
> + follower(Follower跟随Leader，所有写请求都通过Leader路由，数据变更会广播给所有Follower，Follower与Leader保持数据同步。如果Leader失效，则从Follower中选举出一个新的Leader。当Follower与Leader挂掉、卡住或者同步太慢，leader会把这个follower从“in sync replicas”（ISR）列表中删除，重新创建一个Follower。)
> + leader(每个partition有多个副本，其中有且仅有一个作为Leader，Leader是当前负责数据的读写的partition)

详情：<a href="https://www.cnblogs.com/qingyunzong/p/9004509.html">https://www.cnblogs.com/qingyunzong/p/9004509.html</a>