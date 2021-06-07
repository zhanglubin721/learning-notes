# JUC并发核心原理

![img](image/1454456-20200625122837407-1152312809.png)

## JUC

### 线程管理

- 线程池相关类
  - Executor、Executors、ExecutorService
  - 常用的线程池：FixedThreadPool、CachedThreadPool、ScheduledThreadPool、SingleThreadExecutor
- 能获取子线程的运行结果
  - Callable、Future、FutureTask

### 并发流程管理

- CountDwonLatch、CyclicBarrier、Semaphore、Condition

### 实现线程安全

- 互斥同步(锁)
  - Synchronzied、及工具类Vector、Collections
  - Lock接口的相关类：ReentrantLock、读写锁
- 非互斥同(原子类)
  - 原子基本类型、引用类型、原子升级、累加器
- 并发容器
  - ConcurrentHashMap、CopyOnWriteArrayList、BlockingQueue
- 无同步与不可变方案
  - final关键字、ThreadLocal栈封闭

## 线程池

### 使用线程池的作用好处

- 降低资源消耗
  - 重复利用已创建的线程降低线程创建和销毁造成的消耗
- 提高响应速度
  - 任务到达，可以不需要等到线程创建就能立即执行
- 提高线程的可管理性
  - 使用线程池可以进行统一的分配，调优和监控

### 线程池的参数

- corePoolSize、maximumPoolSize、keepAliveTime、workQueue、threadFactory、handler
- 图示
  - 

### 常用线程池的创建与规则

- 线程添加规则
  - 1.如果线程数量小于corePoolSize，即使工作线程处于空闲状态，也会创建一个新线程来运行新任务，创建方法是使用threadFactory
  - 2.如果线程数量大于corePoolSize但小于maximumPoolSize，则将任务放入队列
  - 3.如果workQueue队列已满，并且线程数量小于maxPoolSize，则开辟一个非核心新线程来运行任务
  - 4.如果队列已满，并且线程数大于或等于maxPoolSize，则拒绝该任务，执行handler
  - 图示(分别与3个参数比较)
    - 
- 常用线程池
  - newFixedThreadPool
    - 创建固定大小的线程池，使用无界队列会发生OOM
  - newSingleThreadExecutor
    - 创建一个单线程的线程池，线程数为1
  - newCachedThreadPool
    - 创建一个可缓存的线程池，60s会回收部分空闲的线程。采用直接交付的队列 SynchronousQueue ，队列容量为0，来一个创建一个线程
  - newScheduledThreadPool
    - 创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求
- 如何设置初始化线程池的大小？
  - 可根据线程池中的线程
    处理任务的不同进行分别估计
    - CPU 密集型任务
      - 大量的运算，无阻塞
        通常 CPU 利用率很高
        应配置尽可能少的线程数量
        设置为 CPU 核数 + 1
    - IO 密集型任务
      - 这类任务有大量 IO 操作
        伴随着大量线程被阻塞
        有利于并行提高CPU利用率
        配置更多数量： CPU 核心数 * 2
- 使用线程池的注意事项
  - 1.避免任务堆积(无界队列会OOM)、2.避免线程数过多(cachePool直接交付队列)、3.排查线程泄露

### 线程池的状态和常用方法

- 线程池的状态
  - RUNNING(接受并处理任务中)、
    SHUTDOWN(不接受新任务但处理排队任务)、
    STOP(不接受新任务 也不处理排队任务 并中断正在进行的任务)、
    TIDYING、TEMINATED(运行完成)
- 线程池停止
  - shutdown
    - 通知有序停止，先前提交的任务务会执行
  - shutdownNow
    - 尝试立即停止，忽略队列里等待的任务

### 线程池的源码解析

- 线程池的组成
  - 1.线程池管理器
    2.工作线程
    3.任务队列：无界、有界、直接交付队列
    4.任务接口Task
- Executor家族
  - Executor顶层接口，只有一个execute方法
  - ExecutorService继承了Executor，增加了一些新的方法，比如shutdown拥有了初步管理线程池的功能方法
  - Executors工具类，来创建，类似Collections
