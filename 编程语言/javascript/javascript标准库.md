# https://wangdoc.com/javascript/

## Object

## Array

+ 静态方法
    + Array.isArray(obj)
+ 实例方法
    + valueOf(), toString()
    + push(): 在末尾push元素，并返回数组长度
    + pop()：pop最后一个元素，并返回该元素
    + shift()：删除第一个元素，并返回该元素
    + unshift()： 在开始位置添加元素，并返回长度 
    + join(): 指定参数作为分隔符，并返回分隔后的字符串
    + concat(): 数组合并
    + reverse(): reverse方法用于颠倒排列数组元素
    + slice(start, end): 提取部分数组元素，并返回，原数组不变
    + splice(start, count, addElement1, addElelment2, ...): 删除部分元素，并可以添加新元素，返回新产生的数组，改方法会改变原数组
    + sort(function): 对数组成员进行排序，默认是按照字典顺序排序，排序后，原数组会改变 
        + 自定义需要实现function
    + map(function)：对每一个元素执行function，返回新结果集 
    + filter(function): 对数据进行过滤
    + some(), every()：类似断言
    + reduce(), reduceRight(): 累积计算
    + indexOf, lastIndexOf()
+ 链式使用
~~~javascript
var users = [
  {name: 'tom', email: 'tom@example.com'},
  {name: 'peter', email: 'peter@example.com'}
];

users
.map(function (user) {
  return user.email;
})
.filter(function (email) {
  return /^t/.test(email);
})
.forEach(function (email) {
  console.log(email);
});
~~~

## Boolean

## Number
#### 实例方法
+ Number.prototype.toString()
+ Number.prototype.toFixed(): 指定小数的位数
+ Number.prototype.toExponential(): 指定科学计数法
+ Number.prototype.toPrecision(): 将一个数转为指定位数的有效数字
+ Number.prototype.toLocaleString()：指定数字当地书写格式

## String

#### 静态方法
+ String.fromCharCode(): 返回码点字符串
~~~javascript
String.fromCharCode() // ""
String.fromCharCode(97) // "a"
String.fromCharCode(104, 101, 108, 108, 111)
// "hello"
~~~

#### 实例方法

+ String.prototype.charAt(): 返回指定位置字符串
+ String.prototype.charCodeAt(): 返回指定位置字符码点
+ String.prototype.concat()：连接两个字符串，返回新字符串
+ String.prototype.slice(start, end)：删除部分字符，参数方式与substring类型
+ String.prototype.substring(start, end)
+ String.prototype.substr()
+ String.prototype.indexOf(), String.prototype.lastIndexOf()
+ String.prototype.trim(): 去除字符串两端空格，返回一个新字符串，不改变原字符串
+ String.prototype.toLowerCase(), String.prototype.toUpperCase()
+ String.prototype.match()
+ String.prototype.search(), String.prototype.replace()
+ String.prototype.split()：给分隔符，对字符串进行分隔，返回一个由分割出来的字符串组成的数组
+ String.prototype.localeCompare()：字符串比较


## Math

## Date

## RegExp

## JSON
+ JSON.stringify()
+ JSON.parse()