
+ String
+ List
+ Map
+ Set
+ SortedSet(zset)


https://cloud.tencent.com/developer/news/645221

# String
> 字符串类型的值可以是字符串、复杂的json/xml、数字甚至是二进制（图片、音频、视频），但是值不能超过512M

## 命令
+ set
    + setex: 设置过期时间
    + setnx: 键必须不存在，才可以设置成功，是分布式锁的一种实现方案

+ mset:批量设置值（mset key value [key value .....]）