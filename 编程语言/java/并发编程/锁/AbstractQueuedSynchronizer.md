# AbstractQueuedSynchronizer

>作者Doug Lea写了一篇论文`The java.util.concurrent Synchronizer Framework`，重点在第三章`Design and implementation`中提出了并发组件三个基本组成部分，原文如下

The basic ideas behind a synchronizer are quite straightforward. An acquire operation proceeds as:
~~~java
    while (synchronization state does not allow acquire) {
        enqueue current thread if not already queued;
        possibly block current thread; 
    }
~~~
dequeue current thread if it was queued; And a release operation is:
~~~java
    update synchronization state;
    if (state may permit a blocked thread to acquire)
        unblock one or more queued threads; 
~~~
Support for these operations requires the coordination of three basic components: 
• Atomically managing synchronization state 
• Blocking and unblocking threads 
• Maintaining queues


伪码分析：
+ 当同步状态量不允许获取时，也就是源码中的state>0，那么将线程入队，并调用park阻塞该线程，这里是一个死循环
+ 当同步状态量处于自由状态了，允许被阻塞中的线程加锁了，那么调用unpark方法唤醒阻塞中的线程，也就是unblock thread
> 要想支持这两步操作，需要三个基本组件
我们主要来分析这三点
+ Atomically manaing synchronization state(原子管理同步状态-cas)
    + 同步器的状态量必须是原子性操作，已保证竞争状态下的锁安全问题
+ Blocking and unblocking threads(阻塞和解除阻塞线程-park)
    + 以自旋的方式监听线程状态，利用park来阻塞线程，同时当满足加锁操作时，利用unpark对线程进行唤醒
+ Maintaining queues(维护队列)
    + 线程的竞争是通过队列来进行维护的，可以满足FIFO特性(不支持基于优先级的同步)，随着同步状态state的动态变化，队列中的线程节点不断的进行入队出队，这个队列的维护工作至关重要，它同样也要利用CAS保证队列操作的原子性。

下面我们通过分析AbstractQueuedSynchronizer的源代码，来学习基于上述三点的实现


~~~java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {

    private volatile int state;
    protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }


    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

    protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }

    final boolean acquireQueued(final Node node, int arg){...}
}
~~~

## Atomically manaing synchronization state
> 如何保证同步器状态state的线程安全，实际上也就是我们在JMM模型中的三个问题，可见性/有序性/原子性。源码中state为volatile，volatile本身解决了变量的可见性和有序性，同时源码中state的set方法是CAS提供的是原子性操作，那么也就解决了state的线程安全问题。
+ private volatile int state;
+ protected final boolean compareAndSetState(int expect, int update){...}

## Blocking and unblocking threads
+ 对排队的线程进行LockSupport.park(),acquireQueued的实现入队成功，则立即park它自己
+ 对first in的满足条件的排队者进行unpark()操作，排队者的unpark操作是由它的前驱节点来调用的，也就是说，当某一个获取到锁的线程释放锁的时候，它会去读取队列的head节点，调用head中的thread线程的unpark，对其唤醒。具体实现可以查看release方法

## Maintaining queues
AQS提供了acquire模板方法，首先调用tryAcquire尝试获取锁，如果获取失败，则进行入队操作。而tryAcquire是一个protected方法，可以被子类去实现，具体如果获取锁可以被自定义，如重入锁中的公平锁的实现，它首先判断队列中是否又排队者，如果有则排队，没有则加锁；而非公平锁并没有判断是否有排队者，他可以直接去获取锁，如果没有拿到锁，再入队，走公平锁的逻辑。
我们通过分析重入锁的公平锁tryAcquire来具体分析它是怎么维护队列的

主要分析
~~~java
 public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
~~~


+ !tryAcquire(arg)，尝试加锁
+ acquireQueued(addWaiter(Node.EXCLUSIVE), arg)，入队


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
            // 其实此处想想也容易理解，因为线程如果执行park操作，那么线程已经停止了运行，因为它已经处于阻塞状态，让出了CPU，这时，它自己已经没有办法设置自己的状态值了，必须通过其他线程来设置它的状态值。那问题又来了？为什么不提前设置为-1，再调用park方法呢，这显然与大师的代码规范不符合，必须先park，再修改状态，而不是修改了状态，再park，万一park失败了呢。
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}


private final boolean parkAndCheckInterrupt() {
    // 对当前线程执行park操作，
    LockSupport.park(this);
    //----执行到这个位置当前线程已经阻塞了，想要执行下面的代码，必须执行unpark,而这个unpark的调用是持有锁的线程在释放锁的时候调用的，，可以自行查看重入锁的unlock源码----------

    // 当执行了unpark，则执行终端复位操作
    return Thread.interrupted();
}
~~~


