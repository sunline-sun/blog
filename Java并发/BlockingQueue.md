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


### 各个队列的区别和使用场景
