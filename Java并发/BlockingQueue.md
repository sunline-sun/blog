### ArrayBlockingQueue
- ArrayBlockingQueue是BlockingQueue的有界队列实现类，底层是数组实现的
- ArrayBlockingQueue是通过一个ReentranLock和两个condition实现的，其中有几个属性：包含元素的数组、下一个读取的操作的位置、下一个写入操作的位置、队列中的元素数量

#### ArrayBlockingQueue实现原理
- 读操作和写操作都要获取到锁之后才可以进行操作
- 如果队列为空，读操作会进入读线程队列中排队，等待写线程写入新的元素，唤醒读线程队列的第一个等待线程
- 如果队列已满，写操作会进入写线程队列排队，等待读线程移除元素，唤醒写线程操作的第一个等待线程

#### ArrayBlockingQueue可以指定的参数
- 队列容量，指定了队列最多允许的元素个数
- 指定公平锁还是非公平锁
- 指定一个初始集合放入队列

### LinkedBlockingQueue
- 基于单向链表实现的阻塞队列，可以当做有界也可以当做无界队列
- LinkedBlockingQueue是通过两个ReenTrantLock和两个condition实现的，其中有几个属性：队列容量、队列中元素数量、对头、队尾，其中head是空节点

#### LinkedBlockingQueue实现原理
- 如果要获取一个元素，首先要获取读锁，如果队列为空，插入读队列
- 如果插入一个元素，首先获取到写锁，如果队列满了，插入写队列

#### LinkedBlockingQueue是怎么保证线程安全的，为什么读写一起进行不会导致数据的读取写入错误
- 首先count变量是通过原子操作进行的，可以保证count的大小正确
- 读写操作会获取各自的锁，他们内部是同步的，而且读操作是对队列中的head节点操作，读取之后head变为head.next，而写操作是对tail节点操作的，写完之后是吧tail变为node

#### LinkedBlokingQueue put()实现步骤
- 首先获取写锁
- 判断队列是否满了，满了就调用await()方法插入条件队列等待
- 队列没满的话直接入队，然后count原子性加1，然后判断是否还有空槽，有的话唤醒写队列
- 释放锁
- 判断插入之前是否队列是空的，是的话唤醒读队列

<details>
  <summary>源代码</summary>
  
  ```java 
  public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    // 如果你纠结这里为什么是 -1，可以看看 offer 方法。这就是个标识成功、失败的标志而已。
    int c = -1;
    Node<E> node = new Node(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    // 必须要获取到 putLock 才可以进行插入操作
    putLock.lockInterruptibly();
    try {
        // 如果队列满，等待 notFull 的条件满足。
        while (count.get() == capacity) {
            notFull.await();
        }
        // 入队
        enqueue(node);
        // count 原子加 1，c 还是加 1 前的值
        c = count.getAndIncrement();
        // 如果这个元素入队后，还有至少一个槽可以使用，调用 notFull.signal() 唤醒等待线程。
        // 哪些线程会等待在 notFull 这个 Condition 上呢？
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        // 入队后，释放掉 putLock
        putLock.unlock();
    }
    // 如果 c == 0，那么代表队列在这个元素入队前是空的（不包括head空节点），
    // 那么所有的读线程都在等待 notEmpty 这个条件，等待唤醒，这里做一次唤醒操作
    if (c == 0)
        signalNotEmpty();
}

// 入队的代码非常简单，就是将 last 属性指向这个新元素，并且让原队尾的 next 指向这个元素
// 这里入队没有并发问题，因为只有获取到 putLock 独占锁以后，才可以进行此操作
private void enqueue(Node<E> node) {
    // assert putLock.isHeldByCurrentThread();
    // assert last.next == null;
    last = last.next = node;
}

// 元素入队后，如果需要，调用这个方法唤醒读线程来读
private void signalNotEmpty() {
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
}
  ```
  
  </details>

#### LinkedBlockingQueue take()实现步骤
- 首先获取读锁
- 判断队列是否为空，空了的话调用await方法插入条件队列等待
- 队列不为空的话取出head的下个节点，然后count原子性减1，然后判断是否还有数据，有的话唤醒读队列
- 释放锁
- 判断取出元素之前是否是满的，是的话唤醒写队列


