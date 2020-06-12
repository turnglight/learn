<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [null/undefined](#nullundefined)
- [立即调用的函数表达式（IIFE）](#%E7%AB%8B%E5%8D%B3%E8%B0%83%E7%94%A8%E7%9A%84%E5%87%BD%E6%95%B0%E8%A1%A8%E8%BE%BE%E5%BC%8Fiife)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## null/undefined
> 将一个变量赋值为undefined或null，老实说，语法效果几乎没区别。
~~~javascript
if (!undefined) {
  console.log('undefined is false');
}
// undefined is false

if (!null) {
  console.log('null is false');
}
// null is false

undefined == null
// true
~~~

> 既然含义与用法都差不多，为什么要同时设置两个这样的值，这不是无端增加复杂度，令初学者困扰吗？这与历史原因有关。
> 1995年 JavaScript 诞生时，最初像 Java 一样，只设置了null表示"无"。根据 C 语言的传统，null可以自动转为0。

> Number(null) // 0

> 5 + null // 5

> 上面代码中，null转为数字时，自动变成0。

>但是，JavaScript 的设计者 Brendan Eich，觉得这样做还不够。首先，第一版的 JavaScript 里面，null就像在 Java 里一样，被当成一个对象，Brendan Eich 觉得表示“无”的值最好不是对象。其次，那时的 JavaScript 不包括错误处理机制，Brendan Eich 觉得，如果null自动转为0，很不容易发现错误。

>因此，他又设计了一个undefined。区别是这样的：null是一个表示“空”的对象，转为数值时为0；undefined是一个表示"此处无定义"的原始值，转为数值时为NaN。


```对于null和undefined，大致可以像下面这样理解。```

> null表示空值，即该处的值现在为空。调用函数时，某个参数未设置任何值，这时就可以传入null，表示该参数为空。比如，某个函数接受引擎抛出的错误作为参数，如果运行过程中未出错，那么这个参数就会传入null，表示未发生错误。

> undefined表示“未定义”，下面是返回undefined的典型场景。

~~~javascript
// 变量声明了，但没有赋值
var i;
i // undefined

// 调用函数时，应该提供的参数没有提供，该参数等于 undefined
function f(x) {
  return x;
}
f() // undefined

// 对象没有赋值的属性
var  o = new Object();
o.p // undefined

// 函数没有返回值时，默认返回 undefined
function f() {}
f() // undefined
~~~


## 立即调用的函数表达式（IIFE）

> 在 JavaScript 中，圆括号()是一种运算符，跟在函数名之后，表示调用该函数。比如，print()就表示调用print函数。

> 有时，我们需要在定义函数之后，立即调用该函数。这时，你不能在函数的定义之后加上圆括号，这会产生语法错误。
~~~javascript
function(){ /* code */ }();
// SyntaxError: Unexpected token (
~~~

> 产生这个错误的原因是，function这个关键字即可以当作语句，也可以当作表达式。

~~~javascript
// 语句
function f() {}
// 表达式
var f = function f() {}
~~~

> 为了避免解析上的歧义，JavaScript 引擎规定，如果function关键字出现在行首，一律解释成语句。因此，JavaScript 引擎看到行首是function关键字之后，认为这一段都是函数的定义，不应该以圆括号结尾，所以就报错了。

> 解决方法就是不要让function出现在行首，让引擎将其理解成一个表达式。最简单的处理，就是将其放在一个圆括号里面。

~~~javascript
(function(){ /* code */ }());
// 或者
(function(){ /* code */ })();
~~~

> 上面两种写法都是以圆括号开头，引擎就会认为后面跟的是一个表示式，而不是函数定义语句，所以就避免了错误。这就叫做“立即调用的函数表达式”（Immediately-Invoked Function Expression），简称 IIFE。

> 注意，上面两种写法最后的分号都是必须的。如果省略分号，遇到连着两个 IIFE，可能就会报错。

~~~javascript
// 报错
(function(){ /* code */ }())
(function(){ /* code */ }())
~~~

> 上面代码的两行之间没有分号，JavaScript 会将它们连在一起解释，将第二行解释为第一行的参数。

> 推而广之，任何让解释器以表达式来处理函数定义的方法，都能产生同样的效果，比如下面三种写法。

~~~javascript
var i = function(){ return 10; }();
true && function(){ /* code */ }();
0, function(){ /* code */ }();
~~~
> 甚至像下面这样写，也是可以的。
~~~javascript
!function () { /* code */ }();
~function () { /* code */ }();
-function () { /* code */ }();
+function () { /* code */ }();
~~~

> 通常情况下，只对匿名函数使用这种“立即执行的函数表达式”。它的目的有两个：一是不必为函数命名，避免了污染全局变量；二是 IIFE 内部形成了一个单独的作用域，可以封装一些外部无法读取的私有变量。
~~~javascript
// 写法一
var tmp = newData;
processData(tmp);
storeData(tmp);

// 写法二
(function () {
  var tmp = newData;
  processData(tmp);
  storeData(tmp);
}());
~~~
上面代码中，写法二比写法一更好，因为完全避免了污染全局变量。

## this关键字
> 详情： https://wangdoc.com/javascript/oop/this.html
#### 使用场合
+ 全局环境
+ 构造函数
+ 对象的方法
  + 这条规则很不容易把握

#### 使用注意点
+ 避免多层 this
  + var that = this;这条语句经常用于解决多层结构内this的运行环境不明确的问题
+ 避免数组处理方法中的this
  + var that = this;
  + this.p.foreach(funtion, this)将this作为第二个参数，固定它的环境变量
+ 避免回调函数中的this

#### 绑定this的方法
+ Function.prototype.call()
~~~javascript
var n = 123;
var obj = { n: 456 };

function a() {
  console.log(this.n);
}

a.call() // 123
a.call(null) // 123
a.call(undefined) // 123
a.call(window) // 123
a.call(obj) // 456
~~~


+ Function.prototype.apply()
~~~javascript

~~~

+ Function.prototype.bind()

