# FASTJSON

+ JSON
+ JSONArray
+ JSONObject

### maven依赖
~~~xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
</dependency>
~~~


## JSONObject
+ 实现了MAP接口，主要是以键值对进行存储

## JSONArray
+ 内部是通过List接口中的方法进行操作的。Array顾名思义代表json对象数组

## JSON
+ 主要是实现json对象的转化，最后的数据获取还是要通过JSONObject和JSONArray来实现

### 案例
+ 字符串转JSONObject
    + JSONObject json = JSON.parseObject(str)
+ JSONObject转字符串
    + String str = JSON.toJSONString(jsonObject)
+ 字符串转JSONArray
    + JSONArray jsonArray = JSON.parseArray(str)
+ JSONArray转字符串
    + String str = JSON.toJSONString(jsonArray)
+ 获取JSON对象中的子JSON对象
    + JSONObject sonObject = jsonObject.getJSONObject("?")
    + JSONArray sonArray = jsonObject.getJSONArray("?")
+ JSON转JavaBean
    + Student student = JSON.parseObject(str, Student.class)
+ JavaBean转JSON字符串
    + String str = JSON.toJSONString(jsonObject)
