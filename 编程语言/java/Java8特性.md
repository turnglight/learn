+ Java语言的新特性
+ Java编译器的新特性
+ Java官方库的新特性
+ 新的Java工具
+ JVM的新特性

# 1.Java语言的新特性

## 1.1 Lambda表达式和函数式接口
 + Lambda表达式（也称为闭包）是Java 8中最大和最令人期待的语言改变。它允许我们将函数当成参数传递给某个方法，或者把代码本身当作数据处理：函数式开发者非常熟悉这些概念。

## 1.2 接口的默认方法和静态方法

## 1.3 方法引用

## 1.4 重复注解

## 1.5 更好的类型推断

## 1.6 扩展注解的应用场景

# 2.Java编译器的新特性
## 2.1 参数名称

# 3.Java官方库的新特性
## 3.1 Optional
## 3.2 Steams
+ 新增的Stream API（java.util.stream）将生成环境的函数式编程引入了Java库中。这是目前为止最大的一次对Java库的完善，以便开发者能够写出更加有效、更加简洁和紧凑的代码。Steam API极大得简化了集合操作。

## 3.3 Date/Time API(JSR 310)
+ Java 8引入了新的Date-Time API(JSR 310)来改进时间、日期的处理。时间和日期的管理一直是最令Java开发者痛苦的问题。java.util.Date和后来的java.util.Calendar一直没有解决这个问题（甚至令开发者更加迷茫）。

## 3.4 Nashorn JavaScript
## 3.5 Base64
+ 对Base64编码的支持已经被加入到Java 8官方库中，这样不需要使用第三方库就可以进行Base64编码

## 3.6 并行数组
## 3.7 并发性

# 4.新的Java工具
## 4.1 Nashorn引擎：jjs
## 4.2 类依赖分析器：jdeps

# 5.JVM的新特性
+ 使用Metaspace代替持久代（PermGen space）。在JVM参数方面，使用-XX:MetaSpaceSize和-XX:MaxMetaspaceSize代替原来的-XX:PermSize和-XX:MaxPermSize。


