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
- LinkedHashMap 
