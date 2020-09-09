

ThreadLocal<Integer> threadLocal = new ThreadLocal<>();
threadLocal.set(1);


~~~java
public void set(T value) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 获取当前线程的ThreadLocalMap,再Thread中定义了一个成员变量
    // ThreadLocal.ThreadLocalMap threadLocals = null;
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

// getMap内部是返回thread的一个成员变量ThreadLocal.ThreadLocalMap，这句话的意思也就是获取当前线程的一个ThreadLocalMap，那么ThreadLocalMap是个什么？
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
~~~

# ThreadLocalMap是什么东东?
~~~java
static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    private void set(ThreadLocal<?> key, Object value) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
}
~~~

+ 首先从名称来看，它是一个Map，自然要遵循map的特点；
+ Entry继承了一个弱引用，所以也具备弱引用的特点，弱引用就是程序在接受到GC操作时，会立即回收内存。那么这是为什么呢？为什么要用弱引用呢？

## 强软弱虚
+ 强引用
    + 直接引用，当对象被直接引用称为强引用，称为强关联。当这种关联一直存在时，它不会被GC回收
+ 软引用
    + 当一个对象被声明为软引用，那么在JVM内存不足时，它会被回收
+ 弱引用
    + 当一个对象被声明为弱引用，只要遇到GC，它就会被回收
+ 虚引用
    + 它必须与队列联合使用。它的特点，主要是当对象被回收时，需要将回收信息保存到队列中，起到通知作用。
    + 它主要用于管理堆外内存（DirectByteBuffer）。

### JVM & 堆外内存 & 虚引用
> 想象一下，当有一大段视频数据通过网卡传输我们的内存，如果是JVM来处理，JVM管理内存是通过堆来管理内存的，这是需要将传过来的数据复制到JVM堆内存中，只有这样，JVM才能进行堆内存的管理，但是这种数据复制过程是需要消耗cpu的，那么如何避免这种复制呢。JVM引入的堆外内存的概念，也就是说，去掉了数据复制过程，直接管理堆外内存，那么如何来管理这段内存呢？简单点说，如果这段数据已经没有其它关联引用了，如何回收呢？JVM这时是无法去回收这段内存的？
> 这是引入了虚引用，当引用这段内存的对象被回收时，需要通知JVM，这段内存被回收了，这时JVM才能去处理这段内存。
> 虚引用就是对象与队列的关联使用，当对象被回收时，将回收信息保存到队列中，JVM监听到队列通知，就会去处理堆外内存。

