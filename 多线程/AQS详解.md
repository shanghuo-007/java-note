# AQS详解

## AbstractQueuedSynchronizer简介

AQS是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如`ReentrantLock`，`Semaphore`，其他的诸如`ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask`等等皆是基于AQS的。当然，我们自己也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器。

### AQS 核心思想

AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。==在双向链表的结构下，这样更容易实现取消和超时功能==。

> `CLH(Craig,Landin,and Hagersten)`队列是一个虚拟的双向队列(虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配。

AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

```java
private volatile int state;//共享变量，使用volatile修饰保证线程可见性
```

状态信息通过procted类型的getState，setState，compareAndSetState进行操作

```java
//返回同步状态的当前值
protected final int getState() {  
        return state;
}
 // 设置同步状态的值
protected final void setState(int newState) { 
        state = newState;
}
//原子地(CAS操作)将同步状态值设置为给定值update如果当前同步状态的值等于expect(期望值)
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

### AQS 对资源的共享方式

AQS定义两种资源共享方式

- Exclusive(独占)：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁： 
  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- Share(共享)：多个线程可同时执行，如Semaphore/CountDownLatch。Semaphore、CountDownLatCh、 CyclicBarrier、ReadWriteLock 

ReentrantReadWriteLock 可以看成是组合式，因为ReentrantReadWriteLock也就是读写锁允许多个线程同时对某一资源进行读。

### AQS 提供的一系列模板方法

查看 AQS 的源码我们就可以发现这个类提供了很多方法，看起来让人“眼花缭乱”的。但是最主要的两类方法就是获取资源的方法和释放资源的方法。因此我们抓住主要矛盾就行了：

- public final void acquire(int arg) // 独占模式的获取资源
- public final boolean release(int arg) // 独占模式的释放资源
- public final void acquireShared(int arg) // 共享模式的获取资源
- public final boolean releaseShared(int arg) // 共享模式的释放资源

#### acquire方法

该方法以独占方式获取资源，如果获取到资源，线程继续往下执行，否则进入等待队列，直到获取到资源为止，且整个过程忽略中断的影响。该方法是独占模式下线程获取共享资源的顶层入口。

```scss
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

下面分析下这个`acquire`方法的具体执行流程：

> step1：首先这个方法调用了用户自己实现的方法`tryAcquire`方法尝试获取资源，如果这个方法返回true，也就是表示获取资源成功，那么整个`acquire`方法就执行结束了，线程继续往下执行；
>
> step2：如果`tryAcquir`方法返回false，也就表示尝试获取资源失败。这时`acquire`方法会先调用`addWaiter`方法将当前线程封装成Node类并加入一个FIFO的双向队列的尾部。
>
> step3：再看`acquireQueued`这个**关键方法**。首先要注意的是这个方法中哪个无条件的for循环，这个for循环说明`acquireQueued`方法一直在自旋尝试获取资源。进入for循环后，首先判断了当前节点的前继节点是不是头节点，如果是的话就再次尝试获取资源，获取资源成功的话就直接返回false（表示未被中断过）
>
> 假如还是没有获取资源成功，判断是否需要让当前节点进入`waiting`状态，经过 `shouldParkAfterFailedAcquire`这个方法判断，如果需要让线程进入`waiting`状态的话，就调用LockSupport的park方法让线程进入`waiting`状态。进入`waiting`状态后，这线程等待被`interupt`或者`unpark`（在release操作中会进行这样的操作，可以参见后面的代码）。这个线程被唤醒后继续执行for循环来尝试获取资源。

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                //首先判断了当前节点的前继节点是不是头节点，如果是的话就再次尝试获取资源，
                //获取资源成功的话就直接返回false（表示未被中断过）
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                //判断是否需要让当前节点进入waiting状态
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    // 如果在整个等待过程中被中断过，则返回true，否则返回false。
                    // 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

以上就是`acquire`方法的简单分析。

单独看这个方法的话可能会不太清晰，结合`ReentrantLock`、`ReentrantReadWriteLock`、`CountDownLatch`、`Semaphore`和`LimitLatch`等同步工具看这个代码的话就会好理解很多。

#### release方法

`release(int)`方法是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

//上面已经讲过了，需要用户自定义实现
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}

private void unparkSuccessor(Node node) {
    /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

与acquire()方法中的tryAcquire()类似，tryRelease()方法也是需要独占模式的自定义同步器去实现的。正常来说，tryRelease()都会成功的，因为这是独占模式，该线程来释放资源，那么它肯定已经拿到独占资源了，直接减掉相应量的资源即可(state-=arg)，也不需要考虑线程安全的问题。

但要注意它的返回值，上面已经提到了，release()是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了！所以自义定同步器在实现时，如果已经彻底释放资源(state=0)，要返回true，否则返回false。

`unparkSuccessor(Node)`方法用于唤醒等待队列中下一个线程。**这里要注意的是，下一个线程并不一定是当前节点的next节点，而是下一个可以用来唤醒的线程，如果这个节点存在，调用`unpark()`方法唤醒**。

总之，release()是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。（**需要注意的是队列中被唤醒的线程不一定能立马获取资源，因为资源在释放后可能立马被其他线程（不是在队列中等待的线程）抢掉了**）

#### acquireShared方法

`acquireShared(int)`方法是共享模式下线程获取共享资源的顶层入口。它会获取指定量的资源，获取成功则直接返回，获取失败则进入等待队列，直到获取到资源为止，整个过程忽略中断。

```java
public final void acquireShared(int arg) {
    //tryAcquireShared需要用户自定义实现
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

可以发现，这个方法的关键实现其实是获取资源失败后，怎么管理线程。也就是`doAcquireShared`的逻辑。

```java
//不响应中断
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

可以看出，`doAcquireShared`的逻辑和`acquireQueued`的逻辑差不多。将当前线程加入等待队列尾部休息，直到其他线程释放资源唤醒自己，自己成功拿到相应量的资源后才返回。

简单总结下`acquireShared`的流程：

> step1：tryAcquireShared()尝试获取资源，成功则直接返回；
>
> step2：失败则通过doAcquireShared()进入等待队列park()，直到被unpark()/interrupt()并成功获取到资源才返回。整个等待过程也是忽略中断的。

#### releaseShared方法

`releaseShared(int)`方法是共享模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果成功释放且允许唤醒等待线程，它会唤醒等待队列里的其他线程来获取资源。

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

释放掉资源后，唤醒后继。跟独占模式下的release()相似，但有一点稍微需要注意：独占模式下的tryRelease()在完全释放掉资源（state=0）后，才会返回true去唤醒其他线程，这主要是基于独占下可重入的考量；而共享模式下的releaseShared()则没有这种要求，共享模式实质就是控制一定量的线程并发执行，那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待结点。