# Reentrantlock重入锁

## 1.ReentrantLock的背景
> 要了解ReentrantLock之前，我们需要知道synchronized的最初版本。
### 1.1 synchronized & ReentrantLock
> JDK1.6之前
+ synchronized  重量级锁，调用操作系统函数
jvm中的线程与OS中线程一一对应，线程的切换，需要调用操作系统的用户态/内核态的状态切换。如果了解操作系统，会知道内核级的线程切换，相对jdk而言是非常消耗资源的。
> JDK1.6之后
+ synchronized改进后的锁，会根据锁的竞争情况进行锁的自动升级，偏向锁->轻量级锁->重量级锁, 1.6之后的状态切换，尽量不调用操作系统的函数，重量级的锁才调用系统级别的线程切换函数

> 1.6之前，由于synchronized的同步需要调用操作系统级别的函数，随后ReentrantLock诞生，也许是ReentrantLock的出现，刺激了它爸爸，1.7版本的synchronized进行了升级。它的实现与ReentrantLock类似。synchronized的实现是在JVM级别实现的。

### 1.2 ReentrantLock
+ 它是一款在JDK层面解决，线程资源竞争的解决方案，不涉及到内核级别的线程切换
+ 它提供了非常简便的API操作，较synchronized而言，使用更加简单明了
+ 它的使用较synchronized更有伸缩性，并解决了synchronized无法解决的一些资源竞争场景

## 2.ReentrantLock实际使用
~~~java
final ReentrantLock lock = new ReentrantLock();
new Thread(()->{
    lock.lock();
    ......
    ......
    lock.unlock();
})
~~~
> ReentrantLock的使用非常简单，new一个ReentrantLock对象，调用lock加锁，调用unlock解锁，下面我们主要来分析lock方法的源码

## 3.RentrantLock源码刨析

> final ReentrantLock lock = new ReentrantLock();
~~~java
public ReentrantLock() {
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
~~~
> ReentrantLock有两个构造函数，其中有一个参数fair，它是用来设置Reentrantlock是公平锁还是非公平锁，默认的ReentrantLock是一把非公平锁。

> 那么`公平锁和非公平锁有什么区别呢`？

+ 我们先来看看ReentrantLock这个类的整体结构
~~~java
ReentrantLock implements Lock, java.io.Serializable{
    private final Sync sync;
    abstract static class Sync extends AbstractQueuedSynchronizer { ... }
    static final class NonfairSync extends Sync{...}
    static final class FairSync extends Sync {...}
    public ReentrantLock() {sync = new NonfairSync();}
    public ReentrantLock(boolean fair) {sync = fair ? new FairSync() : new NonfairSync();}
    public void lock() {sync.lock();}
    public boolean tryLock() {...}
    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {...}
    public void unlock() {sync.release(1);}
    public Condition newCondition() {...}
    ...
}
~~~
+ 重入锁首先定义了一个内部抽象类`Sync`，它继承了AQS。Sync的内部主要定义了一个抽象方法`abstract void lock()`, 它的具体实现由子类来完成；
+ 然后定义了两个内部类`NonfairSync`和`FairSync`分别代表公平锁和非公平锁，它们都继承了Sync，分别实现了各自的lock()方法;公平与非公平也主要体现在lock()的加锁策略上面。

~~~java
static final class NonfairSync extends Sync {

    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }
}

static final class FairSync extends Sync {
    final void lock() {
            acquire(1);
        }
}
~~~
+ NonfairSync.lock()
    + 非公平锁的lock刚入方法就利用CAS方法对state进行更新，如果设置成功，则加锁成功；如果设置不成功，则进入公平锁模式；
+ FairSync.lock()
    + 公平锁的lock调用了AQS的方法`acquire`，它利用了模板模式，固化了加锁策略，也就是AQS的入队等待策略；
> 具体AQS的分析，可以单独查看AQS文章。

下面我们来主要分析公平锁中的lock()方法
> lock.lock()

~~~java
final void lock() {
    acquire(1);
}

