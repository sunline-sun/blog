### synchronized介绍
- synchronized解决了线程之间访问资源的同步性，保证被他修饰的代码块或者方法任意时间只有一个线程使用
- 早期Java的snchronizied使用的重量级锁，效率很低（因为是基于操作系统的Mutex lock实现的，Java线程映射到操作系统的原生线程上，切换线程时需要用户态到内核态的切换，耗费时间），Java6后做了优化

### 实际应用中是如何应用synchronized的
- 
