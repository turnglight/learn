# JAVA/J2EE求职面试指南

## JAVA 基础

### 1.使用java的理由

* 持多线程, socket通信, 内存管理
* 面向对象编程
* 移植性强
* 支持web应用,分布式应用,支持HTTP/JRMP(有广泛的标准API)等网络

### 2.java平台与其它平台有什么不同之处

 * java是一个纯软件平台,其运行在linux\windows等基于硬件的操作系统上
 * java有两大组件
    * **Java Virsual Machine**: jvm虚拟机,它是一个可以被各种不同的硬件平台进行移植的软件.字节码(byte)是它的机器语言.
    * **Java Application Programing Interface**

### 3.java packages有什么用

 * 它帮助解决在不同路径下具有相同类名的冲突问题
 * 同时能组织管理项目中的文件
 * 举例:java.io包中有一些功能需要用到java.net包中的类,如果把net包下面的文件全部放在一个包下面,那么管理起来就像一场噩梦.

### 4.解释java类加载器,如果你有一个类,你需要怎么来启动它,解释动态类的加载

 * 类加载器是分层的

### 5.方法重载与方法覆盖有什么区别?

 * 重载表示在同一个类中,具有多个相同方法名称但是具有不同的方法签名,主要是为不同的数据采用不同的方式定义相同的操作.
 * 覆盖表示的是在父子关系的类中,存在相同签名和名称的方法.主要是为了实现在不同的对象类型中采用不同的方式实现相同的操作.

### 6.Vector与ArrayList主要有什么不同?HashMap与HashTable有什么不同?栈和队列的不同之处是什么?

 * Vector/HashMap
   * 同步的,线程安全的
 * ArrayList/HashMap
   * 线程不安全的
   * 实现线程安全可以通过java.util.concurrent包中的ConcurrentHashMap,ConcurrentLinkedQueue,CopyOnWriteArrayList, CopyOnWriteArrayList(写时复制,采用的读写分离的思想,当需要修改元素时,复制一份容器,然后往新的容易中添加元素,而旧的容器新元素没有添加完毕之前,仍然在使用,新元素添加完毕之后,将旧容器引用修改为新容器,旧容器将会被GC)
* stack/queue
   * 栈只能访问最后一个被插入的元素
   * 队列是先进现出

### 7. 解释一下java的集合

 * 集合框架被用到的关键接口是List, Set, Map. List和Set都继承了Collection接口.
   * Set(HashSet, TreeSet)
     * Set是一个不允许有相同元素的集合.
     * TreeSet是一个有序HashSet, 它实现了SortedSet接口
   * List(ArrayList,LinkedList,Vector)
     * List可以存在重复的元素,且是有序的
   * Map(HashMap, TreeMap, HashTable)
     * Map是一个键值映射的对象, 能存在相同的值,但是键必须保持唯一性
     * TreeMap是一个有序的HashMap, 它实现了SortedMap

### 怎样实现集合的排序?

 * SortedSet, SortedMap接口维护排序顺序.
    * 通过实现Comparable, 对集合中的元素进行自然排序操作.
    * 通过实现Comparator,可对集合中的多个元素进行控制排序.

``` 
Comparable -> a.compareTo(b)==0  等价于    a.equals(b)
```

