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
 ```Java
    get(Object obj){
      return (e = getNode(hash(key),key)) == null ? null : e.value;
    } 
 ```
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
