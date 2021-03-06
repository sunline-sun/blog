### HashMap原理
- HashMap 基于散列算法实现，散列算法包括散列在探测和拉链式，HashMap采用的是拉链式，数据结构为数组+链表+红黑树， HashMap允许key为null，null的hash值为0。

### 构造方法
- hashMap() 无参构造方法，将负载因子设为默认值
- hashMap(int initialCapacity) 参数是初始容量，然后调用第三个构造方法
- hashMap(int initialCapacity,float loadFactor) 参数是初始容量、负载因子，设置负载因子、阈值
- HashMap(Map<? extends k,? extends v>m) 参数是map，作用是将一个map的映射拷贝到自己的存储结构中

### 初始容量，负载因子，阈值
- 初始容量并没有用于初始化数据结构
- 初始容量默认为16，负载因子默认为0.75
- 阈值 = 初始容量 * 负载因子
- 阈值在初始化时候不是遵循初始容量 * 负载因子的公式计算的，而是找到大于等于初始容量的最小2次幂，找到的方式是通过位运算。https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15159249414047.jpg
- 负载因子可以反映HashMap桶数组的使用情况，调低负载因子大小，HashMap容纳的键值对就会变少，扩容时，键与键之间的碰撞几率降低，链表变短，增删改查效率提升，也是就是空间换时间。调高负载因子，反之，就是时间换空间

### 查询
<details>
<summary>源代码</summary>
 
  ```Java
    public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 1. 定位键值对所在桶的位置
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // 2. 如果 first 是 TreeNode 类型，则调用黑红树查找方法
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                
            // 2. 对链表进行查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
 ```
 </details>

 #### 查询步骤
- 找到键值对对应桶数组中的位置，判断当前hash是否和参数key的hash一致、key是否一致，如果一致，返回，不一致继续下一步
   - 通过与运算获取对应桶位置，tab[(n-1) & hash]，之所以这样计算是因为位运算比直接取余（hash%length）效率高
- 判断当前桶结构是否是TreeNode类型，如果是，调用红黑树查找方法，不是，继续下一步
- 对链表进行遍历查找，判断hash和key是否和参数一致，一致返回
- 没有找到对应数据返回null

### 重要方法 hash()
```Java
   hash(Object key){
     int h;
     //之所以又做了一步异或运算，没有直接使用hashCode的返回值，是因为后面取余的时候是和length取余，length一般不会太大，会导致取余的时候基本用的是低位运算，高位不参与运算，
     //通过异或运算，可以让高位参与hash运算中，加大低位的随机性，还可以增加hash运算的复杂度。之所以是右移16位，是因为hashCode()生成的hash值是int类型，32位，前16位是高位，后16位是低位
     return key == null ? null : (h = key.hashCode()) ^ (h >>> 16)
   }
```
### 遍历
- 一般通过keySet()或者entrySet()方法遍历，这两种方式编译后就是用的各自迭代器方式
代码日下

<details>
<summary>源代码</summary>

```java
//keySet()遍历
  Set keys = map.keySet();
  for(String key : keys){
    dosomething~
  }
  //迭代器遍历
  Set keys = map.keySet();
  Iterator it = keys.iterator();
  while(it.hasnext()){
   Object key = it.next();
  }
  public Set<K> keySet() {
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
}

/**
 * 键集合
 */
final class KeySet extends AbstractSet<K> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator<K> iterator()     { return new KeyIterator(); }
    public final boolean contains(Object o) { return containsKey(o); }
    public final boolean remove(Object key) {
        return removeNode(hash(key), key, null, false, true) != null;
    }
    // 省略部分代码
}

/**
 * 键迭代器
 */
final class KeyIterator extends HashIterator 
    implements Iterator<K> {
    public final K next() { return nextNode().key; }
}

abstract class HashIterator {
    Node<K,V> next;        // next entry to return
    Node<K,V> current;     // current entry
    int expectedModCount;  // for fast-fail
    int index;             // current slot

    HashIterator() {
        expectedModCount = modCount;
        Node<K,V>[] t = table;
        current = next = null;
        index = 0;
        if (t != null && size > 0) { // advance to first entry 
            // 寻找第一个包含链表节点引用的桶
            do {} while (index < t.length && (next = t[index++]) == null);
        }
    }

    public final boolean hasNext() {
        return next != null;
    }

    final Node<K,V> nextNode() {
        Node<K,V>[] t;
        Node<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        if ((next = (current = e).next) == null && (t = table) != null) {
            // 寻找下一个包含链表节点引用的桶
            do {} while (index < t.length && (next = t[index++]) == null);
        }
        return e;
    }
    //省略部分代码
}
```
</details>

#### 遍历步骤
1. 获取keySet对象，keySet中包含keyIterator迭代器，遍历时调用的就是他的next()方法
2. keyIterator的next()方法里就是获取了它的父类HashIterator的nextKey()的key并返回
3. HashIterator的nextKey()方法就是获取下一个不为空的桶并返回。

##### entrySet同理，只是entryIterator的next()方法返回了HashIterator的nextKey()返回的整个node节点，而不是只返回了key

#### 为什么遍历结果和插入顺序不一致
- 因为遍历是通过遍历数组结构返回的，插入的时候key经过哈希运算插入的位置是随机的，而不是有序的