<details>
  <summary>源代码</summary>
  
  ```java 
  public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    // 首先，需要获取到 takeLock 才能进行出队操作
    takeLock.lockInterruptibly();
    try {
        // 如果队列为空，等待 notEmpty 这个条件满足再继续执行
        while (count.get() == 0) {
            notEmpty.await();
        }
        // 出队
        x = dequeue();
        // count 进行原子减 1
        c = count.getAndDecrement();
        // 如果这次出队后，队列中至少还有一个元素，那么调用 notEmpty.signal() 唤醒其他的读线程
        if (c > 1)
            notEmpty.signal();
    } finally {
        // 出队后释放掉 takeLock
        takeLock.unlock();
    }
    // 如果 c == capacity，那么说明在这个 take 方法发生的时候，队列是满的
    // 既然出队了一个，那么意味着队列不满了，唤醒写线程去写
    if (c == capacity)
        signalNotFull();
    return x;
}
// 取队头，出队
private E dequeue() {
    // assert takeLock.isHeldByCurrentThread();
    // assert head.item == null;
    // 之前说了，头结点是空的
    Node<E> h = head;
    Node<E> first = h.next;
    h.next = h; // help GC
    // 设置这个为新的头结点
    head = first;
    E x = first.item;
    first.item = null;
    return x;
}
// 元素出队后，如果需要，调用这个方法唤醒写线程来写
private void signalNotFull() {
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        notFull.signal();
    } finally {
        putLock.unlock();
    }
}
  ```
  
  </details>


### SynchronousQueue
- 一个同步队列，在线程池中有使用到，创建的时候可以设置公平锁还是非公平锁（对应的是transfer方法还是transferStack方法）,等待队列是单向链表，用的QNode节点，包含几个属性：next、对象数据、属于哪个线程、当前是读节点还是写节点
- 底层是通过transfer方式实现的，就是如果来了一个线程，发现队列里是空的或者和自己是同一种请求的（来的是写线程，队列里也是写线程），就插入队列，如果不是同一种请求，就取走队列头结点的数据。

#### SynchronousQueue put和get方法
- 都是调用了transfer方法，只是传参不一样

<details>
  <summary> 源代码</summary>
  
  ```java
  // 写入值
public void put(E o) throws InterruptedException {
    if (o == null) throw new NullPointerException();
    if (transferer.transfer(o, false, 0) == null) { // 1
        Thread.interrupted();
        throw new InterruptedException();
    }
}
// 读取值并移除
public E take() throws InterruptedException {
    Object e = transferer.transfer(null, false, 0); // 2
    if (e != null)
        return (E)e;
    Thread.interrupted();
    throw new InterruptedException();
}
  ```
  </details>

#### SynchronousQueue transfer()方法实现步骤
- 判断队列为空或者是和自己相同类型的节点
  - 如果是的话，插入队尾，设置尾结点
    - 等待被消费，这时候有两种，会先计算自旋的次数
      - 在自旋次数不到阈值之前，或者剩余时间小于阈值的时候会通过自旋等待被消费
      - 自旋次数到达阈值后，会调用park()方法把线程挂起，等待消费之后唤醒本身线程
    - 被唤醒之后，说明自身被消费了，把自身清理出去就好了
  - 如果不是的话，说明有配对的，拿到队列头结点的数据，然后唤醒头节点的线程，返回数据