- 线程池实现任务复用的原理
  - 线程池对线程作了包装，不需要启动线程，不需要重复start线程，只是调用已有线程固定数量的线程来跑传进来的任务run方法
  - 添加工作线程
    - 4步：1. 获取线程池状态、4.判断是否进入任务队列 、3.根据状态检测是否增加工作线程、4.执行拒绝handler
  - 重复利用线程执行不同的任务

### 面试题

- 为什么要使用线程池？
- 如何使用线程池？
- 线程池有哪些核心参数？
- 初始化线程池的大小的如何算？
- shutdown 和 shutdownNow 有什么区别？

## ThreadLocal

### ThreadLocal的作用好处

- 为每个线程提供存储自身独立的局部变量，实现线程间隔离
- 即：达到线程安全，不需要加锁节省开销，减少参数传递

### ThreadLocal的使用场景

- 1.每个线程需要一个独享的对象，如 线程不安全的工具类，(线程隔离)
- 2.每个线程内需要保存全局变量，如 拦截器中的用户信息参数，让不同方法直接使用，避免参数传递过多，(局部变量安全，参数传递）

### ThreadLocal的实现原理

- 每个 Thread 维护着一个 ThreadLocalMap 的引用；ThreadLocalMap 是 ThreadLocal 的内部类，用 Entry 来进行存储；key就对应一个个ThreadLocal
- get方法：取出当前线程的ThreadLocalMap，然后调用map.getEntry方法，把ThreadLocal作为key参数传入，取出对应的value
- set方法：往 ThreadLocalMap 设置ThreadLocal对应值
  initalValue方法：延迟加载，get的时候设置初始化
- 图示
  - 

### 缺陷注意

- value内存泄漏
  - 原因：ThreadLocal 被 ThreadLocalMap 中的 entry 的 key 弱引用。如果 ThreadLocal 没有被强引用， 那么 GC 时 Entry 的 key 就会被回收，但是对应的 value 却不会回收，就会造成内存泄漏
  - 解决方案：每次使用完 ThreadLocal，都调用它的 remove () 方法，清除value数据。
  - 源码图示
    - 

### 面试题

- ThreadLocal 的作用是什么？
- 讲一讲ThreadLocal的实现原理(组成结构)
- ThreadLocal有什么风险？

## final与不变性

### 什么是不变性（Immutable）

- 如果对象在被创建后，状态就不能被修改，那么它就是不可变的。
- 具有不变性的对象一定是线程安全的，我们不需要对其采取任何额外的安全措施，也能保证线程安全。

### final的作用

- 类防止被继承、方法防止被重写、变量防止被修改
- 天生是线程安全的(因为不能修改)，而不需要额外的同步开销

### final的3种用法：修饰变量、方法、类

- final修饰变量
  - 被final修饰的变量，意味着值不能被修改。
    如果变量是对象，那么对象的引用不能变，但是对象自身的内容依然可以变化。
  - 赋值时机
    - 属性被声明为final后，该变量则只能被赋值一次。且一旦被赋值，final的变量就不能再被改变，如论如何也不会变。
    - 区分为3种
      - final instance variable（类中的final属性）
        - 等号右侧、构造函数、初始化代码块
      - final static variable（类中的static final属性）
        - 等号右侧、静态初始化代码块
      - final local variable（方法中的final变量）
        - 使用前复制即可
    - 为什么规定时机
      - 根据JVM对类和成员变量、静态成员变量的加载规则来看：如果初始化不赋值，后续赋值，就是从null变成新的赋值，这就违反final不变的原则了！
- final修饰方法（构造方法除外）
  - 不可被重写，也就是不能被override，即便是子类有同样名字的方法，那也不是override，与static类似*
- final修饰类
  - 不可被继承，例如典型的String类就是final的

### 栈封闭 实现线程安全

- 在方法里新建的局部便咯，实际上是存储在每个线程私有的栈空间，线程栈不能被其它线程访问，所以不会有线程安全问题，如ThreadLocal

## CAS

### 什么是CAS

- 我认为V的值应该是A，如果是的话那我就把它改成B，如果不是A（说明被别人修改过了），那我就不修改了，避免多人同时修改导致出错。
- CAS有三个操作数：内存值V、预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，才将内存值修改为B，否则什么都不做。最后返回现在的V值。
- 最终执行CPU处理机提供的的原子指令

### 缺点

- ABA问题
  - 我认为 V的值为A，有其它线程在这期间修改了值为B，但它又修改成了A，那么CAS只是对比最终结果和预期值，就检测不出是否修改过
- CAS+自旋，导致自旋时间过长
- 改进：通过版本号的机制来解决。每次变量更新的时候，版本号加 1，如AtomicStampedReference的compareAndSet ()

### 应用场景

- 1 乐观锁：数据库、git版本号； 自旋 2 concurrentHashMap：CAS+自旋
  3 原子类

### CAS底层实现

- 通过Unsafe获取待修改变量的内存递增，
  比较预期值与结果，调用汇编cmpxchg指令

### 以AtomicInteger为例，分析在Java中是如何利用CAS实现原子操作的？

- 1.使用Unsafe类拿到value的内存递增，通过偏移量 直接操作内存数据
- 2.Unsafe的getAndAddInt方法，使用CAS+自旋尝试修改数据
- CAS的参数通过 预期值 与 实际拿到的值进行比较，相同就修改，不相同就自旋
- Unsafe提供硬件级别的原子操作，最终调用原子汇编指令的cmpxchg指令

## 原子类atomic包

### 原子类的作用

- 原子类的作用和锁类似，都是为了保证并发下线程安全
- 粒度更细，变量级别
- 效率更高，除了高度竞争外

### 原子类的种类

- Atomic*基本类型原子类：AtomicInteger、AtomicLong、AtomicBoolean
- Atomic*Array数组类型原子类：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray
- Atomic*Reference 引用类型原子类：AtomicReference等
- AtomicIntegerFiledUpdate等升级类型原子类
- Adder累加器、Accumlator累加器

### AtomicInteger

- 常用方法
  - get、getAndSet、getAndIncrement、compareAndSet(int expect,int update)
- 实现原理
  - AtomicInteger 内部使用 CAS 原子语义来处理加减等操作。CAS通过判断内存某个位置的值是否与预期值相等，如果相等则进行值更新
  - CAS 是内部是通过 Unsafe 类实现，而 Unsafe 类的方法都是 native 的，在 JNI 里是借助于一个 CPU 指令完成的，属于原子操作。
- 缺点
  - 循环开销大。如果 CAS 失败，会一直尝试
  - 只能保证单个共享变量的原子操作，对于多个共享变量，CAS 无法保证，引出原子引用类
  - 用CAS存在 ABA 问题

### Adder累加器

- 引入目的/改进思想
  - AtomicLong在每一次加法都要flush和refresh主存，与JMM内存模型有关。工作线程之间不能直接通信，需要通过主内存间接通信
- 设计思想
  - Java8引入，高并发下LongAdder比AtomicLong效率高，本质是空间换时间
  - 竞争激烈时，LongAdder把不同线程对应到不同的Cell单元上进行修改，降低了冲突的概率，是多段锁的理念，提高了并发性
  - 每个线程都有自己的一个计数器，不存在竞争
  - sum源码分析：最终把每一个Cell的计数器与base主变量相加

### 面试题

- AtomicInteger 怎么实现原子操作的？
- AtomicInteger 有哪些缺点？

## 并发容器

### ConcurrentHashMap

- 集合类历史
  - Vector的方法被synchronizd修饰，同步锁；不允许多个线程同时执行。并发量大的时候性能不好
  - Hashtable是线程安全的HashMap，方法也是被synchronized修饰，同步但并发性能差
  - Collections工具类，提高的有synchronizedList和synchronizedMap，代码内使用sync互斥变量加锁
- 为什么需要
  - 为什么不用HashMap
    - 1.多线程下同时put碰撞导致数据丢失
    - 2.多线程下同时put扩容导致数据丢失
    - 3.死循环造成的CPU100%
  - 为什么不用Collection.synchronizedMap
    - 同步锁并发性能低
- 数据结构与并发策略
  - JDK1.7
    - 数组+链表，拉链法解决冲突
    - 采用分段锁，每个数组结点是一个独立的ReentrantLock锁，可以支持同时并发写
  - JDK1.8
    - 数组+链表+红黑树，拉链法和树化解决冲突
    - 采用CAS+synchronized锁细化
  - 1.7到1.8改变后有哪些优点
    - 1.数据结构由链表变为红黑树，树查询效率更高
    - 2.减少了Hash碰撞，1.7拉链法
    - 3.保证了并发安全和性能，分段锁改成CAS+synchronized
    - 为什么超过8要转为红黑树，因为红黑树存储空间是结点的两倍，经过泊松分布，8冲突概率低
- 注意事项
  - 组合操作线程不安全，应使用putIfAbsent提供的原子性操作

### CopyOnWriteArrayList

- 引入目的
  - Vector和SynchronizedList锁的粒度太大并发效率低，并且迭代时无法编辑exceptMod!=Count
- 适合场景
  - 读多写少，如黑名单管理每日更新
- 读写规则
  - 是对读写锁的升级：读取完全不用加锁，读时写入也不会阻塞。只有写入和写入之间需要同步
- 实现原理
  - 创建数据的新副本，实现读写分离，修改时整个副本进行一次复制，完成后最后再替换回去；由于读写分离，旧容器不变，所以线程安全无需锁
  - 在计算机内存中修改不直接修改主内存，而是修改缓存(cache、对拷贝的副本进行修改)，再进行同步(指针指向新数据)。
- 缺点
  - 1.数据一致性问题，拷贝不能保证数据实时一致，只能保证数据最终一致性
  - 2.内存占用问题，写复制机制，写操作时内存会同时驻扎两个对象的内存

### 并发队列

- 为什么使用队列
  - 用队列可以在线程间传递数据，缓存数据
  - 考虑锁等线程安全问题的重任转移到了“队列”上
- 并发队列关系图示
  - 
- BlockingQueue阻塞队列
  - 阻塞队列是局由自动阻塞功能的队列，线程安全；take方法移除队头，若队列无数据则阻塞直到有数据；put方法插入元素，如果队列已满就无法继续插入则阻塞直到队列里有了空闲空间
  - ArrayBlockQueue
    - 有界可指定容量、可公平
    - Put源码加锁，可中断的上锁方法。没满才可以入队，否则一直await等待。
  - LinkedBlockingQueue
    - 无界容量为MAX_VALUE，内部结构Node
    - 使用了两把锁take锁和put锁互补干扰
  - PriorityBlockingQueue
    - 支持优先级，无界队列
  - SynchronousQueue
    - 直接传递的队列，容量0，效率高线程池的CacheExecutorPool使用其作为工作队列
  - DelayQueue
    - 无界队列，根据延迟时间排序
- 非阻塞队列
  - ConcurrentLinkedQueue
    - 使用链表作为队列存储结构
    - 使用Unsafe的CAS非阻塞方法来实现线程安全，无需阻塞，适合对性能要求较高的并发场景
- 选择合适的队列
  - 边界上看
    - ArrayBlockQueue有界；LinkedBlockQueue无界适合容量大容量激增
  - 内存上看
    - ArrayBlockQueue内部结构是array，从内存存储上看，连续存储更加整齐。而LinkedBlockQueue采用链表结点，可以非连续存储。
  - 吞吐量上看
    - 从性能上看LinkedBlockQueue的put锁和锁分开，锁粒度更细，所以优于ArrayBlockQueue

### 总结并发容器对比

- 分为3类：Concurrent*、CopyOnWrite*、Blocking*
- Concurrent*的特定是大部分使用CAS并发；而CopyOnWrite通过复制一份元数据写加锁实现；Blocking通过ReentLock锁底层AQS实现

## 并发流程控制工具类

### 控制并发流程工具类的作用

- 控制并发流程的工具类，作用是帮助程序员更容易让线程之间相互配合，来满足业务逻辑
- 并发工具类图示
  - 

### CountDownLatch倒计时门闩

- 作用(事件)
  - 一个线程等多个线程、或多个线程等一个线程完成到达，才能继续执行
- 常用方法
  - 构造函数中传入倒数值、await、countDown

### Semaphore信号量

- 作用
  - 用来限制管理数量有限的资源的使用情况，相当于一定数量的“许可证”
- 常用方法
  - 构造函数中传入数量、acquire、release

### Condition条件对象

- 作用
  - 等待条件满足才放行，否则阻塞；一个锁可以对应多个条件
- 常用方法
  - lock.newCondition、await、signal

### CyclicBarrier循环栅栏

- 作用(线程)
  - 多个线程互相等待，直到达到同一个同步点（屏障），再继续一起执行
- 常用方法
  - 构造函数中传入个数、await

## AQS

### AQS的作用

- AQS是一个用于构建锁、同步器、协作工具类的框架，有了AQS后，更多的协作工具类都可以很方便的写出来

### AQS的应用场景

- Exclusive(独占)
  - ReentrantLock 公平和非公平锁
- Share(共享)
  - Semaphore/CountDownLatch/CyclicBarrier

### AQS原理解析

- 核心三要素
  - 1.sate
    - 使用一个 int 成员变量来表示同步状态 state，被volatile修饰，会被并发修改，各方法如getState、setState等使用CAS保证线程安全
    - 在ReentrantLock中，表示可重入的次数
    - 在Semaphore中，表示剩余许可证信号的数量
    - 在CountDownLatch中，表示还需要倒数的个数
  - 2.控制线程抢锁和配合的FIFO队列
    - 获取资源线程的排队工作
  - 3.期望协作工具类去实现的“获取/释放”等唤醒分配的方法策略
- AQS的用法
  - 第一步：写一个类，想好协作的逻辑，实现获取/释放方法
  - 第二步：内部写一个Sync类继承AbstractQueueSynchronizer
  - 第三步：Sync类根据独占还是共享重写tryAcquire/tryRelease或tryAcquireShared和tryReleaseShared等方法，在之前写的获取/释放方法中调用AQS的acquire/release或则Shared方法

### AQS应用实例源码解析

- AQS在CountDownLatch的应用
  - 内部类Sync继承AQS
  - 1.state表示门闩倒数的count数量，对应getCount方法获取
  - 2.释放方法，countDown方法会让state减1，直到减为0时就唤醒所有线程。countDown方法调用releaseShared，它调用sync实现的tryReleaseShared，其使用CAS+自旋锁，来实现安全的计数-1
  - 3.阻塞方法，await会调用sync提供的aquireSharedInterruptly方法，当state不等于0时，最终调用LockUpport的park，它利用Unsafe的park，native方法，把线程加入阻塞队列
  - 总结
    - 
- AQS在Semphore的应用
  - state表示信号量允许的剩余许可数量
  - tryAcquire方法，判断信号量大于0就成功获取，使用CAS+自旋改变state状态。如果信号量小于0了，再请求时tryAcquireShared返回负数，调用aquireSharedInterruptly方法就进入阻塞队列
  - release方法，调用sync实现的releaseShared，会利用AQS去阻塞队列唤醒一个线程
  - 总结
    - 
- AQS在ReentrantLock的应用
  - state表示已重入的次数，独占锁权保存在AQS的Thread类型的exclusiveOwnerThread变量中
  - 释放锁: unlock方法调用sync实现的release方法，会调用tryRelease，使用setState而不是CAS来修改重入次数state，当state减到0完全释放锁
  - 加锁lock方法：调用sync实现的lock方法。CAS尝试修改锁的所有权为当前线程，如果修改失败就要调用acquire方法再次尝试获取，acquire方法调用了AQS的tryAcquire，这个实现在ReentantLock的里面，获取失败加入到阻塞队列

### 通过AQS自定义同步器

- 自定义同步器在实现时只需要根据业务逻辑需求，实现共享资源 state 的获取与释放方式策略即可
- 至于具体线程等待队列的维护（如获取资源失败入队 / 唤醒出队等），AQS 已经在顶层实现好了