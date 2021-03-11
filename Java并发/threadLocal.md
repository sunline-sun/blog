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
- 向后查找的过程发现这个Entry的key为null（GC回收了），这时候会执行replaceStaleEntry()方法，进行替换过期数据
- 向后查找过程中发现key相等的位置，会直接替换（出现这种情况是因为hash冲突的解决方式是向后查找，所以hash一样的数据下标位置不一定一样）
- 如果是没有通过查找而是直接插入的走下面的逻辑
- size++
- 调用cleanSomeslots()做一起启发性清理 
- 如果没有清理任何数据并且size超过了阈值（数组长度的三分之二），或进行扩容rehash()操作
- rehash()方法里会再一次探测式清理过期key，完成后如果size>=threshold-threshold/4，则进行真正的扩容

<details>
  <summary>源代码</summary>
  
  ```java
  private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        `ThreadLocal`<?> k = e.get();

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
  ```
 
  </details>

### relaceStaleEntry()方法，替换过期数据，使用了探测式数据清理
- 把slotToExpunge赋值当前节点下标，然后当前节点开始向前迭代查找，找其他过期的数据，找到了就更新slotToExpunge的值，知道碰到entry为null为止
- 然后从当前节点开始向后迭代查找，
  - 如果找到key相等的entry元素，下标为i，则交换两个元素，目的是把这个元素放到hash确定的下标位置，优化结构
    - 判断slotToexpunge和staleSlot的值是否相等，相等则代表没有找到过期数据，把slotToExpunge赋值为i
    - 进行数据清理，从slotToExpunge开始，先探测式清理，在启发式清理
  - 如果找到key为null的也就是过期数据，判断slotToexpunge和staleSlot是否想等，也就是说向前找的时候是否找到了已过期的数据，没找到就把这个下标赋值给slotToexpunge
- 如果向后找查找时候没有找到相等的key，创建一个新的Entry对象，替换当前的节点（因为当前节点是过期数据）
- 判断slotToExpunge不等于当前节点，说明有过期数据，进行清理，先探测式清理，后启发式清理。

<details>
  <summary>源代码</summary>
  
  ```java
  private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                                       int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;
    Entry e;

    int slotToExpunge = staleSlot;
    for (int i = prevIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = prevIndex(i, len))

        if (e.get() == null)
            slotToExpunge = i;

    for (int i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {

        `ThreadLocal`<?> k = e.get();

        if (k == key) {
            e.value = value;

            tab[i] = tab[staleSlot];
            tab[staleSlot] = e;

            if (slotToExpunge == staleSlot)
                slotToExpunge = i;
            cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
            return;
        }

        if (k == null && slotToExpunge == staleSlot)
            slotToExpunge = i;
    }

    tab[staleSlot].value = null;
    tab[staleSlot] = new Entry(key, value);

    if (slotToExpunge != staleSlot)
        cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
}
  ```
  
  </details>

### 两种清理方式：探测式清理和启发式清理
#### expungeStaleEntry() 探测式清理
 - 从开始位置开始向后进行探测式清理，讲过期数据设置为null
 - 碰到有效的数据，先判断这个数据的位置是否经过偏移，如果是会重新进行hash与运算定位新的位置，如果新的位置有数据，会找到最近的一个entry为null 的位置，保证离正确桶的位置近一些
 - 直到碰到entry为空循环终止

<details>
  <summary>源代码</summary>
  
  ```java
  private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        `ThreadLocal`<?> k = e.get();
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;

                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
  ```
  
  </details>
  
#### cleanSomeSlots（）启发式清理
- 从当前节点开始，向后遍历，循环条件是n>>>1 != 0，也就是长度是16，遍历长度就是4次
- 如果循环过程中有过期数据，调用探测式清理进行数据清理，同时重置n为数组长度，也就是会再次遍历4次
- 遍历结束后如果还是没有发现过期数据，则返回

<details>
  <summary>源代码</summary>
  
  ```java
  private boolean cleanSomeSlots(int i, int n) {
    boolean removed = false;
    Entry[] tab = table;
    int len = tab.length;
    do {
        i = nextIndex(i, len);
        Entry e = tab[i];
        if (e != null && e.get() == null) {
            n = len;
            removed = true;
            i = expungeStaleEntry(i);
        }
    } while ( (n >>>= 1) != 0);
    return removed;
}
  ```
  
  </details>
  
### ThreadLocalMap扩容
- set()方法最后如果size >= threshold * 2/3 的时候触发rehash（）扩容方法
- 进行整个数组的探测式清理
- 如果探测式清理后的size >= threshold - threshold * 1/4，也就是threshold的四分之三，调用resize()进行真正的扩容
  - 创建新的数组，大小为旧数组的两倍
  - 循环旧数组，entry不为空的话且key不为空，对key进行hash与运算确定插入位置，如果此位置有值，向后查找最近的一个entry为null的位置插入
 
 <details>
  <summary>rehash源代码</summary>
  
  ```java
  private void rehash() {
    expungeStaleEntries();

    if (size >= threshold - threshold / 4)
        resize();
}

private void expungeStaleEntries() {
    Entry[] tab = table;
    int len = tab.length;
    for (int j = 0; j < len; j++) {
        Entry e = tab[j];
        if (e != null && e.get() == null)
            expungeStaleEntry(j);
    }
}
  ```
  
  </details>
  
  <details>
  <summary>resize源代码</summary>
  
  ```java
  private void resize() {
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
    int newLen = oldLen * 2;
    Entry[] newTab = new Entry[newLen];
    int count = 0;

    for (int j = 0; j < oldLen; ++j) {
        Entry e = oldTab[j];
        if (e != null) {
            `ThreadLocal`<?> k = e.get();
            if (k == null) {
                e.value = null;
            } else {
                int h = k.threadLocalHashCode & (newLen - 1);
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                newTab[h] = e;
                count++;
            }
        }
    }

    setThreshold(newLen);
    size = count;
    table = newTab;
}
  ```
  
  </details>
  
### ThreadLocalMap.get()方法
- 通过hash与运算找到对应数组位置
  - 如果entry存在并且key相等，直接返回
  - 如果entry为null，直接返回null
  - 如果entry不为null并且key不相等，则向后遍历，直到entry为null
    - 如果循环过程中key相等，则返回
    - 如果key为空，说明是过期数据，调用清理逻辑，从当前位置开始

<details>
  <summary>源代码</summary>
  
  ```java
  private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        `ThreadLocal`<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
  ```
  
  </details>
  
### InheritableThreadLocal实现原理
- InheritableThreadLocal可以实现子线程可以访问父线程的数据
- 实现原理是子线程是通过在父线程中调用new thread()方式创建的，创建的构造方法中有init方法，会把父线程的数据拷贝给子线程，实现数据访问
- 这种实现在使用线程池的情况下会有问题，因为线程池是线程的复用，解决方式可以用阿里巴巴的TransmittableThreadLocal组件

### ThreadLocal使用场景

### 
