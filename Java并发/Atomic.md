### Atomic原子类
- 其实就是具有原子操作特性的类

### CAS ABA问题
- 就是一个线程获取到了变量的值为A，另外一个线程也获取到了变量的值，然后把A改成了B，又把B改回了A，这时候第一个线程CAS操作的时候，会当做拿到的变量没有被人改过，其实已经改过了。

### 基本类型原子类
- AtomicInteger：整型原子类
- AtomicLong：长整型原子类
- AtomicBoolean ：布尔型原子类

#### AtomicInteger实现原子性的原理就是通过CAS加volatile实现的

### 数组类型原子类
- AtomicIntegerArray：整形数组原子类
- AtomicLongArray：长整形数组原子类
- AtomicReferenceArray ：引用类型数组原子类

### 引用类型原子类
基本数据类型只能一次性修改一个变量，需要修改多个变量的时候可以使用引用类型原子类
- AtomicReference：引用类型原子类
- AtomicStampedReference：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。
- AtomicMarkableReference ：原子更新带有标记的引用类型。该类将 boolean 标记与引用关联起来

### 对象的属性修改类型原子类
- AtomicIntegerFieldUpdater:原子更新整形字段的更新器
- AtomicLongFieldUpdater：原子更新长整形字段的更新器
- AtomicReferenceFieldUpdater ：原子更新引用类型里的字段的更新器
