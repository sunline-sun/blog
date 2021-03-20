### AQS概念
- 用来构建锁和同步器的框架，ReentrantLock、ReentranReadWriteLock、SynchronousQueue、FutureTask都是基于AQS实现的

### AQS原理
- 核心思想就是：如果被请求的资源空闲，那么当前请求资源的线程就是工作线程，然后将资源标记为锁定状态，如果请求的资源是被锁定状态，就加入阻塞队列挂起等待唤醒。

### AQS两种资源共享方式
- 独占锁：只有一个线程能持有资源锁，ReenTrantLock就是基于独占锁实现的
- 共享锁：多个线程可以同时执行，比如Semaphore/CountDownLatch。Semaphore、CountDownLatCh、 CyclicBarrier、ReadWriteLock都是共享锁

### AQS底层用的设计模式是模板模式
- AQS同步器的设计是基于模板模式的，自定义的同步器需要继承AQS抽象类并重写他的部分方法（获取锁这些），然后使用的时候调用AQS的方法，AQS的方法里会使用自定义同步器中重写的方法

#### 模板模式
- 模板模式是基于继承实现的，就是在不改变模板结构的前提下重写部分方法。比如起床-坐车上班-干活，只有坐车上班需要自己实现方式，其他都是统一用父类的就可以

### acquire()方法
- 通常用作获取锁的操作，reentranLock就是通过这个方法加锁的

### ConditionObject内部类
- 维护了一个条件队列，是一个单向链表，节点也是Node节点，因为后面需要把节点转移到阻塞队列也就是AQS中，通常被用做实现生产者-消费者模式，ArrayBlockingQueue等就是依赖这个实现的

### Condition 大致处理流程
- 通过ReentranLock.newCondition()可以创建condition
- 线程1调用condition.await()方法可以将线程1包装成Node节点，插入条件队列，然后阻塞在这里
- 线程2调用condition.signal()触发一次唤醒，此时唤醒的是condition的头结点，会把头节点转移到阻塞队列队尾，等待获取锁，获取到锁之后await()方法返回

### Condition await()方法
- 先响应中断，如果线程中断状态，直接抛出异常
- 将线程包装成Node节点放入条件队列
  - 判断条件最后一个节点的状态，如果取消状态，就移除这个节点并遍历条件队列把所有取消节点移除
  - 将线程包装成Node节点
  - 把Node节点插入到队尾
- 完全释放独占锁（此时是完全释放，如果重入多次，也是一次性释放）
  - 调用Realease()释放锁，如果成功释放，返回释放前的状态
  - 如果释放失败，代表这个线程没有获取到锁，抛出异常，并把这个节点状态改为取消，等待后面清除出去
- 判断是否在阻塞队列，如果在（其他节点唤醒转移的），返回，如果没在，线程挂起，等待唤醒
  - 判断是否在阻塞队列：如果prev节点是null，说明肯定不在阻塞队列，因为条件队列没有prev，如果有next节点了，说明肯定在阻塞队列了，如果还判断不出来，就循环阻塞队列查找有没有相同节点
- 挂起线程，等待唤醒，有三种情况会唤醒
  - 转移到阻塞队列并获取到锁
  - 另一个线程会这个线程进行了中断
  - 转移到阻塞队列时候前一个节点取消状态或者CAS操作失败 
- 被唤醒之后，检查线程的中断状态，如果中断，是唤醒前中断还是唤醒后中断，这个判断方式是通过判断节点状态，因为唤醒过程会把状态从condition改为0
  - 如果唤醒前中断，将节点转移到阻塞队列中
  - 如果唤醒后中断，自旋判断是否在阻塞队列中的
- 此时节点已经进入了阻塞队列，开始获取独占锁，并把释放锁之前的state还原回来，这里也会设置中断状态（interruptMode）是唤醒前中断还是唤醒后中断
- 处理中断，如果中断状态是唤醒前也就是await期间中断的，那么抛出异常，否则设置线程中断状态为中断