### 插入
##### table的初始化是在插入步骤执行的
<details>
<summary>源代码</summary>

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 初始化桶数组 table，table 被延迟到插入新数据时再进行初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 如果桶中不包含键值对节点引用，则将新键值对节点的引用存入桶中即可
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 如果键的值以及节点 hash 等于链表中的第一个键值对节点时，则将 e 指向该键值对
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
            
        // 如果桶中的引用类型为 TreeNode，则调用红黑树的插入方法
        else if (p instanceof TreeNode)  
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 对链表进行遍历，并统计链表长度
            for (int binCount = 0; ; ++binCount) {
                // 链表中不包含要插入的键值对节点时，则将该节点接在链表的最后
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 如果链表长度大于或等于树化阈值，则进行树化操作
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                
                // 条件为 true，表示当前链表包含要插入的键值对，终止遍历
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        
        // 判断要插入的键值对是否存在 HashMap 中
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // onlyIfAbsent 表示是否仅在 oldValue 为 null 的情况下更新键值对的值
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 键值对数量超过阈值时，则进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
</details>

#### 插入核心逻辑
- 当桶数组 table 为空时，通过扩容的方式初始化 table
- 查找要插入的键值对是否已经存在，存在的话根据条件判断是否用新值替换旧值
- 如果不存在，则将键值对链入链表中，并根据链表长度决定是否将链表转为红黑树
- 判断键值对数量是否大于阈值，大于的话则进行扩容操作
### 扩容机制
#### 背景
- 在 HashMap 中，桶数组的长度均是2的幂，阈值大小为桶数组长度与负载因子的乘积。当 HashMap 中的键值对数量超过阈值时，进行扩容。
#### 扩容过程
- HashMap 的扩容机制与其他变长集合的套路不太一样，HashMap 按当前桶数组长度的2倍进行扩容，阈值也变为原来的2倍（如果计算过程中，阈值溢出归零，则按阈值公式重新计算）。扩容之后，要重新计算键值对的位置，并把它们移动到合适的位置上去。
#### 扩容源码
<details>
<summary>源代码</summary>

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 如果 table 不为空，表明已经初始化过了
    if (oldCap > 0) {
        // 当 table 容量超过容量最大值，则不再扩容
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        } 
        // 按旧容量和阈值的2倍计算新容量和阈值的大小
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    } else if (oldThr > 0) // initial capacity was placed in threshold
        /*
         * 初始化时，将 threshold 的值赋值给 newCap，
         * HashMap 使用 threshold 变量暂时保存 initialCapacity 参数的值
         */ 
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        /*
         * 调用无参构造方法时，桶数组容量为默认容量，
         * 阈值为默认容量与默认负载因子乘积
         */
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    
    // newThr 为 0 时，按阈值计算公式进行计算
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    // 创建新的桶数组，桶数组的初始化也是在这里完成的
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 如果旧的桶数组不为空，则遍历桶数组，并将键值对映射到新的桶数组中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // 重新映射时，需要对红黑树进行拆分
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 遍历链表，并将链表节点按原顺序进行分组
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 将分组后的链表映射到新桶中
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```
</details>

#### 扩容步骤
- 判断是初始化还是扩容，然后初始化相关参数，初始容量、阈值。如果是有参初始化那么旧的阈值就是初始容量。
- 根据参数创建桶数组
- 复制旧桶数组数据到新桶数组，循环旧桶数组，如果桶上只有一个节点，直接取余定位下标然后复制到新桶，如果桶上是红黑树结构，需要进行拆分复制，如果桶上是链表，则循环链表，通过判断hash与length是否为0将链表分为两个新链表，取余为0的代表扩容后的位置不变，不为0的扩容后位置为旧下标+旧length，最后将两个新链表插入到新的桶数组对应位置

#### 链表树化
##### 插入数据过程中，如果链表长度超过8并且桶数组长度超过64，会将链表转为红黑树。之所以判断数组长度，是因为数组长度小的时候hash碰撞几率比较高，链表长度容易变长，扩容效果更好。
##### 树化过程
- 将Node<>转为TreeNode<>格式，TreeNode中保留了next属性，方便后面切分或者转回链表
- 对链表进行排序，以实现红黑树存储方式
   - 比较hash大小，如果hash相同，继续比较
   - 看对象是否有compare方法，有的话调用compare方法比较，没有的话继续
   - 进行仲裁的方式比较。
#### 红黑树拆分
##### 扩容的时候，节点需要重新映射，因为红黑树通过next和prev保存了链表的顺序，所以可以直接按照链表的映射方式进行映射。
##### 拆分过程
- 按照链表的方式（next）对红黑树进行遍历，拆分为两个TreeNode组成的链表
- 进行判断，如果两个新链表的长度小于6，那么会把红黑树转为链表插入新的桶数组，如果拆分后树结构没动，那么直接插入通数组，否则进行树化后插入桶数组

#### 红黑树链化
##### 因为TreeNode中存储了next结构，保留了源链表的顺序，就是遍历TreeNode，将TreeNode节点转为Node节点的过程。

### 删除
- 通过取余获取桶的位置
- 找到要删除的节点
   - 判断当前节点hash和key是否相等，相等的话这个就是要删除的节点，否则继续
   - 如果是红黑树，就用红黑树的查找方式找到这个要删除的节点
   - 遍历链表，找到要删除的节点
- 删除节点，并修复链表和红黑树，就是把next指针指向删除的下一个节点

### hashMap为什么没有使用默认的序列化
- 因为桶数组中的数据存储是随机的，用默认的序列化浪费空间
- 不同的jvm环境下，获取的hash值是不一样的，所以桶数组结构可能不一样，序列化的数据不同的jvm下可能不能正确反序列化
##### 所以hashMap通过实现readObject/writeObject 自定义了序列化的内容，只序列化了键值对，这样各种环境都可以通过键值对构建桶数组，也节约空间
   
