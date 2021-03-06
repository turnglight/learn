# Elasticsearch入门

## Elasticsearch是什么？

+ Elasticsearch(ES)是一个机遇Lucene构建的开源、分布式、RESTFUL接口全文搜索引擎。
+ Elasticsearch还是一个分布式文档数据库，其中每个字段均是被索引的数据且可被搜索，能够扩展至数以百计的服务器存储以及处理PB级的数据。
+ 可以在很短的时间内存储、搜索和分析大量的数据。
+ 通常作为具有复杂搜索场景情况下的核心发动机。

## Elasticsearch能做什么？

+ 存储数据，提供精准搜索
+ 日志收集统计分析
+ 设置规则值，数据定时过滤器处理，达到规则值进行通知
+ 数据分析可视化，利用Kibana建立自定义的仪表板

## Elasticsearch为什么被需要？有什么特性？

+ 横向可扩展性
  + 只需要增加一台服务器，做一点儿配置，启动一下Elasticsearch进程就可以并入集群
+ 分片机制提供更好的分布性
  + 同一个索引分成多个分片（sharding），这点类似于HDFS的块机制；分而治之的方式可提升处理效率
+ 高可用
  + 提供复制（replica）机制，一个分片可以设置多个复制，使得某太服务器在宕机的情况下，集群仍旧可以照常运行，并会吧服务器宕机丢失的数据信息复制恢复到其他可用节点上
+ 使用简单
  + 只需要一条命令就可以下载文件，然后很快就能搭建一个站内搜索引擎

## 相关产品

### Beats

+ 它是一个代理，将不同类型的数据发送到Elasticsearch。主要由三部分组成：Filebeat、Topbeat、Packetbeat
  + Filebeat用来收集日志
  + Topbeat用来收集系统基础设置数据
  + Packetbeat是一个网络包分析工具，统计收集网络信息

### Shield

+ 它为Elasticsearch带来企业级的安全性，加密通信、认证保护整个Elasticsearch数据，它是基于角色的访问控制与审计。
+ 收费

### Watcher

+ 它是Elasticsearch的警报和通知工具

### Marvel

+ 它是Elasticsearch的管理和监控工具。它检测Elasticsearch集群索引和节点的活动，快速诊断问题。
+ 收费

## 全文搜索

~~~remark
全文搜索是指计算机搜索程序通过扫描文章中的每一个词，对每一个词建立索引，指明该词在文章中出现的次数和位置，当用户查询时，搜索程序就根据事先建立的索引进行查找。
~~~

### lucene

#### Lucene倒排索引

+ 根据属性的指查找记录
  + 这种索引表中的每一项都包括一个属性值和具有该属性值得各记录的地址
  + 由于不是由记录来确定属性值，而是相反根据属性值来确定记录的位置，因而称为`倒排索引`
  + 带有倒排索引的文件，称为倒排索引文件，简称倒排文件

~~~remark
搜索引擎的关键步骤就是建立倒排索引，倒排索引一般表示为一个关键词，然后是他的频度（出现的次数）、位置
~~~

![1553396442433](C:\Users\Dyn\AppData\Roaming\Typora\typora-user-images\1553396442433.png)

### 压缩算法

~~~remark
为了减少索引文件的大小，Lucene对索引还使用了压缩技术
~~~

## 基础知识

### Elasticsearch术语与概念

#### 索引词（term）

+ 在Elasticsearch中索引词是一个能够被索引的精确值。

#### 文本(text)

+ 文本是一段普通的非结构化文字。文本通常会被分析称一个个的所因此，存储在Elasticsearch的索引库中。为了让文本能够进行搜索，文本字段需要实现进行分析。

#### 分析(analysis)

+ 分析是将文本转换为索引词的过程，分析的结构依赖于分词器。

#### 集群(cluster)

+ 集群有一个或多个节点组成，对外提供服务
+ 在所有节点中，一个集群有一个唯一的名称默认为`Elasticsearch`
+ 节点只能是集群的一部分，当该节点被设置成相同的集群名称时，就会自动加入集群
+ 一个节点只能加入一个集群

#### 节点(node)

+ 一个节点是一个逻辑上独立的服务，它是集群的一部分
+ 每个节点可以有唯一的名字
+ 在网络中Elasticsearch集群通过节点名称进行管理和通信

#### 路由(routing)

+ 当存储一个文档的时候，它会存储在唯一的主分片中，具体哪个分片是通过散列值进行选择的

#### 分片(shared)

+ 这是Elasticsearch管理的比较底层的功能
+ 索引是知香主分片和副本分片的逻辑空间

##### 主分片(primary shard)

+ 每个文档都存在在一个分片中，当你存在一个文档的时候，首先存储在主分片中，然后会复制到不同的副本中
+ 一个索引可以有5个主分片，可是事先制定分片的数量，一旦建立，则不能修改

##### 副本分片(replica shard)

主要由两个目的

+ 增加共可用
  + 主分片失败，可以从副本分片中选择一个座位主分片
+ 提高性能
  + 副本分片必须部署在不同与主分片的节点上，不能部署在和主分片的节点上

分片主要由两个重要原因

+ 允许水平分割扩展数据
+ 允许分配和并行操作，从而提高性能和吞吐量

#### 复制(replica)

~~~remark
复制是非常有用的功能，不然会有单点的问题。当网络中的某一个节点出现问题的时候，复制可以对故障进行转移，保证系统的高可用。
~~~

复制是重要的，主要两个原因

+ 提供了高可用。需要注意的是，一个复制的分片不会存储在同一个节点上。
+ 允许扩展搜索量，提高并发量

~~~remark
默认情况下，每个索引分配5个分片和一个副本，这意味着你的集群节点至少要有两个节点，你将拥有5个主要的分片和10个副本分片。
~~~

#### 索引(index)

+ 索引是具有相同结构的文档集合
+ 在系统上索引的名字全部小写，通过这个名字可以用来执行索引、搜索、更新、删除操作等

#### 类型（type）

+ 在索引中，可以定义一个或多个类型，类型是索引的逻辑分区

#### 文档（document）

+ 文档是存储在Elasticsearch中的一个JSON格式的字符串