<details>
  <summary>源代码</summary>
  
  ```java
  // 首先，这个方法是可被中断的，不可被中断的是另一个方法 awaitUninterruptibly()
// 这个方法会阻塞，直到调用 signal 方法（指 signal() 和 signalAll()，下同），或被中断
public final void await() throws InterruptedException {
    // 老规矩，既然该方法要响应中断，那么在最开始就判断中断状态
    if (Thread.interrupted())
        throw new InterruptedException();

    // 添加到 condition 的条件队列中
    Node node = addConditionWaiter();

    // 释放锁，返回值是释放锁之前的 state 值
    // await() 之前，当前线程是必须持有锁的，这里肯定要释放掉
    int savedState = fullyRelease(node);

    int interruptMode = 0;
    // 这里退出循环有两种情况，之后再仔细分析
    // 1. isOnSyncQueue(node) 返回 true，即当前 node 已经转移到阻塞队列了
    // 2. checkInterruptWhileWaiting(node) != 0 会到 break，然后退出循环，代表的是线程中断
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    // 被唤醒后，将进入阻塞队列，等待获取锁
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}

// 将当前线程对应的节点入队，插入队尾
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // 如果条件队列的最后一个节点取消了，将其清除出去
    // 为什么这里把 waitStatus 不等于 Node.CONDITION，就判定为该节点发生了取消排队？
    if (t != null && t.waitStatus != Node.CONDITION) {
        // 这个方法会遍历整个条件队列，然后会将已取消的所有节点清除出队列
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    // node 在初始化的时候，指定 waitStatus 为 Node.CONDITION
    Node node = new Node(Thread.currentThread(), Node.CONDITION);

    // t 此时是 lastWaiter，队尾
    // 如果队列为空
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}

// 等待队列是一个单向链表，遍历链表将已经取消等待的节点清除出去
// 纯属链表操作，很好理解，看不懂多看几遍就可以了
private void unlinkCancelledWaiters() {
    Node t = firstWaiter;
    Node trail = null;
    while (t != null) {
        Node next = t.nextWaiter;
        // 如果节点的状态不是 Node.CONDITION 的话，这个节点就是被取消的
        if (t.waitStatus != Node.CONDITION) {
            t.nextWaiter = null;
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;
            if (next == null)
                lastWaiter = trail;
        }
        else
            trail = t;
        t = next;
    }
}

// 首先，我们要先观察到返回值 savedState 代表 release 之前的 state 值
// 对于最简单的操作：先 lock.lock()，然后 condition1.await()。
//         那么 state 经过这个方法由 1 变为 0，锁释放，此方法返回 1
//         相应的，如果 lock 重入了 n 次，savedState == n
// 如果这个方法失败，会将节点设置为"取消"状态，并抛出异常 IllegalMonitorStateException
final int fullyRelease(Node node) {
    boolean failed = true;
    try {
        int savedState = getState();
        // 这里使用了当前的 state 作为 release 的参数，也就是完全释放掉锁，将 state 置为 0
        if (release(savedState)) {
            failed = false;
            return savedState;
        } else {
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}

int interruptMode = 0;
// 如果不在阻塞队列中，注意了，是阻塞队列
while (!isOnSyncQueue(node)) {
    // 线程挂起
    LockSupport.park(this);

    // 这里可以先不用看了，等看到它什么时候被 unpark 再说
    if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
        break;
}

// 在节点入条件队列的时候，初始化时设置了 waitStatus = Node.CONDITION
// 前面我提到，signal 的时候需要将节点从条件队列移到阻塞队列，
// 这个方法就是判断 node 是否已经移动到阻塞队列了
final boolean isOnSyncQueue(Node node) {

    // 移动过去的时候，node 的 waitStatus 会置为 0，这个之后在说 signal 方法的时候会说到
    // 如果 waitStatus 还是 Node.CONDITION，也就是 -2，那肯定就是还在条件队列中
    // 如果 node 的前驱 prev 指向还是 null，说明肯定没有在 阻塞队列(prev是阻塞队列链表中使用的)
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;
    // 如果 node 已经有后继节点 next 的时候，那肯定是在阻塞队列了
    if (node.next != null) 
        return true;

    // 下面这个方法从阻塞队列的队尾开始从后往前遍历找，如果找到相等的，说明在阻塞队列，否则就是不在阻塞队列

    // 可以通过判断 node.prev() != null 来推断出 node 在阻塞队列吗？答案是：不能。
    // 这个可以看上篇 AQS 的入队方法，首先设置的是 node.prev 指向 tail，
    // 然后是 CAS 操作将自己设置为新的 tail，可是这次的 CAS 是可能失败的。

    return findNodeFromTail(node);
}

// 从阻塞队列的队尾往前遍历，如果找到，返回 true
private boolean findNodeFromTail(Node node) {
    Node t = tail;
    for (;;) {
        if (t == node)
            return true;
        if (t == null)
            return false;
        t = t.prev;
    }
}

  ```
  
  </details>

### signal()唤醒方法执行流程
- 判断是否持有独占锁，如果没持有，直接抛出异常
- 将Condition中firstWaiter指向头结点下一个节点，并把头结点链接关系断掉，next设置为null，如果没有下个节点了，把尾结点设置为空
- 把头节点从条件队列转移到阻塞队列
  - CAS修改状态，如果修改失败，说明这个节点取消了排队，返回false然后继续取下一个节点转移
  - 修改状态成功后会自旋插入阻塞队列的队尾，插入成功后，返回队尾节点的前一个节点
  - 判断队尾前一个节点的状态，
    - 如果大于0（取消了等待），会唤醒node节点线程
    - 如果等于0，CAS把状态改为-1，也就是正常状态，如果CAS失败，也会唤醒node节点线程

