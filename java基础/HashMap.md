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
 <details>

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

