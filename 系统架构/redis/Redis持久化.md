
Redis提供了两种不同的持久化方式
+ RDB(SNAPSHOTTING:Redis DataBase)
    + 
    + 在指定的时间间隔能对你的数据进行快照存储
+ AOF(append only file)
    + 对redis的写操作会被AOF文件记录，当服务器重启时，会重新执行这些命令来恢复原始的数据

# RDB

## 三种触发机制
+ save
    + 阻塞型触发方式，当执行save命令期间，主进程不能处理命令，直到rdb过程结束，相当于JVM在垃圾回收时的stop the world
+ bgsave(save m n)
    + 非阻塞的异步触发方式，当执行bgsave命令期间，主进程依然能继续处理其它命令，它只在fork子进程时，进行短暂的暂停，时间很短。
    + 基本上Redis内部所有的RDB都采用bgsave命令
+ 自动触发bgsave
    + 通过在redis.conf中进行配置

## RDB优点
+ 速度快
+ 文件紧凑，全量备份，非常适合用于进行备份和恢复
+ 生成RDB时，redis主进程会fork一个子进程来处理保存工作，主进程不需要进行任何IO操作

## RDB缺点
+ 可能会丢失数据
    + 持久化的工作是由子进程处理，在持久化的过程中，如果IO类型是异步的，那么主进程还是可以接受任务，此时发生的数据变更，子进程无法感知，所以在备份的过程中RDB模式会导致少量的数据丢失。


# AOF

## 三种触发机制
+ 手动触发：bgrewriteaof
+ 自动触发：通过设置下面两个参数来设置
    + auto-aof-rewrite-percentage:代表当前AOF文件空间
    + auto-aof-rewrite-min-size:AOF重写的文件最小体积


## 文件同步的三种策略
+ always
    + 每一个命令更新都触发一次数据变更日志的磁盘同步，性能较差，数据完整性较好
+ everysec
    + 每一秒触发一次数据变更日志的磁盘同步，异步操作。如果在一秒内系统宕机，这一秒内的数据被丢失
+ no
    + 从不同步，不可控

## 开启AOF
> appendonly yes
> 默认AOF不开启，默认文件名为appendonly.aof

工作流程如下：
+ 命令写入(append)
    + 命令的写入并非直接写入到硬盘，而是先append到aof_buffer缓冲区，这样响应命令的性能就可以不用依赖与硬盘的读写
+ 文件同步(fsync)
    + 
+ 文件重写
+ 重新加载

## AOF优点
+ 数据完整性较RDB更好
+ 由于又完整的日志记录，所以非常适合做灾难性的误删除操作
+ 恢复数据速度较RDB慢

## AOF缺点
+ 日志文件通常较大
+ 支持的写QPS比RBD支持的QPS要低，因为AOF一般会配置成每秒fsync（内存与硬盘的同步）一次日志文件


> 通常两者加在一起使用较好



## 常用配置
RDB持久化配置
Redis会将数据集的快照dump到dump.rdb文件中。此外，我们也可以通过配置文件来修改Redis服务器dump快照的频率，在打开6379.conf文件之后，我们搜索save，可以看到下面的配置信息：

save 900 1              #在900秒(15分钟)之后，如果至少有1个key发生变化，则dump内存快照。

save 300 10            #在300秒(5分钟)之后，如果至少有10个key发生变化，则dump内存快照。

save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，则dump内存快照。

AOF持久化配置
在Redis的配置文件中存在三种同步方式，它们分别是：

appendfsync always     #每次有数据修改发生时都会写入AOF文件。

appendfsync everysec  #每秒钟同步一次，该策略为AOF的缺省策略。

appendfsync no          #从不同步。高效但是数据不会被持久化。