<details>
  <summary>源代码</summary>
  
  ```java
  // 唤醒等待了最久的线程
// 其实就是，将这个线程对应的 node 从条件队列转移到阻塞队列
public final void signal() {
    // 调用 signal 方法的线程必须持有当前的独占锁
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}

// 从条件队列队头往后遍历，找出第一个需要转移的 node
// 因为前面我们说过，有些线程会取消排队，但是可能还在队列中
private void doSignal(Node first) {
    do {
          // 将 firstWaiter 指向 first 节点后面的第一个，因为 first 节点马上要离开了
        // 如果将 first 移除后，后面没有节点在等待了，那么需要将 lastWaiter 置为 null
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        // 因为 first 马上要被移到阻塞队列了，和条件队列的链接关系在这里断掉
        first.nextWaiter = null;
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
      // 这里 while 循环，如果 first 转移不成功，那么选择 first 后面的第一个节点进行转移，依此类推
}

// 将节点从条件队列转移到阻塞队列
// true 代表成功转移
// false 代表在 signal 之前，节点已经取消了
final boolean transferForSignal(Node node) {

    // CAS 如果失败，说明此 node 的 waitStatus 已不是 Node.CONDITION，说明节点已经取消，
    // 既然已经取消，也就不需要转移了，方法返回，转移后面一个节点
    // 否则，将 waitStatus 置为 0
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    // enq(node): 自旋进入阻塞队列的队尾
    // 注意，这里的返回值 p 是 node 在阻塞队列的前驱节点
    Node p = enq(node);
    int ws = p.waitStatus;
    // ws > 0 说明 node 在阻塞队列中的前驱节点取消了等待锁，直接唤醒 node 对应的线程。唤醒之后会怎么样，后面再解释
    // 如果 ws <= 0, 那么 compareAndSetWaitStatus 将会被调用，上篇介绍的时候说过，节点入队后，需要把前驱节点的状态设为 Node.SIGNAL(-1)
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        // 如果前驱节点取消或者 CAS 失败，会进到这里唤醒线程，之后的操作看下一节
        LockSupport.unpark(node.thread);
    return true;
}
  ```
  </details>

### 带超时机制的await()
- 其实就是从park()挂起换成了parkNanos()指定休眠时间，中间有些小判断，如果到时间了直接放到阻塞队列然后获取锁，如果时间小于1毫秒，自旋等待到时间放到阻塞队列

### awaitUninterruptibly()
- 其实就是不处理中断的await方法

### 中断的使用和处理
- 中断其实只是线程的一个状态，true代表中断，false代表正常，可以通过中断的状态去做处理。像await()、sleep()等方法都抛出了InterruptedException异常，这种方法大部分是阻塞的，也就是需要被唤醒的，这时候我们可以通过中断这个线程的方式强行唤醒这个线程，然后抛出异常，结束等待。当然这个也需要我们在代码里判断中断状态，抛出异常，一般放在循环的开始处。
- 一般并发包都会提供两种方式，一种响应中断，一种不响应中断，比如lock()、lockInterruptibly()

### Semaphore(信号量)-允许多个线程同时访问
- 构造的时候可以指定可以被多少个线程同时获取，也可以指定公平锁还是非公平锁，一般这个用来控制线程数量用的
- 实现原理就是初始的时候会把传参的线程数赋值给state，如果实际的线程数超过了这个值，就会进入阻塞队列并挂起，至少有线程释放了才可以获取到。

###  CountDownLatch （倒计时器）

#### CountDownLatch实现原理
- 构造的时候指定的count数赋值给state，当调用countDown()方法的时候，state会减1，调用await的时候，会判断state是否是0，不为0 的话就会阻塞，CountDownLatch会自旋判断state是否为0，如果等于0了就会唤醒所有等待的线程，程序才可以执行

#### CountDownLatch使用场景
- 一个线程运行前要等待其他多个线程执行完。比如启动一个服务时，需要加载所有的组件，才可以继续。
- 需要多个线程同时执行的时候，可以给多个线程一个共享的CountDownLatch对象，初始值设置为1，每个线程都调用await方法，主线程调用countDown方法，所有线程就会一起执行

#### CountDownLatch的不足
- 一次性的使用，只有被构造的时候初始化一次，用完了就不能再次使用了

### CyclicBarrier(循环栅栏)
- 使用上和CountDownLatch类型，CountDownLatch是基于AQS的，CylicBarrier是基于ReentrantLock的，CyclicBarrier的优势在于可以重置，多次使用

#### CyclicBarrier的使用场景
- 可以用于多线程执行数据，最后合并计算结果的场景。比如计算多个Sheet页的数据汇总，可以多线程汇总每个Sheet页的，最后合并
- 可以使多个线程达到某一个同步点，一起执行

#### CyclicBarrier await方法执行逻辑
- 获取锁
- 把count- 1
- 如果count = 0 了，就会唤醒所有等待线程，并重置count
- 如果count > 0那么就挂起
- 释放锁


### CyclicBarrier 和 CountDownLatch 的区别
- 对于 CountDownLatch 来说，重点是“一个线程（多个线程）等待”，而其他的 N 个线程在完成“某件事情”之后，可以终止，也可以等待。而对于 CyclicBarrier，重点是多个线程，在任意一个线程没有完成，所有的线程都必须等待。
- CountDownLatch 是计数器，线程完成一个记录一个，只不过计数不是递增而是递减，而 CyclicBarrier 更像是一个阀门，需要所有线程都到达，阀门才能打开，然后继续执行。

