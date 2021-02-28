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

- 上述代码做了三件事：
   - 计算新桶数组的容量 newCap 和新阈值 newThr
   - 根据计算出的 newCap 创建新的桶数组，桶数组 table 也是在这里进行初始化的
   - 将键值对节点重新映射到新的桶数组里。如果节点是 TreeNode 类型，则需要拆分红黑树。如果是普通节点，则节点按原顺序进行分组。
- 源码分析（方法resize在初始化和扩容时都调用，所以第一步判断是扩容还是初始化）newCap（新数组大小），newThr（新阈值）
   - oldCap > 0，表示数组已经初始化过。oldCap >= MAXIMUM_CAPACITY大于等于最大数组长度，则不扩容；否则按照原数组和原阈值的大小的2倍扩容
   - oldThr > 0，表示HashMap的初始化，调用第二个或者第三个初始化函数，由于初始化时，用阈值大小暂时保存初始化数组的值，所以在此将阈值赋值给newCap
   - 其他情况，表示调用的HashMap的无参构造函数，将默认的阈值和数组大小赋值给变量newCap，newThr
   - 当newThr=0，按阈值公式计算
   - 创建长度为newCap的桶数组
   - 如果旧数组桶不为空，遍历数组桶中的数据，重新计算之后映射到新的桶中。  
   - 如果该节点只有一个节点，直接通过hash*(n-1)映射到新桶
   - 如果该节点类型是红黑树，通过红黑树方法映射
   - 如果该节点是链表类型，先通过节点上的key对长度取余，将数据分为=0和!=0两类，映射到新的桶中，顺序不会发生改变
 JDK1.8的扩容速度高于JDK1.7，1.7为了键值对的均匀分布，在计算hash值时加入了随机种子，在扩容过程中，相关方法会根据容量判断是否需要生成新的随机种子，并重新计算所有节点的 hash。而在 JDK1.8 中，则通过引入红黑树替代了该种方式。从而避免了多次计算 hash 的操作，提高了扩容效率。


   
