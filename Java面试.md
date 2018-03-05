##STAR：“STAR”是SITUATION（背景）、TASK（任务）、ACTION（行动）和RESULT（结果）


##基础
###JVM：
1. JVM内存模型：JVM内存主要分为以下五个区域：线程栈、Java堆、方法区、程序计数器、Native方法栈；
  3. 线程栈：用于存放（帧栈）程序指令和局部变量表
    4. 每个方法调用对应一个帧栈；
    5. 局部变量表包含各种基本类型：boolean、byte、char、short、int、float、long、double以及对象的引用。
    6. 栈的大小：受jvm参数：-XSS的影响，64bit机器默认为1024K；栈的大小影响到了线程的最大数量，尤其在大流量的server中，我们很多时候的并发数受到的是线程数的限制
  4. Java堆：Java内存最大的一块，所有对象实例、数组都存放在Java堆，线程共享;
    5. 首先堆可以划分为新生代(Young Generation)和老年代(Old Generation):
    6. 然后新生代又可以划分为一个Eden区和两个Survivor（幸存）区。
    7. 按照规定，新对象会首先分配在Eden中（如果对象过大，比如大数组，将会直接放到老年代）。在GC中，Eden中的对象会被移动到survivor中，直至对象满足一定的年纪（定义为熬过minor GC的次数），会被移动到老年代。新生代 ( Young ) 与老年代 ( Old ) 的比例的值为 1:2 ( 该值可以通过参数 –XX:NewRatio 来指定 )
默认的，Eden : from : to = 8 : 1 : 1 ( 可以通过参数 –XX:SurvivorRatio 来设定 )，即： Eden = 8/10 的新生代空间大小，from = to = 1/10 的新生代空间大小。
  5. 静态方法区：又称为永久代（Perm Generation）。它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。各线程共享；
  3. Native方法栈：和虚拟栈相似，只不过它服务于Native方法，同样线程私有
  2. 程序计数器：当前线程所执行的字节码的指示器，用于记录下一条要运行的指令，线程私有
6. 垃圾回收机制和常见算法：
  7. 垃圾回收发生在Java的堆区，Java堆分为Young Generation和Old Generation，通常来说比例为1：2；Young Generation又分为Eden区和Survivor区，所有新实例化的对象都存放在Eden区，Eden和Survivor的比例一般为4：1；Survivor区又分为两个子区，用于将对象来回复制，这样，在Survivor的对象会在两个区中来回经历GC，达到一定年龄后会被移到老年代。因为这个对象多次垃圾回收依然存活，表明这个对象比较稳定，此后在老年代经历垃圾回收的频率非常低。
  8. 如果一个新的对象太大，以至于新生代经过一次垃圾回收后依然没有足够空间存放它。JVM会通过分配担保来把这个对象放在老年代。如果老年代空间不够，经过一次Full GC还是没有空间，那虚拟机无法为这个对象创建内存空间，只能抛出OOM异常停止运行。
  9. GC前需要进行对象的存活判断，主要有以下两种：
    1. 引用计数：每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象相互循环引用的问题，形成内存孤岛。
    2. 可达性分析（Reachability Analysis）：从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。GC root包含：1)虚拟机栈中引用的对象、2)方法区中类静态属性实体引用的对象、3)方法区中常量引用的对象、4)本地方法栈中JNI引用的对象。
</div>
  10. 主要算法：
    11. 标记 -清除算法：
“标记-清除”（Mark-Sweep）算法，如它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收掉所有被标记的对象。之所以说它是最基础的收集算法，是因为后续的收集算法都是基于这种思路并对其缺点进行改进而得到的。
它的主要缺点有两个：一个是效率问题，标记和清除过程的效率都不高；另外一个是空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致，当程序在以后的运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。
    10. 复制算法：
“复制”（Copying）的收集算法，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。这样使得每次都是对其中的一块进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。缺点是这种算法的代价是将内存缩小为原来的一半，持续复制长生存期的对象则导致效率降低。
    11. 分代回收算法：GC分代的基本假设：绝大部分对象的生命周期都非常短暂，存活时间短。算法把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用“标记-清理”或“标记-整理”算法来进行回收。

