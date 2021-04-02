### Caffeine cache和guava功能对比
![image](https://user-images.githubusercontent.com/55612309/113386446-d9ac1400-93bc-11eb-8085-f97e85d0f1ea.png)

### 性能对比
- guava的读写操作夹杂着过期时间的处理，也就是put的时候可以会做淘汰操作
- Caffeine的淘汰操作是异步的，将事件提交至队列（使用 Disruptor RingBuffer），然后会通过默认的 ForkJoinPool.commonPool()，或自己配置的线程池，进行取队列操作，然后再进行后续的淘汰、过期操作。

### 命中率
- Guava 使用 S-LRU 分段的最近最少未使用算法
- Caffeine 采用了一种结合 LRU、LFU 优点的算法：W-TinyLFU，其特点是：高命中率、低内存占用。

### 几种淘汰策略

#### LRU
- 最近最少访问，每次把访问的数据放到队列头，队列满了淘汰尾部的数据
- 需要维护每个数据的访问频率，每次访问都需要更新，消耗很大
- 其缺点是，如果某一时刻大量数据到来，很容易将热点数据挤出缓存

#### LFU
- 淘汰一定时间内被访问次数最少的数据，用Queue保存访问记录
- 优点是避免了LRU的缺点，不会被突然的大量请求挤掉热点数据
- 缺点是容易被偶发性、周期性的批量操作，降低命中率

#### TinyLFU
- TinyLFU 维护了近期访问记录的频率信息，不同于传统的 LFU 维护整个生命周期的访问记录，所以他可以很好地应对突发性的热点事件（超过一定时间，这些记录不再被维护）。这些访问记录会作为一个过滤器，当新加入的记录（New Item）访问频率高于将被淘汰的缓存记录（Cache Victim）时才会被替换。流程如下：
- ![image](https://user-images.githubusercontent.com/55612309/113387359-c437e980-93be-11eb-9231-b7735bdec8a8.png)
- 尽管维护的是近期的访问记录，但仍然是非常昂贵的，TinyLFU 通过 Count-Min Sketch 算法来记录频率信息，它占用空间小且误报率低，关于 Count-Min Sketch 算法可以参考论文：pproximating Data with the Count-Min Data Structure

#### W-TinyLFU
- W-TinyLFU 是 Caffeine 提出的一种全新算法，它可以解决频率统计不准确以及访问频率衰减的问题。这个方法让我们从空间、效率、以及适配举证的长宽引起的哈希碰撞的错误率上做均衡。
- 算法：当一个数据进来的时候，会进行筛选比较，进入W-LRU窗口队列，以此应对流量突增，经过淘汰后进入过滤器，通过访问访问频率判决是否进入缓存。如果一个数据最近被访问的次数很低，那么被认为在未来被访问的概率也是最低的，当规定空间用尽的时候，会优先淘汰最近访问次数很低的数据；
- 优点：使用Count-Min Sketch算法存储访问频率，极大的节省空间；定期衰减操作，应对访问模式变化；并且使用window-lru机制能够尽可能避免缓存污染的发生，在过滤器内部会进行筛选处理，避免低频数据置换高频数据。
- 缺点：是由谷歌工程师发明的一种算法，目前已知应用于Caffeine Cache组件里，应用不是很多。
- W-TinyLFU 算法是对 TinyLFU算法的优化，能够很好地解决一些稀疏的突发访问元素。在一些数目很少但突发访问量很大的场景下，TinyLFU将无法保存这类元素，因为它们无法在短时间内积累到足够高的频率，从而被过滤器过滤掉。W-TinyLFU 将新记录暂时放入 Window Cache 里面，只有通过 TinLFU 考察才能进入 Main Cache。大致流程如下图：
- ![image](https://user-images.githubusercontent.com/55612309/113387431-e6316c00-93be-11eb-998c-8ef93b07ff3c.png)

### 实践1
- 配置方式：设置 maxSize、refreshAfterWrite，不设置 expireAfterWrite
- 存在问题：get 缓存间隔超过 refreshAfterWrite 后，触发缓存异步刷新，此时会获取缓存中的旧值
- 适用场景
  - 缓存数据量大，限制缓存占用的内存容量
  - 缓存值会变，需要刷新缓存
  - 可以接受任何时间缓存中存在旧数据
- ![image](https://user-images.githubusercontent.com/55612309/113387547-1aa52800-93bf-11eb-9ed2-16516d5c1e5a.png)
- 设置 maxSize、refreshAfterWrite，不设置 expireAfterWrite

### 实践2
- 配置方式：设置 maxSize、expireAfterWrite，不设置 refreshAfterWrite
- 存在问题：get 缓存间隔超过 expireAfterWrite 后，针对该 key，获取到锁的线程会同步执行 load，其他未获得锁的线程会阻塞等待，获取锁线程执行延时过长会导致其他线程阻塞时间过长
- 适用场景
  - 缓存数据量大，限制缓存占用的内存容量
  - 缓存值会变，需要刷新缓存
  - 不可以接受缓存中存在旧数据
- 同步加载数据延迟小（使用 redis 等）
- ![image](https://user-images.githubusercontent.com/55612309/113387644-4b855d00-93bf-11eb-8f6b-e7188dbc12a0.png)
- 设置 maxSize、expireAfterWrite，不设置refreshAfterWrite

### 实践3
- 配置方式：设置 maxSize，不设置 refreshAfterWrite、expireAfterWrite，定时任务异步刷新数据
- 存在问题：需要手动定时任务异步刷新缓存
- 适用场景：
  - 缓存数据量大，限制缓存占用的内存容量
  - 缓存值会变，需要刷新缓存
  - 不可以接受缓存中存在旧数据
- 同步加载数据延迟可能会很大
- ![image](https://user-images.githubusercontent.com/55612309/113387717-72439380-93bf-11eb-9b25-97af8c361308.png)
- 设置 maxSize，不设置 refreshAfterWrite、expireAfterWrite，定时任务异步刷新数据

### 实践4
- 配置方式：设置 maxSize、refreshAfterWrite、expireAfterWrite，refreshAfterWrite < expireAfterWrite
- 存在问题：
  - get 缓存间隔在 refreshAfterWrite 和 expireAfterWrite 之间，触发缓存异步刷新，此时会获取缓存中的旧值
  - get 缓存间隔大于 expireAfterWrite，针对该 key，获取到锁的线程会同步执行 load，其他未获得锁的线程会阻塞等待，获取锁线程执行延时过长会导致其他线程阻塞时间过长
- 适用场景：
  - 缓存数据量大，限制缓存占用的内存容量
  - 会变，需要刷新缓存
  - 受有限时间缓存中存在旧数据
- 同步加载数据延迟小（使用 redis 等）
- ![image](https://user-images.githubusercontent.com/55612309/113387801-9a32f700-93bf-11eb-9f6d-6ac60b86590c.png)
- 设置 maxSize、refreshAfterWrite、expireAfterWrite

### 迁移方案
- 切换至 Caffeine
  - 在 pom 文件中引入 Caffeine 依赖：
  - <dependency>
  - <groupId>com.github.ben-manes.caffeine</groupId>
  - <artifactId>caffeine</artifactId>
  - </dependency>
  - Caffeine 兼容 Guava API，从 Guava 切换到 Caffeine，仅需要把 CacheBuilder.newBuilder()改成 Caffeine.newBuilder() 即可。
- Get Exception
  - 需要注意的是，在使用 Guava 的 get()方法时，当缓存的 load()方法返回 null 时，会抛出 ExecutionException。切换到 Caffeine 后，get()方法不会抛出异常，但允许返回为 null。
  - Guava 还提供了一个getUnchecked()方法，它不需要我们显示的去捕捉异常，但是一旦 load()方法返回 null时，就会抛出 UncheckedExecutionException。切换到 Caffeine 后，不再提供 getUnchecked()方法，因此需要做好判空处理。

