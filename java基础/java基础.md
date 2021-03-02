### 深拷贝 vs 浅拷贝
- 浅拷贝：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。
- 深拷贝：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。

### 为什么重写 equals 时必须重写 hashCode 方法？
如果两个对象相等，则 hashcode 一定也是相同的。两个对象相等,对两个对象分别调用 equals 方法都返回 true。但是，两个对象有相同的 hashcode 值，它们也不一定是相等的 。因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖。

###  String StringBuffer 和 StringBuilder 的区别是什么? String 为什么是不可变的?
- String 类中使用 final 关键字修饰字符数组来保存字符串，private final char value[]，所以String 对象是不可变的。
- 而 StringBuilder 与 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串char[]value 但是没有用 final 关键字修饰，所以这两种对象都是可变的
- 补充（来自issue 675）：在 Java 9 之后，String 、StringBuilder 与 StringBuffer 的实现改用 byte 数组存储字符串 private final byte[] value
- 线程安全性：String 中的对象是不可变的，也就可以理解为常量，线程安全。AbstractStringBuilder 是 StringBuilder 与 StringBuffer 的公共父类，定义了一些字符串的基本操作，如 expandCapacity、append、insert、indexOf 等公共方法。StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。StringBuilder 并没有对方法进行加同步锁，所以是非线程安全的。
- 性能：每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象。StringBuffer 每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 StringBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

### == 与 equals区别
- == : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)。
- equals() : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：
   - 情况 1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
   - 情况 2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来比较两个对象的内容是否相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。

### transient 关键字作用
- 阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复。transient 只能修饰变量，不能修饰类和方法。

### 反射的应用场景
- 我们在使用 JDBC 连接数据库时使用 Class.forName()通过反射加载数据库的驱动程序；
- Spring 框架的 IOC（动态加载管理 Bean）创建对象以及 AOP（动态代理）功能都和反射有联系；
- 动态配置实例的属性；

### 常见的异常
![图片](https://user-images.githubusercontent.com/55612309/109501773-0a094580-7ad3-11eb-8d7c-2c24d45c91ed.png)
![图片](https://user-images.githubusercontent.com/55612309/109501908-3b821100-7ad3-11eb-9f63-1253ac077941.png)
- Exception :程序本身可以处理的异常，可以通过 catch 来进行捕获。Exception 又可以分为 受检查异常(必须处理) 和 不受检查异常(可以不处理)
- Error ：Error 属于程序无法处理的错误 ，我们没办法通过 catch 来进行捕获 。例如，Java 虚拟机运行错误（Virtual MachineError）、虚拟机内存不够错误(OutOfMemoryError)、类定义错误（NoClassDefFoundError）等 。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。
除了RuntimeException及其子类以外，其他的Exception类及其子类都属于受检查异常 。常见的受检查异常有： IO 相关的异常、ClassNotFoundException 、SQLException...。
- 受检查异常：Java 代码在编译过程中，如果受检查异常没有被 catch/throw 处理的话，就没办法通过编译 。除了RuntimeException及其子类以外，其他的Exception类及其子类都属于受检查异常 。常见的受检查异常有： IO 相关的异常、ClassNotFoundException 、SQLException...。
- 不受检查异常：Java 代码在编译过程中 ，我们即使不处理不受检查异常也可以正常通过编译。RuntimeException 及其子类都统称为非受检查异常，例如：NullPoin​terException、NumberFormatException（字符串转换为数字）、ArrayIndexOutOfBoundsException（数组越界）、ClassCastException（类型转换错误）、ArithmeticException（算术错误）等。

### 常见的IO模型
- 同步阻塞 I/O、同步非阻塞 I/O、I/O 多路复用、信号驱动 I/O 和异步 I/O
- 
- BIO (Blocking I/O): BIO 属于同步阻塞 IO 模型 。BIO是一个连接一个线程。同步阻塞 IO 模型中，应用程序发起 read 调用后，会一直阻塞，直到在内核把数据拷贝到用户空间。。![图片](https://user-images.githubusercontent.com/55612309/109654403-1eb21000-7b9d-11eb-8c32-30191ed63143.png)
- NIO (Non-blocking/New I/O): NIO 是一种同步非阻塞的 I/O 模型，NIO是一个请求一个线程.同步非阻塞 IO 模型中，应用程序会一直发起 read 调用，等待数据从内核空间拷贝到用户空间的这段时间里，线程依然是阻塞的，直到在内核把数据拷贝到用户空间。![图片](https://user-images.githubusercontent.com/55612309/109656209-24105a00-7b9f-11eb-970d-e036c1b0d1b4.png)

- I/O 多路复用，IO 多路复用模型中，线程首先发起 select 调用，询问内核数据是否准备就绪，等内核把数据准备好了，用户线程再发起 read 调用。read 调用的过程（数据从内核空间->用户空间）还是阻塞的。![图片](https://user-images.githubusercontent.com/55612309/109656190-1f4ba600-7b9f-11eb-8aca-db53d4d7eeb2.png)

- AIO (Asynchronous I/O): AIO 也就是 NIO 2。AIO是一个有效请求一个线程。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的 IO 模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。![图片](https://user-images.githubusercontent.com/55612309/109656315-41ddbf00-7b9f-11eb-9b69-afd772970cb6.png)
- ![图片](https://user-images.githubusercontent.com/55612309/109658477-98e49380-7ba1-11eb-820e-efa5e4bb21d3.png)



### Bigdicimal精度丢失
![图片](https://user-images.githubusercontent.com/55612309/109504153-2d81bf80-7ad6-11eb-8a91-f3fff6dda847.png)

### 



