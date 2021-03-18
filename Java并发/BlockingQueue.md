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
- LinkedBlockingQueue是通过两个ReenTrantLock和两个condition实现的，其中有几个属性：队列容量、队列中元素数量、对头、队尾

#### LinkedBlockingQueue实现原理
- 如果要获取一个元素，首先要获取读锁，如果队列为空，插入读队列
- 如果插入一个元素，首先获取到写锁，如果队列满了，插入写队列
