### ThreadLocal的数据结构
- Thread类有一个TreadLocal.ThreadLocalMap的实例变量，每个线程都有自己的一个ThreadLocalMap
- ThreadLocalMap类似于HashMap，但是只有数组结构，没有链表，里面的节点是Entry，继承自Weakreference，也就是弱引用类型
- ThreadLocalMap的key为ThreadLocal（实际是一个ThreadLocal的弱引用），value为实际存储的Object对象
- 每个线程读写ThreadLocal的时候，都是操作ThreadLocalMap，从而实现线程之间的分离

### 四种引用类型
- 强引用：我们常常new出来的对象就是强引用类型，只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足的时候
- 软引用：使用SoftReference修饰的对象被称为软引用，软引用指向的对象在内存要溢出的时候被回收
- 弱引用：使用WeakReference修饰的对象被称为弱引用，只要发生垃圾回收，若这个对象只被弱引用指向，那么就会被回收
- 虚引用：虚引用是最弱的引用，在 Java 中使用 PhantomReference 进行定义。虚引用中唯一的作用就是用队列接收对象即将死亡的通知

### ThreadLocal.get()请求的时候GC，之后key是否为null
- 不会为null，正常来说ThreadLocalMap中的key为弱引用，GC后会回收，但是get()请求证明还有强引用存在，GC不会回收

### ThreadLocal内存泄漏
- ThreadLocalMap中的key为弱引用，GC后会回收，但是value对象不会回收，导致内存泄漏

### ThreadLocal.set()方法
- 获取当前Thread的Threadlocals变量
- 如果不存在创建新的ThreadLocalMap，如果存在往ThreadLocalMap里set值

### ThreadLocalMap.set()方法
- 通过与运算获取数组中的位置，key.threadLocalHashCode & (len-1)，ThreadLocalMap中计算hash的方式是采用斐波那契数当做计算因子，每新增一个ThreadLocal，hash值就会增加斐波那契数大小，通过这个来保证在散列数组中的均匀分布
- 判断这个位置是否有值，没有就插入，有的话判断key是否相等，相等的话覆盖数据，然后返回
- 否则会向后查找，直到找到一个为空的位置，然后插入（这个和HashMap是有区别的，因为没有链表结构）
- 向后查找的过程发现这个Entry的key为null（GC回收了），这时候会替换过期的数据，
