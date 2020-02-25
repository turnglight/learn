## 三者的区别
它们有什么区别呢？
>IBM 的软件架构师 Albert Barron 曾经使用披萨作为比喻，解释这个问题。David Ng 进一步引申，让它变得更准确易懂。请设想你是一个餐饮业者，打算做披萨生意。\
>你可以从头到尾，自己生产披萨，但是这样比较麻烦，需要准备的东西多，因此你决定外包一部分工作，采用他人的服务。你有三个方案。

+ 方案一：IaaS
他人提供厨房、炉子、煤气，你使用这些基础设施，来烤你的披萨。

+ 方案二：PaaS
除了基础设施，他人还提供披萨饼皮。你只要把自己的配料洒在饼皮上，让他帮你烤出来就行了。也就是说，你要做的就是设计披萨的味道（海鲜披萨或者鸡肉披萨），他人提供平台服务，让你把自己的设计实现。

+ 方案三：SaaS
他人直接做好了披萨，不用你的介入，到手的就是一个成品。你要做的就是把它卖出去，最多再包装一下，印上你自己的 Logo。

>上面的三种方案，可以总结成下面这张图，即将做披萨的各个流程分开，三个概念的不同体现在提供的功能的多寡上面。
><img src="https://img-blog.csdn.net/20170806151313100?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd29qaXVzaGl3bzk0NXlvdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast">

**从左到右，自己承担的工作量（上图蓝色部分）越来越少，IaaS > PaaS > SaaS。**

## 软件开发上的对应关系
将做披萨的一系列操作，对应到软件开发，则是下面这张图：
<img src="https://img-blog.csdn.net/20170806151542611?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd29qaXVzaGl3bzk0NXlvdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast">

+ **SaaS** 是软件的开发、管理、部署都交给第三方，不需要关心技术问题，可以拿来即用。普通用户接触到的互联网服务，几乎都是 SaaS。
+ **PaaS** 提供软件部署平台（runtime），抽象掉了硬件和操作系统细节，可以无缝地扩展（scaling）。开发者只需要关注自己的业务逻辑，不需要关注底层。
+ **IaaS** 是云服务的最底层，主要提供一些基础资源。它与 PaaS 的区别是，用户需要自己控制底层，实现基础设施的使用逻辑。
