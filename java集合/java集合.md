![图片](https://user-images.githubusercontent.com/55612309/109374255-dc63a700-78ee-11eb-83e6-c28c5543c43d.png) 集合类父子关系图

### collection接口下的集合
- List
   - ArrayList Object[]数组
   - LinkedList 双向链表
   - Vector Object[]数组
- Set
   - HashSet（无序，唯一） 基于HashMap实现，底层是HashMap存储数据 
   - LinkedHashSet  基于LinkedHashMap实现
   - TreeSet（有序） 基于红黑树实现（自平衡的排序二叉树）

### map接口下的集合
- HashMap jdk1.8之前是通过数组加链表实现，链表主要为了解决哈希冲突的问题，jdk1.8后引入红黑树提高访问性能，当链表超过阈值（默认为8）时，链表转为红黑树
 （转换前会判断当前数组长度是否超过64，如果不够，会先选择扩容）
- LinkedHashMap 继承自HashMap，但是额外维护了一个双向链表，保证插入的顺序，通过对链表的操作，实现访问顺序。
- HashTable 数组加链表的结构
- TreeMap 红黑树（自平衡的排序二叉树）

### ArrayList 和 LinkedList区别
- 底层数据结构：ArrayList底层是数组结构，LinkedList底层是双向链表
- 插入和删除速度：ArrayList插入删除收到插入位置影响，插入尾部，O(1),插入头部，O(N-k)。LinkedList插入删除受到位置影响，插入尾部，O(1)，插入中间O(N)。
- 随机访问速度，ArrayList O(1)复杂度，LinkedList O(N)

### 双向链表和双向循环链表的区别
- 双向循环链表就是在双向链表的基础上，尾结点的next指针指向了头结点，头节点的prev指针指向了尾结点，构成了一个环

### Comparable和Comparator区别
- comparable是java.lang下面的包，通过compareTo（Object obj1）方法实现排序
   - 实现方式：实现comparable接口，重写compareTo方法，实现自己的排序方式   
- comparator是java.util下的包，通过compare（Object obj1,Object obj2）方法实现排序
   - Collections.sort(obj,new Comparator(){compare(实现自己的排序方式)})

### HashSet、LinkedHashSet、TreeSet区别
- HashSet底层是HashMap，线程不安全，可以存储null
- LinkedHashSet 维护了插入的顺序
- TreeSet，底层是红黑树，维护了插入的顺序，排序方式有自然排序和定制排序

### HashMap和HashTable区别
- 线程是否安全。HashMap不安全，HashTable不安全
- 初始容量和扩容机制，HashMap默认16，扩容是2N，HashMap默认11，扩容2N+1
- 底层数据结构，HashMap有链表超过8转为红黑树优化，HashTable没有
- 对null的支持，HashMap支持，HashTable不支持

### HashMap和TreeMap的区别
- TreeMap 和HashMap都实现了AbstractMap，但是TreeMap多实现了两个接口，NavigableMap和sortedMap，NavigableMap提供了搜索的能力，sortedMap提供了排序的功能。TreeMap的排序也可以用自己排序方式，构造的时候就可以指定，TreeMap a = new TreeMap(new Comparator(){compare()});

### HashMap多线程死循环的问题
- jdk8之前多线程操作HashMap有死循环问题，1.8后解决了，但是还会有数据丢失的问题，所以不建议多线程下使用HashMap。