2. Java内存堆和栈区别
  1. 栈内存用来存储基本类型的变量和对象的引用变量，堆内存用来存储Java中的对象，无论是成员变量，局部变量，还是类变量，它们指向的对象都存储在堆内存中
  2. 栈内存归属于单个线程，每个线程都会有一个栈内存，其存储的变量只能在其所属线程中可见，即栈内存可以理解成线程的私有内存，堆内存中的对象对所有线程可见。堆内存中的对象可以被所有线程访问
  3. 如果栈内存没有可用的空间存储方法调用和局部变量，JVM会抛出java.lang.StackOverFlowError，如果是堆内存没有可用的空间存储生成的对象，JVM会抛出java.lang.OutOfMemoryError
  4. 栈的内存要远远小于堆内存，如果你使用递归的话，那么你的栈很快就会充满，-Xss选项设置栈内存的大小。-Xms选项可以设置堆的开始时的大小
5. Class Loader的种类和双亲委派机制：http://tracylihui.github.io/2015/09/09/java/Java%E7%9A%84ClassLoader%E5%8E%9F%E7%90%86/
  6. Java程序在启动的时候，并不会一次性加载程序所要用的所有class文件，而是根据程序的需要，通过Java的类加载机制（ClassLoader）来动态加载某个class文件到内存当中的，从而只有class文件被载入到了内存之后，才能被其它class所引用。所以ClassLoader就是用来动态加载class文件到内存当中用的；
  7. Java默认提供的三个ClassLoader：
    8. BootStrap ClassLoader：称为启动类加载器，是Java类加载层次中最顶层的类加载器，负责加载JDK中的核心类库，如：rt.jar、resources.jar、charsets.jar等；
    9. Extension ClassLoader：称为扩展类加载器，负责加载Java的扩展类库，默认加载JAVA_HOME/jre/lib/ext/目下的所有jar；
    10. App ClassLoader：称为系统类加载器，负责加载应用程序classpath目录下的所有jar和class文件，或者-Dclasspath指定的目录下的所有jar和class文件；
    11. 除了Java默认提供的三个ClassLoader之外，用户还可以根据需要定义自已的ClassLoader，而这些自定义的ClassLoader都必须继承自java.lang.ClassLoader类，也包括Java提供的另外二个ClassLoader（Extension ClassLoader和App ClassLoader）在内，但是Bootstrap ClassLoader不继承自ClassLoader，因为它不是一个普通的Java类，底层由C++编写，已嵌入到了JVM内核当中，当JVM启动后，Bootstrap ClassLoader也随着启动，负责加载完核心类库后，并构造Extension ClassLoader和App ClassLoader类加载器。
  11. 双亲委派机制：
每个ClassLoader实例都有一个父类加载器的引用（作为Java对象的成员存在），虚拟机内置的类加载器（Bootstrap ClassLoader）本身没有父类加载器，但可以用作其他ClassLoader实例的父类加载器。
当一个ClassLoader实例需要加载某个类时，它会试图亲自搜索某个类之前，先把这个任务委托给它的父类加载器，这个过程是由上至下依次检查的，首先由最顶层的类加载器Bootstrap ClassLoader试图加载，如果没加载到，则把任务转交给Extension ClassLoader试图加载，如果也没有加载到，则转交给App ClassLoader进行加载，如果它也没有加载得到的话，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。如果它们都没有加载到这个类时，则抛出ClassNotFoundException异常。反之，则将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的Class实例对象。

    12. 双亲委派机制可以避免重复加载：当父类已经加载了该类的时候，就没有必要子ClassLoader再加载一次；
    13. JVM在判定两个class是否相同时，不仅要判断两个类名是否相同，而且要判断是否由同一个类加载器实例加载的。只有两者同时满足的情况下，JVM才认为这两个class是相同的。
14. Java8专题
  15. Metaspace vs PermGen:https://stackoverflow.com/questions/27131165/what-is-the-difference-between-permgen-and-metaspace
      16. Metaspace is in native memory while PermGen is in Java heap.
      17. Metaspace will perform GarbageCollection once class metadata reached MaxMetaspace setting.
      18. Metaspace will auto-resize when needed.
  19. http://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html

