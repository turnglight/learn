# 反应式编程（ReactiveProgramming）
 > 两种编程风格
 + 命令式
    + 顺序执行
    + 阻塞
 + 反应式
    + 发布-订阅
    + 流失，非阻塞
    + 异步

## 反应式流（ReactiveStreams）
> 无阻塞回压的异步流处理标准，由此自然地，异步->并行->回压，并行处理涉及到一个并发量的问题，
> 反应式流规范定义了回压机制，当服务端压力过大，会向调用方进行反馈。这也就是背压（回压）机制。


## 接口规范(发布-订阅)
+ Publisher(发布者)
    + void subscribe(Subscriber<? super T> subscriber)
+ Subscriber(消费者)
    + void onSubscribe(Subscription sub)
    + void onNext(T item)
    + void onError(Throwable able);
    + void onComplete();
+ Subscription(消费者生命周期管理)
    + void request(long n);
    + void cancel();
+ Processor(满足发布-订阅的一种约定处理器，是发布者，也是消费者)
    + extends Subscriber<T>, Publisher<R>

> Publisher是发布者，Subscriber是订阅者，通过调用Publisher.subscriber(?)，传入subscriber
> 进行消息的订阅。发布者生产消息后，调用订阅者的Subscriber.onSubscribe方法，并传入Subscription对象，
> Subscription.request可以控制消息量，这是回压产生作用的地方；同时也可以调用cancel方法，来取消订阅。

## Mono & Flux
> 都实现了反应式流的Publisher接口
+ 几种常见操作
    + 创建操作
    + 组合操作
    + 转换操作
    + 逻辑操作

### Flux
> 代表具有一个或者多个数据项的管道
+ 创建
~~~java
// 创建管道，并添加数据
Flux<String> fruitFlux = Flux.just("apple", "Orange", "Grape", "Banana");
// 添加订阅者，相当于打开水龙头
fruitFlux.Subscribe(f -> System.out.println("Here's some fruit: " + f));

// 还可以通过
fruitFlux = Flux.fromArray(...)
fruitFlux = Flux.fromStream
~~~

### Mono
> 数据项不超过一个的数据流管道
