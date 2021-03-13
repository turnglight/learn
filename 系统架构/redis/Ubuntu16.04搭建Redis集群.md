# Ubuntu16.04搭建Redis4.0.2集群

## 1.下载安装redis-4.0.2
+ 下载redis-4.0.2
+ tar -zxvf redis-4.0.2.tar.gz

## 2.编译安装
+ cp -r redis-4.0.2 /usr/local/redis-4.0.2
+ cd /usr/local/redis-4.0.2
+ make（编译安装）

## 3.配置redis节点
### 3.1 创建节点目录
+ mkdir cluster（创建集群配置目录）
+ cd cluster（进入目录）
+ mkdir 23367（创建6个节点目录，分别对应不同的端口）
+ mkdir 23368
+ mkdir 23369
+ mkdir 23370
+ mkdir 23371
+ mkdir 23372
### 3.2 配置节点启动脚本
+ cd 23367（分别进入不同的节点目录，创建对应节点的redis.conf启动配置文件，vim redis.conf）
+ vim redis.conf
~~~shell
port 23367
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
~~~
+ redis-server redis.conf（进入节点目录启动节点，自动生成上面的nodes.conf文件，否则启动下面的cluster-start.sh脚本会报错，无法找到nodes.conf）
+ cd /usr/local/redis-4.0.2/cluster
+ vim cluster-start.sh
~~~shell
#!/bin/sh
cd /usr/local/redis-4.0.2/cluster/23367
nohup redis-server redis.conf > redis-server.out 2>&1 &
cd /usr/local/redis-4.0.2/cluster/23368
nohup redis-server redis.conf > redis-server.out 2>&1 &
cd /usr/local/redis-4.0.2/cluster/23369
nohup redis-server redis.conf > redis-server.out 2>&1 &
cd /usr/local/redis-4.0.2/cluster/23370
nohup redis-server redis.conf > redis-server.out 2>&1 &
cd /usr/local/redis-4.0.2/cluster/23371
nohup redis-server redis.conf > redis-server.out 2>&1 &
cd /usr/local/redis-4.0.2/cluster/23372
nohup redis-server redis.conf > redis-server.out 2>&1 &
~~~

## 4.集群部署
+ sudo apt-get install ruby ruby-dev（redis-trib.rb依赖于ruby）
+ cd /usr/local/redis-4.0.2/src（下面的redis-trib.rb在该目录下）
+ ./redis-trib.rb  create --replicas 1 127.0.0.1:23367 127.0.0.1:23368 127.0.0.1:23369 127.0.0.1:23370 127.0.0.1:23371 127.0.0.1:23372
+ redis-cli -h 192.168.0.160 -p 23367