###并发和多线程
1. ThreadLocal和线程同步解决的问题是什么；
  2. ThreadLocal通过为每个线程提供一个独立的变量副本解决了变量并发访问的冲突问题。对于多线程资源共享的问题，同步机制采用了“以时间换空间”的方式，而ThreadLocal采用了“以空间换时间”的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。
  3. ThreadLocal并不能替代同步机制，两者面向的问题领域不同。
    1. 同步机制是为了同步多个线程对相同资源的并发访问，是为了多个线程之间进行通信的有效方式；
    2. 而threadLocal是隔离多个线程的数据共享，从根本上就不在多个线程之间共享变量，这样当然不需要对多个线程进行同步了。
2. ReentrantLock和synchronized的区别：http://zhwbqd.github.io/2015/02/13/lock-in-java.html
  3. synchronized机制提供了对与每个对象相关的隐式监视器锁的访问, 并强制所有锁获取和释放均要出现在一个块结构中, 当获取了多个锁时, 它们必须以相反的顺序释放. synchronized机制对锁的释放是隐式的, 只要线程运行的代码超出了synchronized语句块范围, 锁就会被释放；synchronized是在JVM层面实现的；
  4. 而Lock机制必须显式的调用Lock对象的lock/unlock()方法才能获取锁/释放锁, 这为获取锁和释放锁不出现在同一个块结构中, 以及以更自由的顺序释放锁提供了可能；Lock是在Java代码级别实现的；
  5. 不同点：
    1. ReentrantLock多了锁投票，定时锁等候，中断锁等候；synchronized锁不能被打断；
    1. synchronized是在JVM层面上实现的，不但可以通过一些监控工具监控synchronized的锁定，而且在代码执行时出现异常，JVM会自动释放锁定；但是使用Lock则不行，lock是通过代码实现的，要保证锁定一定会被释放，就必须将unLock()放到finally{}中；
    1. 在资源竞争不是很激烈的情况下，Synchronized的性能要优于ReetrantLock，但是在资源竞争很激烈的情况下，Synchronized的性能会下降几十倍；但是ReetrantLock的性能能维持常态；
    1. 线程A和B都要获取对象O的锁定，假设A获取了对象O锁，B将等待A释放对O的锁定：使用 synchronized，如果A不释放，B将一直等下去，不能被中断；使用ReentrantLock，如果A不释放，可以使B在等待了足够长的时间以后，中断等待，而干别的事情
  1. ReentrantLock获取锁的四种方式：
    1. lock()：如果获取了锁立即返回，如果别的线程持有锁，当前线程则一直处于休眠状态，直到获取锁
    1. tryLock()：如果获取了锁立即返回true，如果别的线程正持有锁，立即返回false；
    1. tryLock(long timeout,TimeUnit unit)：如果获取了锁定立即返回true，如果别的线程正持有锁，会等待参数给定的时间，在等待的过程中，如果获取了锁定，就返回true，如果等待超时，返回false；
    1. lockInterruptibly()：如果获取了锁定立即返回，如果没有获取锁定，当前线程处于休眠状态，直到或者锁定，或者当前线程被别的线程中断。

