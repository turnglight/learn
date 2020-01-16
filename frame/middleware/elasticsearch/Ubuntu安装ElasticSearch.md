# ElasticSearch安装和部署

## 1.下载ElasticSearch安装包
 + https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-4-3

## 2.安装准备
+ 上传至服务器
+ 解压
+ mv elasticsearch-6.4.3 /usr/local/elasticsearch

## 3.安装
+ 添加新用户
    + useradd elasticUser
+ 给新用户授权
    + chown -R elasticUser:elasticUser /usr/local/elasticsearch/
    + cat /etc/group
+ 设置用户最大打开文件数
    + cd /etc/security
    + vim limits.conf
    ~~~
        elasticUser soft nofile 65536
        elasticUser hard nofile 65536
    ~~~
+ 修改用户最大内存大小权限
    + vim /etc/sysctl.conf
    ~~~
        vm.max_map_count=262144
    ~~~
    + sysctl -p

## 4.配置
+ cd /usr/local/elasticsearch/config
+ vim elasticsearch.yml
在文件最后加入以下几句：
~~~
http.cors.enabled: true
http.cors.allow-origin: "*"
 
cluster.name: lcc
node.name: master
node.master: true
 
network.host: 0.0.0.0
discovery.zen.ping.unicast.hosts: ["你服务器的IP地址"]
~~~

## 5.测试
+ su elasticUser
+ 启动elasticsearch
    + cd /usr/local/elasticsearch/bin
    + ./elasticsearch -d
    + ps -ef | grep elasticsearch
    + curl http://localhost:9200
