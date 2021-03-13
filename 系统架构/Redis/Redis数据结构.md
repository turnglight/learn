
+ String
+ List
+ Map
+ Set
+ SortedSet(zset)


https://cloud.tencent.com/developer/news/645221

# String
> 字符串类型的值可以是字符串、复杂的json/xml、数字甚至是二进制（图片、音频、视频），但是值不能超过512M

## 命令
+ set key value:设置KEY的值
    + setex key seconds key: 设置过期时间
    + setnx key value: 键必须不存在，才可以设置成功，是分布式锁的一种实现方案
+ del key [key ....]
+ mset key value [key value .....]:批量设置值
+ 批量获取值（mget key [key .....]:批量获取值
+ incr key:自增操作
    + incrby key value:指定自增数值
    + incrbyfloat key value:指定自增浮点数
+ decr key:自减操作
    + descby key value:指定自增数值
+ append key value:追加值
+ strlen key:获取字符串长度，这个长度是字节长度，不是字符长度
+ getset key value:设置新值，并返回旧值
+ setrange key offeset value：设置指定位置的值
+ getrange key start end:或者指定位置的值

|命令|时间复杂度|
|---|------------------------|
|set|o(1)|
|get|o(1)|
|incr|o(1)|
|decr|o(1)|
|incrby|o(1)|
|decrby|o(1)|
|incrbyfloat|o(1)|
|append|o(1)|
|setrange|o(1)|
|getrange|o(n)、n是字符串的长度，获取字符串是非常快的，所以如果字符串不是很长，可以视同为o(1)|
|del|O(k),k为键的个数|
|mset|O(k),k为键的个数|
|mget|O(k),k为键的个数|

## 应用场景
+ 计数器
+ session共享
+ 数据缓存
+ 短信访问限速（setex incr）
+ 