<details>
  <summary> 源代码</summary>
  
  ```java
  /**
 * Puts or takes an item.
 */
Object transfer(Object e, boolean timed, long nanos) {

    QNode s = null; // constructed/reused as needed
    boolean isData = (e != null);

    for (;;) {
        QNode t = tail;
        QNode h = head;
        if (t == null || h == null)         // saw uninitialized value
            continue;                       // spin

        // 队列空，或队列中节点类型和当前节点一致，
        // 即我们说的第一种情况，将节点入队即可。读者要想着这块 if 里面方法其实就是入队
        if (h == t || t.isData == isData) { // empty or same-mode
            QNode tn = t.next;
            // t != tail 说明刚刚有节点入队，continue 即可
            if (t != tail)                  // inconsistent read
                continue;
            // 有其他节点入队，但是 tail 还是指向原来的，此时设置 tail 即可
            if (tn != null) {               // lagging tail
                // 这个方法就是：如果 tail 此时为 t 的话，设置为 tn
                advanceTail(t, tn);
                continue;
            }
            // 
            if (timed && nanos <= 0)        // can't wait
                return null;
            if (s == null)
                s = new QNode(e, isData);
            // 将当前节点，插入到 tail 的后面
            if (!t.casNext(null, s))        // failed to link in
                continue;

            // 将当前节点设置为新的 tail
            advanceTail(t, s);              // swing tail and wait
            // 看到这里，请读者先往下滑到这个方法，看完了以后再回来这里，思路也就不会断了
            Object x = awaitFulfill(s, e, timed, nanos);
            // 到这里，说明之前入队的线程被唤醒了，准备往下执行
            if (x == s) {                   // wait was cancelled
                clean(t, s);
                return null;
            }

            if (!s.isOffList()) {           // not already unlinked
                advanceHead(t, s);          // unlink if head
                if (x != null)              // and forget fields
                    s.item = s;
                s.waiter = null;
            }
            return (x != null) ? x : e;

        // 这里的 else 分支就是上面说的第二种情况，有相应的读或写相匹配的情况
        } else {                            // complementary-mode
            QNode m = h.next;               // node to fulfill
            if (t != tail || m == null || h != head)
                continue;                   // inconsistent read

            Object x = m.item;
            if (isData == (x != null) ||    // m already fulfilled
                x == m ||                   // m cancelled
                !m.casItem(x, e)) {         // lost CAS
                advanceHead(h, m);          // dequeue and retry
                continue;
            }

            advanceHead(h, m);              // successfully fulfilled
            LockSupport.unpark(m.waiter);
            return (x != null) ? x : e;
        }
    }
}

void advanceTail(QNode t, QNode nt) {
    if (tail == t)
        UNSAFE.compareAndSwapObject(this, tailOffset, t, nt);
}

// 自旋或阻塞，直到满足条件，这个方法返回
Object awaitFulfill(QNode s, Object e, boolean timed, long nanos) {

    long lastTime = timed ? System.nanoTime() : 0;
    Thread w = Thread.currentThread();
    // 判断需要自旋的次数，
    int spins = ((head.next == s) ?
                 (timed ? maxTimedSpins : maxUntimedSpins) : 0);
    for (;;) {
        // 如果被中断了，那么取消这个节点
        if (w.isInterrupted())
            // 就是将当前节点 s 中的 item 属性设置为 this
            s.tryCancel(e);
        Object x = s.item;
        // 这里是这个方法的唯一的出口
        if (x != e)
            return x;
        // 如果需要，检测是否超时
        if (timed) {
            long now = System.nanoTime();
            nanos -= now - lastTime;
            lastTime = now;
            if (nanos <= 0) {
                s.tryCancel(e);
                continue;
            }
        }
        if (spins > 0)
            --spins;
        // 如果自旋达到了最大的次数，那么检测
        else if (s.waiter == null)
            s.waiter = w;
        // 如果自旋到了最大的次数，那么线程挂起，等待唤醒
        else if (!timed)
            LockSupport.park(this);
        // spinForTimeoutThreshold 这个之前讲 AQS 的时候其实也说过，剩余时间小于这个阈值的时候，就
        // 不要进行挂起了，自旋的性能会比较好
        else if (nanos > spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanos);
    }
}
  ```
  </details>
  
  #### SynchronousQueue transferStack()方法实现步
- 判断队列是空的，或者队列中的节点和当前的线程操作类型一致，这种情况下，将当前线程加入到等待栈中，等待配对。然后返回相应的元素，或者如果被取消了的话，返回 null。
- 如果栈中有等待节点，而且与当前操作可以匹配。将当前节点压入栈顶，和栈中的节点进行匹配，然后将这两个节点出栈。配对和出栈的动作其实也不是必须的，因为下面的一条会执行同样的事情。
- 如果栈顶是进行匹配而入栈的节点，帮助其进行匹配并出栈，然后再继续操作。


### PriorityBlockingQueue
- 是一个带排序功能的BlockingQueue，是一个无界队列，就算创建的时候指定了队列长度（默认初始长度11），后面也会自动扩容。就是PriorityQueue的线程安全版本，线程安全也是通过ReentranLock和condition实现的，底层结构是基于数组的二叉堆，
- 二叉堆：一颗完全二叉树，它非常适合用数组进行存储，对于数组中的元素 a[i]，其左子节点为 a[2*i+1]，其右子节点为 a[2*i + 2]，其父节点为 a[(i-1)/2]，其堆序性质为，每个节点的值都小于其左右子节点的值。二叉堆中最小的值就是根节点，但是删除根节点是比较麻烦的，因为需要调整树。
  
  <details>
  <summary> 源代码</summary>
  
  ```java
  
  ```
  </details>
  
    <details>
  <summary> 源代码</summary>
  
  ```java
  
  ```
  </details>
  
    <details>
  <summary> 源代码</summary>
  
  ```java
  
  ```
  </details>
  
    <details>
  <summary> 源代码</summary>
  
  ```java
  
  ```
  </details>

### 各个队列的区别和使用场景