3. Java Thread的状态及状态间的互相转换；
http://www.jianshu.com/p/dbbcceb6bc2a
![1401055-d5d061d01194e437.gif](file:///Users/tangp/Pictures/1401055-d5d061d01194e437.gif)
4. Thread.join()函数：Waits for this thread to die，既将当前线程休眠，直到被调用的线程实例执行完成；
4. ExecutorService的工作原理:http://wiki.jikexueyuan.com/project/java-concurrency/executor.html
5. 自旋锁：http://blog.onlycatch.com/post/%E8%87%AA%E6%97%8B%E9%94%81
  6. 自旋锁（SpinLock）：是指当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环；获取锁的线程一直处于活跃状态，但是并没有执行任何有效的任务，使用这种锁会造成busy-waiting。
<pre>
    public class SpinLock {
        private AtomicReference<Thread> cas = new AtomicReference<Thread>();

        public void lock() {
            Thread current = Thread.currentThread();
            // 利用CAS
            while (!cas.compareAndSet(null, current)) {
                // DO nothing
            }
        }

        public void unlock() {
            Thread current = Thread.currentThread();
            cas.compareAndSet(current, null);
        }
    }
</pre>
6. ConcurrentHashMap：https://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/index.html
  7. 用分离锁实现多个线程间的并发写操作：ConcurrentHashMap包含int concurrencyLevel个Segment，一个Segment是一组HashEntry元素组成的Table，利用多Segment分离锁实现多个线程之间的并发写操作；
  8. 根据HashEntry的 hash 值找到对应的 Segment，在这个 Segment 中执行具体的 put 操作（会加锁）；锁定某个 Segment 对象而非整个 ConcurrentHashMap因此整个ConcurrentHashMap的并发性能得到了提高；
  9. HashEntry 类的 value 域被声明为 Volatile 型，由于对 Volatile 变量的写入操作将与随后对这个变量的读操作进行同步，当一个写线程修改了某个 HashEntry 的 value 域后，另一个读线程读这个值域，Java 内存模型能够保证读线程读取的一定是更新后的值。
  10. ConcurrentHashMap 的高并发性主要来自于三个方面：
    11. 用分离锁实现多个线程间的更深层次的共享访问。
    12. 用 HashEntery 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。
    13. 通过对同一个 Volatile 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性。

###Java语法和基础数据类型
1. final/finally关键字的含义；
2. abstract类里面可以出现final修饰的方法吗？
3. HashMap的实现：https://tech.meituan.com/java-hashmap.html
  4. 注意Hash算法、负载因子以及扩容等知识点；
  5. 经过观测可以发现，HashMap使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。
3. ArrayList和LinkedList的区别；
  4. ArrayList：数据结构是一个数组，当size大于capacity的时候会扩容为原来capacity的两倍，产生复制元素开销；提供O(1)操作复杂度；
  1. LinkedList：数据结构是一个双向链表，不需要扩容，适用于频繁插入的场景，但如果调用set(int,Object value)的时候由于需要遍历指针，因此性能较慢；最坏提供O(N)操作复杂度；
2. Object类的基础方法：
  3. getClass()；
  4. hashCode()；
  4. equals()；
  5. clone();
  6. toString();
  7. notify():通知(唤醒)正在等待当前实例的线程，如果有多个线程在等待，则随机选取一个；
  8. notifyAll()：同上，只不过会唤醒所有正在等待的线程；
  9. wait(long timeout)：将当前线程休眠并等待别的线程唤醒，timeout==0时，将不超时，一直等待，否则等待timeout后恢复线程执行；

###设计模式
单例模型、工厂模式、观察者模式、适配器、委托模式；

###Spring框架
1. Spring AOP的实现原理：https://www.ibm.com/developerworks/cn/java/j-lo-springaopcglib/index.html
  2. Spring AOP 采用运行时生成 AOP 代理类，无需使用特定编译器进行处理。由于 Spring AOP 需要在每次运行时生成 AOP 代理，因此性能略差一些；
3. 静态代理和动态代理的区别：http://layznet.iteye.com/blog/1182924
  4. 静态代理采取编译时增强的方法，既为目标类生成代理类的方式来达成目的，需要手动写一个代理类或者使用额外的编译工具自动生成代理类；优点是性能好，确定是如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度；
  5. 动态代理：动态代理类的源码是在程序运行期间由JVM根据反射等机制动态的生成，所以不存在代理类的字节码文件。Java 动态代理机制生成的所有动态代理类的父类是java.lang.reflect.Proxy，Proxy 静态方法生成动态代理类同样需要通过类装载器来进行装载才能使用，它与普通类的唯一区别就是其字节码是由 JVM 在运行时动态生成的而非预存在于任何一个 .class 文件中。
  6. 动态代理的特点： 
    7. 包：如果所代理的接口都是 public 的，那么它将被定义在顶层包（即包路径为空），如果所代理的接口中有非 public 的接口（因为接口不能被定义为 protect 或 private，所以除 public 之外就是默认的 package 访问级别），那么它将被定义在该接口所在包（假设代理了 com.ibm.developerworks 包中的某非 public 接口 A，那么新生成的代理类所在的包就是 com.ibm.developerworks），这样设计的目的是为了最大程度的保证动态代理类不会因为包管理的问题而无法被成功定义并访问；
    8. 类修饰符：该代理类具有 final 和 public 修饰符，意味着它可以被所有的类访问，但是不能被再度继承；
    9. 类名：格式是“$ProxyN”，其中 N 是一个逐一递增的阿拉伯数字，代表 Proxy 类第 N 次生成的动态代理类，值得注意的一点是，并不是每次调用 Proxy 的静态方法创建动态代理类都会使得 N 值增加，原因是如果对同一组接口（包括接口排列的顺序相同）试图重复创建动态代理类，它会很聪明地返回先前已经创建好的代理类的类对象，而不会再尝试去创建一个全新的代理类，这样可以节省不必要的代码重复生成，提高了代理类的创建效率。
    10. 类继承关系：动态代理类是java.lang.reflect.Proxy的子类，同时实现了所有其所代理的一组接口；
  11. 动态代理的优缺点：
    12. 动态代理与静态代理相比较，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。
    13. 由于动态代理类继承至Proxy类，因此它仅支持 interface 代理；
2. Spring MVC的原理：
  3. http://www.jianshu.com/p/74794cc5b72b
  4. ![Spring MVC.png](file:///Users/tangp/Pictures/Spring MVC.png)
  5. Spring MVC是通过将需要Spring MVC处理的请求映射到一个名叫DispatcherServlet的servlet上实现的。客户端请求首先会交给DispatcherServlet，DispatcherServlet会通过HandlerMapping去查找当前请求的URL对应的那个Handler（通常是Controller中对应的一个方法）。
  6. DispatcherServlet会将请求交给第2步找到的那个Handler方法执行
执行的过程可能会调用若干的Service来完成业务的处理
  6. 最后在这个Handler中将处理的结果封装成未ModelAndView对象返回给DispatcherServlet。ModelAndView是模型和视图的封装对象。
  6. DispatcherServlet根据ModelAndView中的View，去ViewResolver（视图解析器）中找到对应的视图。
  6. DispatcherServlet将ModelAndView中的Model交给第6步中找到的那个View（JSP,JSTL...）进行视图的渲染。
  6. 渲染后，将视图转为HTTP响应流返回给客户端。

3. BeanFactory和FactoryBean：
  4. BeanFactory：是一个接口；定义了 IOC 容器的最基本形式，并提供了 IOC 容器应遵守的的最基本的接口；
  5. FactoryBean：Spring 提供了一个 org.springframework.bean.factory.FactoryBean 的工厂类接口，用户可以通过实现该接口定制实例化某个Bean的逻辑；


###算法和数据结构
1. 平衡二叉树：又称 AVL 树。它或者是一棵空树,或者是具有如下性质的二叉树：
  1. 它的左子树和右子树都是平衡二叉树，
  1. 左子树和右子树的深度之差的绝对值不超过1。
2. 哈希表：
  3. 哈希函数：也叫散列函数，它对不同的输出值得到一个固定长度的消息摘要。理想的哈希函数对于不同的输入应该产生不同的结构，同时散列结果应当具有同一性（输出值尽量均匀）和雪崩效应（微小的输入值变化使得输出值发生巨大的变化）。
  4. 冲突解决：现实中的哈希函数不是完美的，当两个不同的输入值对应一个输出值时，就会产生“碰撞”，这个时候便需要解决冲突。常见的冲突解决方法有开放定址法，链地址法，建立公共溢出区等。实际的哈希表实现中，使用最多的是链地址法；
5. 排序算法：
  6. 稳定排序算法：
    7. 冒泡排序（Bubble Sort） — O(n²)
    8. 插入排序（Insertion Sort）— O(n²)
    9. 桶排序（Bucket Sort）— O(n); 需要 O(k) 额外空间
    9. 计数排序 (Counting Sort) — O(n+k); 需要 O(n+k) 额外空间
    9. 合并排序（Merge Sort）— O(nlogn); 需要 O(n) 额外空间
    9. 二叉排序树排序 （Binary tree sort） — O(n log n) 期望时间;O(n²)最坏时间; 需要 O(n) 额外空间
    9. 基数排序（Radix sort）— O(n·k); 需要 O(n) 额外空间
  7. 不稳定算法：
    9. 堆排序（Heapsort）— O(nlogn)
    9. 快速排序（Quicksort）— O(nlogn) 期望时间, O(n²) 最坏情况; 对于大的、乱数串行一般相信是最快的已知排序

###DataBase
1. 数据库的索引文件数据结构：https://www.jianshu.com/p/1775b4ff123a
5. 事务的特性（ACID）
  1. atomacity 原子性：事务必须是原子工作单元；对于其数据修改，要么全都执行，要么全都不执行。通常，与某个事务关联的操作具有共同的目标，并且是相互依赖的。如果系统只执行这些操作的一个子集，则可能会破坏事务的总体目标。原子性消除了系统处理操作子集的可能性。
  2. consistency 一致性：事务将数据库从一种一致状态转变为下一种一致状态。也就是说，事务在完成时，必须使所有的数据都保持一致状态（各种 constraint 不被破坏）。
  3. isolation 隔离性：由并发事务所作的修改必须与任何其它并发事务所作的修改隔离。事务查看数据时数据所处的状态，要么是另一并发事务修改它之前的状态，要么是另一事务修改它之后的状态，事务不会查看中间状态的数据。换句话说，一个事务的影响在该事务提交前对其他事务都不可见。
  4. durability 持久性：事务完成之后，它对于系统的影响是永久性的。该修改即使出现致命的系统故障也将一直保持。
5. 事务的隔离级别
  6. 脏读(Dirty Read)当一个事务读取另一个事务尚未提交的修改时，产生脏读。
  7. 非重复读(Nonrepeatable Read) 一个事务对同一行数据重复读取两次，但是却得到了不同的结果。同一查询在同一事务中多次进行，由于其他提交事务所做的修改或删除，每次返回不同的结果集，此时发生非重复读。
  8. 幻像读(Phantom Reads) 事务在操作过程中进行两次查询，第二次查询的结果包含了第一次查询中未出现的数据（这里并不要求两次查询的SQL语句相同）。这是因为在两次查询过程中有另外一个事务插入数据造成的。
  9. 丢失修改(Lost Update)
    10. 第一类：当两个事务更新相同的数据源，如果第一个事务被提交，第二个却被撤销，那么连同第一个事务做的更新也被撤销。
    11. 第二类：有两个并发事务同时读取同一行数据，然后其中一个对它进行修改提交，而另一个也进行了修改提交。这就会造成第一次写操作失效。
12. 数据库4个隔离级别：
  13. 未提交读(Read Uncommitted)：事务可以看到其他事务尚未提交的更改。这些称为脏读。
  14. 已提交读(Read Committed)：
  15. 可重复读(Repeatable Read)；
  16. 串行读(Serializable). [Read more](http://www.jianshu.com/p/4e3edbedb9a8)；[Read more from IBM](https://www.ibm.com/support/knowledgecenter/zh/SSZJPZ_9.1.0/com.ibm.swg.im.iis.conn.drs.doc/topics/DRS040.html)
14. 隔离机制的实现：锁
  15. 共享锁(S锁)：用于只读操作(SELECT)，锁定共享的资源。共享锁不会阻止其他用户读，但是阻止其他的用户写和修改。
  16. 更新锁(U锁)：用于可更新的资源中。防止当多个会话在读取、锁定以及随后可能进行的资源更新时发生常见形式的死锁。
  17. 独占锁(X锁，也叫排他锁)：一次只能有一个独占锁用在一个资源上，并且阻止其他所有的锁包括共享缩。写是独占锁，可以有效的防止“脏读”。
18. 乐观锁和悲观锁
  19. 乐观锁（Optimistic Lock），每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在提交更新的时候会判断一下在此期间别人有没有去更新这个数据。乐观锁适用于读多写少的应用场景，这样可以提高吞吐量。使用数据版本（Version）记录机制实现。
  20. 悲观锁：每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，所以一开始就对记录加上锁（SELECT ... FOR UPDATE）。
21. 数据库主从复制：
  22. 基于语句的复制：主服务器上面执行的语句在从服务器上面再执行一遍。存在的问题：时间上可能不完全同步造成偏差，执行语句的用户也可能是不同一个用户。
  23. 基于行的复制：把主服务器上面改编后的内容直接复制过去，而不关心到底改变该内容是由哪条语句引发的。存在的问题：比如一个工资表中有一万个用户，我们把每个用户的工资+1000，那么基于行的复制则要复制一万行的内容，由此造成的开销比较大，而基于语句的复制仅仅一条语句就可以了。
24. 如何解决主从数据库同步延迟问题？
  25. 主从架构是一种用于数据容错的<b>高可用性</b>解决方案，而不是一种处理<b>高并发压力</b>的解决方案。它的目的是主机当机以后备机可以马上顶上，而不是让备机来分担并发压力。
  26. 这取决于应用场景。对于一般的互联网应用，并发压力大但不要求支持事务，可以考虑分布式缓存。对于银行这样严格要求强一致性的应用，对于写入延迟一般没什么要求，数据不出错就行，可以适用完全同步的模式。
  27. 主机与备机之间的物理延迟是不可控的，也是无法避免的。但是如果仅仅需要满足这种强一致性，是相对简单的事：只需要在主机写入时，确认更新已经同步到备机之后，再返回写操作成功即可。这回极大地损害性能；

###网络
1. 网络分层；
2. HTTP response header;
3. HTTP status code: 2xx, 3xx, 4xx, 5xx;
4. GET/POST/OPTION/PUT/DELETE;
5. 如何实现分片下载；
6. Cookie、SESSION；
7. CSRF（Cross-site request forgery，跨站请求伪造）。如何防范 CSRF 攻击？可以注意以下几点：
  8. 关键操作只接受POST请求
  8. 验证码
  8. 检测 Referer
  8. Token：目前主流的做法是使用CSRF Token 抵御 CSRF 攻击。
9. XSS（Cross Site Scripting，跨站脚本攻击）
10. TCP协议：三次握手等；
11. Socket：大体的程序与调用的函数逻辑如下：
  1. socket() 创建套接字
  1. bind() 分配套接字地址
  1. listen() 等待连接请求
  1. accept() 允许连接请求
  1. read()/write() 数据交换
  1. close() 关闭连接

###前端
1. 对Angular/React/Vue的见解;
2. React/Angular的lifecycle
2. 封装、继承、多态使用JS实现；
3. 闭包；为什么要使用闭包？
4. Promise 的使用；
5. 原型链；
6. ES6有哪些新特性；
7. let,const；
7. String与Array常用方法；
8. apply,call,bind,
6. JS有哪些数据类型，这些数据类型中，哪些是引用类型；

###编程题
1. 给定一个整形数组，找出所有相加等于N的组合；
2. 超大整数相加

###缓存Tair
1. ldb：level DB，writen by Google;
2. mdb：memcache;
3. rdb：redis;
4. http://www.jianshu.com/p/c1b9ec30b994

##流程参考
1. 做过最满意的项目是什么？
2. 项目
  2. 为什么要做这件事情？
  2. 最终达到什么效果？
  2. 你处于什么样的角色，起到了什么方面的作用？
  2. 在项目中遇到什么技术问题？具体是如何解决的？
  2. 如果再做这个项目，你会在哪些方面进行改善？
3. 你最擅长的技术是什么？（或者此处改为技术基础问题）
4. 最近在学什么？接下来半年你打算学习什么？
5. 做什么方面的事情最让你有成就感？需求设计？规划？具体开发？
6. 后续想做什么？3 年后你希望自己是什么水平？
7. 在之前做过的项目中，有没有什么功能或改进点是由你提出来的？是否有参与和改进其它开源项目