public final void acquire(int arg) {
    // tryAcquire尝试获取锁，如果没有获取到锁，则表示有资源竞争
    if (!tryAcquire(arg) &&
        // 如果没有获取到锁，则将该线程加入到等待队列中
        // 1.首先生成线程Node，并加入到队列中
        // 2.再次尝试获取锁，没有获取到锁，则执行park，阻塞队列
            // 直到线程获取到资源，返回线程的终端标志位，如果有被中断过，则当前线程补上自己中断操作，
            // 因为如果线程在等待的过程中如果被中断过，它是响应不了的。只有获取到锁资源后再进行自我中断
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        // 自己执行中断操作
        selfInterrupt();
}
~~~
> acquire是AQS的加锁的资源竞争的模板方法
    
+ !tryAcquire(arg)，尝试获取锁，注意这里是取非(`!`)
    + 如果获取成功，则直接返回(!tryAcquire(arg)=false)
    + 如果获取不成功，则调用acquireQueued，进行入队，但是它也不是简单的入队，请看上面的注释，acquireQueued内部代码是一个for的死循环，只有在获取到锁的时候，它的循环才会结束
        + 如果入队成功，当前线程则被park，处于阻塞状态，知道被unpark，继续执行内部for循环，知道获取到锁，返回中断标志
        + 如果入队不成功，说明该方法已经获取到锁，返回中断标志
    + 最后如果获取到的中断标志为false，则拿到锁退出；如果拿到了中断标志，那么执行自行中断；

> 这里我们引出ReentrantLock的三个特性；
+ CAS
    + AQS的线程锁状态值state，已经内部链表队列节点的关联操作，都是通过CAS特性进行的原子性操作，保证了资源竞争中的安全问题
+ 队列
    + 对于无法获取锁的线程，全部加入到队列中进行排队
+ 自旋 + LorkSupport.park()
    + AQS中就是基于自旋实现多线程资源竞争的并发框架；内部有多个for死循环结构，再通过LorkSupport.park()来阻塞线程，减少cpu的消耗
    + 它与自旋一起使用，调用park阻塞for循环，当排队到了第一位时，调用unpark对线程进行唤醒

作者DougLea在`The java.util.concurrent Synchronizer FrameWork`中也又说明

~~~markdown
The basic ideas behind a synchronizer are quite straightforward. An acquire operation proceeds as:

while (synchronization state does not allow acquire) {
    enqueue current thread if not already queued;
possibly block current thread; 
}

dequeue current thread if it was queued; 
And a release operation is:

    update synchronization state;
    if (state may permit a blocked thread to acquire)
        unblock one or more queued threads;

Support for these operations requires the coordination of three basic components:

Atomically managing synchronization state(CAS原子管理同步状态)
Blocking and unblocking threads(线程阻塞/非阻塞，park/unpark)
Maintainping queues(维护队列)
~~~
> 整个AQS的设计也是围绕着这三点来实现，后面围绕这篇文章单独分析AQS。

> 下面在该文章重点分两部分，分别重点分析

+ !tryAcquire(arg)
+ acquireQueued(addWaiter(Node.EXCLUSIVE), arg))

#### !tryAcquire(arg)
~~~java

protected final boolean tryAcquire(int acquires) {
    // 获取当前线程
    final Thread current = Thread.currentThread();
    // 获取state的值
    // 1.t1进入，state=0
    // 2.t1已结束，t2进入，state=0
    // 3.t1未结束，t2进入，state=1
    int c = getState();
    // c=0表示没有被持有，处于自由状态
    if (c == 0) {
        // 判断是否有等待队列，且当前线程是否排在队列的第一位，hasQueuedPredecessors源码在下面
        // 一句话，判断自己是否需要排队
        // 此处是公平锁与非公平锁的唯一区别
        if (!hasQueuedPredecessors() &&
            // 不需要排队，则进行CAS操作
            compareAndSetState(0, acquires)) {
            // 设置当前持有锁的线程
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 当t1未结束，t2进入时，state=1,判断当前线程是否为当前持有线程，如果是，则为重入锁
    else if (current == getExclusiveOwnerThread()) {
        // 重入锁，state=1
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    // 当t1未结束，t2进入，有锁竞争时，如果不是重入锁，则返回false
    return false;
}
~~~
+ 首先获取锁的state值
    + 当值为0值，表示当前锁为自由状态
        + 先检查是否有在排队等待的线程
            + 如果AQS队列中没有排队的节点，则可以进行加锁操作
            + 如果有排队中的节点，则返回false，有人排队，那么你也需要排队，所以获取锁失败，这里就是公平与非公平的区别所在。
    + 当值>0时，表示当前锁已经被持有
        + 如果持有该锁的当前线程，则说明当前线程想最次获取锁，这里就是重入锁的体现，当前线程再次获取锁，直接将state+1即可
        + 如果不是当前线程，则直接返回false，因为当前锁还没有释放，或者获取锁失败
> 这里需要说明的一点，就是当state=0时，不能直接去进行加锁操作，因为这个是公平锁，所谓公平，就是FIFO先到先出；否则就是非公平锁，非公平锁的实现逻辑就是直接加锁，可以看NofairSync的lock方法。


~~~java
// 进入此函数的前提判断是state=0，也就是说，竞争的资源已经处于自由状态，处于等待被持有的状态
// 此时，我们需要分析下一个线程在整个等待队列中的状态
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    //1. t1进入，h = null, t = null, 那么h!=t为false, 则该方法直接return false结束，表示不需要排队。
    //2. 当t1执行完毕后，t2进入，t2的过程与t1是一样的，它们属于交替执行，不存在资源的竞争，则此多线程交易执行的场景下，在JDK级别就可以完全解决多线程的并发问题
    //第一种情况和第二种情况都是处理队列在没有初始化的状态下进行的，因为如果多线程在没有资源竞争的情况下，也就是交替执行的情况下，无需竞争锁，不需要任何其他的加锁操作，此情况下的多线程消耗非常小，可以忽略不记
    //那么在线程队列已经被初始化的情况下，又有两种情况
    //3. 队列已经初始化，该段逻辑较为复杂，总体思维就是，当队列有排队的节点，则表示需要进行排队，当排在第一个位置的是自己时，不需要进行排队，这一段代码技术非常高，要分为很多中情况，具体分析，这里的分析只供参考。


    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
~~~

> 下面的代码就是分析队列的状态，请查看注释，这里的代码量非常少，但是所涵盖的内容非常多，总体上是分两种情况
+ 当队列没有初始化,则head=tail=null, h!=t --> false，返回false，表示当前队列中没有排队的线程
+ 队列已经完成初始化
    + 当队列中的节点>1，h!=t --> true, (s = h.next) == null--> false
        + 当第一个节点是自己, s.thread != Thread.currentThread()-->false, 所以true && (false || false)，整体返回false
        + 当第一个节点不是自己, s.thread != Thread.currentThread()-->true, 所以true && (false || true)，整体返回true
    + 当队列中的节点=1, head=tail=node,那么h!=t--> false, 整体直接返回false
> 这段逻辑特别不容易理解，特别是最后一种情况，当对象只是最后一个排队者时，需要清楚，head和tail分别指向的是什么，因为我们必须要了解，队列在整个加锁和解锁的过程中，他们内部节点是如何变化的，也就是head和tail的关联操作时什么实现的。这里建议画一张双向链表图，然后根据代码一步一步的分析。主要位置就是release方法，调用了unpark方式，并且

#### acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
~~~java

private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // 初始化前驱节点为队列中的最后一个节点
    Node pred = tail;
    if (pred != null) {//如果有前驱节点，则直接设置当前线程节点的前驱节点
        node.prev = pred;
        // 对同步队列中的tail变量进行CAS赋值，如果cas复制成功，则将该节点加入到队尾
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 如果队列中不存在tail，说明队列没有初始化
    enq(node);
    // 返回入队的线程节点
    return node;
}


private Node enq(final Node node) {
    // 该处为一个死循环，主要是第一次初始化t=null执行一次队列初始化
    // 初始化后执行第二段逻辑
    for (;;) {
        Node t = tail;
        if (t == null) { // 队列初始化
            // new Node()实际生成了一个空节点，并将head进行CAS赋值为该node，即为空节点
            if (compareAndSetHead(new Node()))
                // 将tail和head执行同一个空的node节点，然后执行第二遍for循环逻辑
                tail = head;
        } else {
            // 将传进来的线程节点的前驱节点指向tail
            node.prev = t;
            // 并设置队列的队尾为该线程节点
            if (compareAndSetTail(t, node)) {
                // t.next中的t实际上是head，因为第一遍的for循环逻辑中tail=head，而t=tail，所以t=head,故t.next实际上是head.next指向了当前节点
                t.next = node;
                return t;
            }
        }
    }
}



/**
* Acquires in exclusive uninterruptible mode for thread already in
* queue. Used by condition wait methods as well as acquire.
*
* @param node the node
* @param arg the acquire argument
* @return {@code true} if interrupted while waiting
*/
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        // 此处同样是一个死循环
        for (;;) {
            // 首先获取该线程节点的前驱节点
            final Node p = node.predecessor();
            //如果前驱节点是head, 说明当前节点排在队首，则try获取锁
            if (p == head && tryAcquire(arg)) {
                // 如果获取锁成功，则当队列的头部设置为当前节点，
                setHead(node);
                // 并且将头节点的后续节点设置为空
                p.next = null; // help GC
                failed = false;
                return interrupted;
                // 此处实际上是线程在加入队列的过程中，当它发现它的前驱节点为head时，则再次尝试获取锁
                // 因为head中的线程节点是已经获取到锁的正在运行中的线程，由于CPU运行很快，所以此操作有很大的成功机会，这样就避免了入队
            }
            // 如果没有获取到锁，则需要执行park操作，对该线程进行阻塞，此时分为两种情况
            // 1.该线程的前驱节点为head
            // 2.该线程的前驱节点不为head
            // 可看下面对shouldParkAfterFailedAcquire函数的注释
            if (shouldParkAfterFailedAcquire(p, node) &&
                // 对当前线程执行park操作，在unpark之后检查中断标志，判断线程有没有被中断过，如果被中断过，则返回中断标志，当它获取到锁资源后，在上一个if逻辑中，return最终的终端标志位
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}


private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    // node中有一个线程的等待状态值
    // Node.SIGNAL=-1,表示线程是阻塞状态
    // Node.CANNEL=1，表示线程是取消状态
    // Node.CONDITION=-1,表示线程处于CONDITION的阻塞状态
    // Node.PROPAGATE  ????
    
    // 首先判断前端节点的线程状态值，如果它处于阻塞状态，则直接放回true，表示node也应该被park，因为它前面的排队的都处于阻塞状态，
    // 那么它自己肯定也需要被阻塞，不然就不叫排队了
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        /*
            * This node has already set status asking a release
            * to signal it, so it can safely park.
            */
        return true;
    // 如果它的前驱节点线程处于被取消的状态，则不断的去遍历处于正常状态下的前驱节点，直到找到一个不处于取消状态下的线程节点为止
    if (ws > 0) {
        /*
            * Predecessor was cancelled. Skip over predecessors and
            * indicate retry.
            */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
        // 此时它找到了一个线程状态不大于0的线程节点，而后该函数返回false，而外部for死循环会再次执行代码逻辑
        // 当再次跳入此方法时，前端节点的状态值如果时park状态，则执行上面的第一个if逻辑，如果是cancel状态，
        // 则再次执行此段逻辑，直到park为止
    } else {
        /*
            * waitStatus must be 0 or PROPAGATE.  Indicate that we
            * need a signal, but don't park yet.  Caller will need to
            * retry to make sure it cannot acquire before parking.
            */
            // 线程节点的waitstatus的初始值为0，那么直接设置前驱节点为park状态
            // 此处有一点让人感觉奇怪的地方是，当前线程居然对上一个排队的线程设置为park状态，为什么不是前驱节点自己设置自己的线程状态呢？
            // 其实此处想想也容易理解，因为线程如果执行park操作，那么线程已经停止了运行，因为它已经处于阻塞状态，让出了CPU，这时，它自己
            // 已经没有办法设置自己的状态值了，必须通过其他线程来设置它的状态值。
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}


private final boolean parkAndCheckInterrupt() {
    // 对当前线程执行park操作，
    LockSupport.park(this);
    //----执行到这个位置当前线程已经阻塞了，想要执行下面的代码，必须执行unpark----------

    // 当执行了unpark，则执行终端复位操作
    return Thread.interrupted();
}
~~~




