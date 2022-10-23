 :punch::label::boom:

:fish:记忆，用于自己标注，不会的就标注

:zap:重要扩展，记忆加分

:bulb:提示，不需要记忆

:thinking:理解，不需要记忆

？疑问

# Java基础



## 集合



### #Collecion 和Collecions

Collecion：集合接口，是很多集合的父类，提供通用的接口方法。

Collecions：工具类，提供很多操作集合的静态方法，例如排序复制填充反转，实现线程安全，实现只读集合。

```java
Collections.synchronizedXxx(集合) //返回线程安全类
Collections.unmodifiableXxx(集合) //返回只读类
```



### #集合分类

```java
Iterable接口————Collection接口:
  ————List接口：存储有序、可重复的数据；“动态”数组
  	----ArrayList、LinkedList、Vector————Stack
  ————Set接口：存储无序、不可重复的数据-->数学集合
  	----HashSet————LinkedHashSet
  	————SortedSet接口----TreeSet(有序)
  ————Queue接口：存储有序、可重复的数据
  	————Deque接口----LinkedList    
      
Map接口：存储键值对的集合，是无序
  ----TreeMap(有序)、ConcurentHashMap、HashMap————LinkedHashMap、HashTable————Properties
```

:bulb:Vector、Stack、HashTable都是线程安全的

#### List、Set区别？	

List存储有序、可重复的数据，允许多个null值，**取List时可以使用迭代器取出全部元素再逐一遍历，还可以使用get(int index)获取指定下标的元素。**

Set存储无序、不可重复的数据，至多能有一个null值，取set时只能使用迭代器全部取出再逐一遍历。

### #List实现类:fish:

ArrayList：是List主要实现类、**线程不安全**、查找效率高、底层是**Object[] elementData**
LinkedList：双向链表、线程不安全、对于插入删除操作，比ArrayList效率高、底层是双链表(JDK1.6 之前为循环链表，JDK1.7 取消了)，比ArrayList更占内存。
Vector：List接口的古老实现类、线程安全、效率低、底层是Object[] elementData

:bulb:查找效率：ArrayList底层是数组，支持快速随机访问；插入删除：LinkedList无需更新数组。

#### #ArrayList的底层原理:fish:

底层是基于数组实现的，实例化时，**如果是选择空参构造器，首次添加元素时，底层数组才会扩容长度10**。而对于带参的构造器，则一开始就创建指定长度的数组。当添加的元素个数超过数组的长度，则默认扩容为原来容量的1.5倍(length+(length>>1))。			

:bulb:加粗为jdk8对jdk7的改进，即不同点，作用就是延迟数组创建、节省内存

#### #LinkedList的底层原理:fish:

LinkedList是一个实现了List接口和Deque接口的双向链表，支持高效的插入和删除操作，同时具有队列的特性。
	内部有两个指向链表首尾节点的属性（类型是 Node<E>）
	链表节点由内部私有类Node<E>实现，Node包含三个属性：前驱、后驱、 数据
	插入：链表指定位置插入时，需先找到插入位置，该查找过程并不是从头到尾查找，因为链表双端有两个引用，可以根据查找位置趋近于那边，就从那边开始往前/后找
	查找过程：Node<E> node(int index)

:bulb:指定位置插入代码

```java
Node<E> node(int index) {
    if (index < (size >> 1)) { //跟中间位置做判断
        Node<E> x = first;
		for...
    } else {
        Node<E> x = last;
        for...
    }
}
```

### #Set实现类:fish:

HashSet：Set接口的主要实现类、线程不安全、可以存储null值、底层是HashMap 
LinkedHashSet：作为HashSet的子类、线程不安全 、能按照元素的添加顺序遍历、底层是linkedHashMap
TreeSet：线程不安全、能对元素进行排序、底层是TreeMap

:bulb:LinkedHashSet添加也是无序的、只是链表的头尾和数据之间维护了两个引用，遍历效率比HashSet高
:bulb:TreeSet可以按照添加对象的指定属性进行排序（排序的方式有自然排序和定制排序），TreeSet默认有序，不可以存储null值(但可通过修改排序实现)

#### #HashSet底层原理

基于HashMap实现，使用HashMap来保存所有元素，元素跟HashMap的key一样，都是不可重复的，很多操作都是通过调用HashMap中的方法来实现。			

:bulb:Set接口中没有额外定义的新方法、使用的都是Collection中声明过的方法
:thinking:无序性和不可重复性，无序性不等价随机性 ，只是存储的数据在底层结构中不是按照索引顺序添加、而是由数据的哈希值决定的，不可重复性保证添加的元素按照equals()判断时、不会返回真，即相同的元素需要唯一

### #Map实现类:fish:

HashMap：Map的主要实现类、效率高，但线程不安全、key和value的值可为null

HashTable：Map的古老实现类、线程安全、但效率低、key和value的值都不可为null

ConcurrentHashMap：线程安全，优于HashTable(synchronized)

LinkedHashMap：能按照元素添加的顺序遍历（原因是添加了一对指针）

TreeMap：实现对元素的排序，默认按照key升序排序，可自定义排序比较器



HashMap的底层：数组+链表（jdk7及以前） 数组+链表+红黑树（jdk8）

TreeMap的底层： 红黑树

HashTable： 数组+链表  （全表锁）同步方法，效率低

ConcurrentHashMap：分段数组(从而实现分段锁--数据分段锁住)+链表（jdk7及以前） 数组+链表+红黑树（jdk8）---CAS



HashMap的底层原理：数据结构+添加元素底层+扩容

#### #HashMap添加元素的流程 :fish:

通过put添加元素，get获取元素。**首次添加元素的时候，才创建长度为16的数组**，根据key的hashCode和扰动函数计算hash值，然后计算存放在数组(**Node[]**，**Node实现了Entry**)中的位置(`hash`&`数组长度-1`), 如果该位置为空则直接添加，否则判断添加的元素和该位置的元素的hash和key是否一样，一样则更新；否则以链表的方式进行存储，**其中若链表的长度大于8，且当前数组长度大于64，则需要将链表改为红黑树存储**

:bulb:加粗为jdk8对jdk7的改进，即不同点

#### #HashMap扩容:fish:

添加元素过程中，当数组长度超出临界值（threshold）时，则扩容为原来容量的两倍，并将原有的数据复制过来

是否扩容：threshold = 负载因子（0.75）* 数组长度，当数组长度大于threshold则扩容



#### #HashMap 的长度为什么是 2 的幂次方?

首先哪怕是使用有参构造器传入指定长度，其底层也会通过算法将其转为2的幂次方。
目的是减少哈希冲突，让数据分布均匀。hash值无法直接作为数组下标，因为其值可能很大，内存放不下，需要通过运算来确定下标，最简单的就是与数组长度取模运算，但是这样效率不高，如果把数组长度设定为2的幂次方，就可以用**与运算**代替，即hash&(n-1) == hash % n;

#### #HashMap线程安全问题

1.8之前 HashMap 多线程操作会导致死循环问题，1.8之后解决，但可能存在数据被覆盖的情况，例如两线程添加元素时，计算数组存放位置一致，但可能都还没添加，也就是实际出现冲突却不知道，导致先后添加出现覆盖



### #排序接口

#### #Comparable 和 Comparator 的区别

都是接口，用于集合的定制化排序；使用Comparable 的话，排序的对象需要实现该接口，并重写compareTo方法，该方法用于编写比较方式，例如String等类的实现就是如此。对于Comparator，只需要重写compare()方法，然后编写比较方式，实例化该类并通过sort的参数传入



### #迭代器

Iterator迭代器：提供遍历集合(List，Set)的接口，通过集合中的迭代器方法获取迭代器实例，迭代器允许迭代过程中移除元素。

迭代器特点：安全，遍历时，如果元素被修改，能抛出异常。

如何使用：  Iterator it =  集合对象.iterator();  	while(it.hasNext()) it.next();

ListIterator： 继承于Iterator，只能遍历List，但能双向遍历。



### #JDK和JRE的区别:fish:

*  JDK是java开发工具包，提供开发环境和运行环境，包含有JRE，javac编译器，调试和分析工具
* JRE仅提供java运行环境，无法编写java程序，包含有JVM，java类库

:bulb:JDK(Java Development Kit)、JRE(Java Runtime Environment)



### # == 和 equals 区别:fish:

* ==可用于基本类型和引用类型的比较，基本类型的比较是内容、引用类型比较的是引用；
* equals用于引用的比较，默认比较的是引用，但对于重写了equals的类，比较的则是内容。



### #重写equals，为什么也需要重写HashCode:fish:

为了保证对象相等时(equals), 哈希值也一样，像Map，Set等集合，为了保证存放对象唯一，需要判断对象是否相等，其中就用到了equals和HashCode,  只有两者都返回真，对象才相等，否则当两对象相等时，但hash值并不相同，这时集合会判断两对象不等，都会添加到集合中，这显然不对，而且获取时也获取不到，因为获取时同样需要判断这个函数，这就是为什么需要重写的原因。反之，两对象哈希值相等，不一定相等(equals)， 因为会有哈希冲突。例如：String s1= "通话"   String s2= "重地"；



### 重载和重写的区别？

重载：发生在同一个类或父子类中，方法名必须相同，必须修改参数列表，方法返回值和访问修饰符可以不同，发生在编译时，可以声明新的或更广的异常。

重写：发生在父子类中，方法名、参数列表必须相同。返回值范围和抛出异常范围小于等于父类，访问修饰符范围大于等于父类。声明为**final/private/static**的方法以及构造方法不能被重写，但是被static修饰的方法可以被再次声明。如果子类和父类**不在同一个包**中，那么子类只能够重写父类的声明为 **public 和 protected** 的非 final 方法。

### #final 的作用

是一个修饰符

* 修饰的类不能被继承
* 修饰的方法不能被重写
* 修饰的变量一旦初始化就不能改值
  * 修饰类变量，声明时赋值或在静态代码块中赋值。
  * 修饰成员变量，可以在声明时赋值，也可以在构造代码块或构造函数中赋值。
  * 修饰局部变量，只要在使用前赋值即可。
  * 修饰基本数据类型时，一旦赋值不可修改，修饰引用数据类型的，其引用不可修改，值可以修改。


#### 为什么局部内部类和匿名内部类只能访问局部final变量？

如果外部类被销毁，外部类的局部变量也会被销毁。所以会拷贝一份供内部类使用，这时为了保证变量值一致，采用final进行修饰，确保不可修改。

### #String、StringBuffer、StringBuilder的区别:fish:

* String是不可变的对象，每次操作都会生成**新对象**，而StringBuffer，StringBuilder操作的是原有对象；
* StringBuilder线程不安全，但性能高；而StringBuffer是线程安全的；
* 他们的底层都是char数组，都是final类。

:zap:String不可变的原因：final类+private final char[]
:zap:append底层：数组扩容复制

### StringBuffer为什么是可变类？

​		因为StringBuffer的append方法的底层实现就是创建一个新的目标对象，然后将各个字符引用串接起来，这就是一个新的对象了，本质上就是栈多了一个变量，而堆上面并没有变化（为什么是在栈上呢？因为触发到的arraycopy方法是native的，也就是其生成的变量会放到**本地方法栈**上面去）。

　　那为什么我们没有把StringBuffer划分为不可变类呢？它明明在栈中创建了一个新的对象。因为一般的对象和数组都会在堆中进行存储，除非是发生了栈上分配现象。我们相比较String对象的存储，就可以知道，StringBuffer对象在此处并不符合栈上分配的条件（ *将线程私有的对象打散分配在栈上，可以在函数调用结束后自行销毁对象，不需要垃圾回收器的介入，有效避免垃圾回收带来的负面影响，栈上分配速度快，提高系统性能**）*，所以我们可以知道，StringBuffer的append方法并不会在堆上创建新的StringBuffer对象且内容是结果字符串，而是在arraycopy方法的帮助下，将各个字符引用连接起来。

### *StringBuffer StringBuilder扩容

扩容的容量为原来的2倍+2，同时进行数组复制。建议使用制定好容量的构造器，因为扩容需要进行数组复制。



### #普通类，抽象类区别:fish:

* 普通类不可有抽象方法，抽象类可以有

* 普通类可以实例化，抽象类不能

  

### #抽象类，接口区别:fish:

* 接口不能有构造方法
* 接口方法默认修饰符为public(jdk1.8可用default默认实现)；抽象类普通方法没有限制，抽象方法只能为public或protected
* 类只能继承一个抽象类，但能实现多个接口

![image-20220716162624477](%E9%9D%A2%E8%AF%95/image-20220716162624477.png)



### #IO流分类

按功能划分有输入输出流；按类型有字节字符流；

:zap:其中字节流按8位以字节为单位输入输出数据，而字符流则是16位，以字符为单位。

![img](%E9%9D%A2%E8%AF%95copy/iostream2xx.png)



### #BIO, NIO, AIO:fish:

Block IO 同步阻塞式IO, 就是传统IO，模式简单使用方便，并发处理能力底
New IO 同步非阻塞式IO, 传统IO的升级，客户端和服务端通过通道（Channnel）通讯，实现多路复用
Asynchronous IO  异步非阻塞式IO, NIO的升级(NIO2)，基于事件和回调机制



## 多线程

多线程能让一个进程并行的处理多个任务，从而提高CPU的利用率

### #并行和并发

* 并行同个时间点执行多个任务
* 并发同个时间段内执行多个任务

### #线程和进程的区别

​	一个进程至少有一个线程，创建进程的开销大，但进程之间互不影响，线程则相反。

:zap:进程是CPU资源分配的最小单位，线程是CPU调度的最小单位。

### #创建线程的三种方式

* 继承Thread类，重写run和调用start
* 实现Runnable接口，重写run，实例化并作为参数传入Thread对象，调用start
* 实现Callable接口，重写call，使用线程池 :fish:

### #实现和继承的优缺点

* 实现方式没有类的单继承的局限性
* 继承使用this获取当前线程，比较方便，实现需要调用方法。 :fish:

:question:实现方式更适合多个线程处理共享数据的情况

### #守护线程是什么 :fish:

1. 实现：thread.setDaemon(true)

2. 介绍：jvm进程结束时，不管守护线程是否执行完毕，守护线程会自动结束。例如Jvm的垃圾回收线程。适合执行后台任务且希望能自动结束的场景。

### #Runnable和Callable区别

​	Runnable没有返回值，Collable可以指定泛型 :fish:，并拿到返回值，也可以声明异常

### #线程的6种状态 :fish:

* NEW  未开启状态(没调用start)
* RUNNABLE  可运行状态(包括就绪态)

* BLOCKED  阻塞状态(锁住了)
* WAITING  等待状态，无法自动唤醒。
* TIME_WAITING  等待指定时间后重新被唤醒的状态
* TERMINATED  执行完成，终止状态。(执行完run)

### #sleep和wait的区别

* sleep来自Thread类，wait来自Object :fish:
* 调用sleep不释放锁，wait会释放锁
* sleep是到时间自动恢复，而wait是通过其它方法唤醒

:bulb:wait和notify都是由锁来调用。且都需要在同步方法中调用

### #notify和notifyAll的区别

​	唤醒调用wait等待的线程；notifyAll唤醒所有线程，它会将等待池的所有线程移到锁池，锁池的线程需要通过竞争获取锁，才能执行；而nofify只会唤醒一个

### #run和start的区别:fish:

​	run用于完成线程的任务，可重复调用；start用于启动线程和调用run，仅能调用一次

### #什么是死锁

​	多线程各自持有不同的锁，并试图获取对方已持有的锁，导致无限等待。

### #如何防止死锁:fish:

* [多线程获取锁的顺序要一致](https://www.liaoxuefeng.com/wiki/1252599548343744/1306580888846370)
* 通过设置超时时间，超时可以退出，例如使用tryLock设置获取锁的时间，超时返回false
* 使用java.util.concurrent包下的并发类代替手写锁

### #synchronized同步机制

* 同步代码块

  	synchronized(同步监视器){
  		//需要被同步的代码块
  	}

* 同步方法，隐藏了锁

  ```java
  public synchronized void method(){}  this作为锁
  public static synchronized void method(){} 当前类本身作为锁
  ```

:bulb:基本概念

```java
~操作共享数据的代码，即为需要被同步的代码
~共享数据：多个线程共同操作的数据
~同步监视器：俗称锁---任何一个类的对象都可充当锁，但多个线程必须公用同一把锁
~临界区--加锁和解锁的代码块
```

### #synchronized的原理（javap -c -s -v -l  Xxx.class）:fish:

​	对于同步代码块，由monitorenter和monitorexit指令实现，这对指令分别指向同步代码块的开始位置和结束位置。（执行monitorenter时，通过获取对象监视器monitor来加锁，锁计数器加一；通过执行monitorexit释放锁，锁计数器减一）。而对于方法，由ACC_SYNCHRONIZED标识，指明该方法为同步方法。

:zap:JDK1.6前后的synchronized

6前，锁的实现依靠操作系统内部的互斥锁，而操作系统线程之间的切换需要从用户态转换到内核态，花费时间长，性能底。6的时候，由虚拟机提供三种不同的锁，分别是偏向锁，轻量锁，重量级锁，改进了性能。

？*synchronized升级原理	CAS
	

### #ReentrantLock和synchronized:fish:

* 都是可重入锁---递归锁（计数）
* RLock需要手动加锁和释放锁，synchronized不用
* RLock只适用于代码块，synchronized还可用于方法
* synchronized发生异常会自动释放锁，而RLock需要配合try/finally释放锁。
* RLock可以知道是否成功获取锁（tryLock）。synchronized不能

?RLock使用起来比较灵活

### #volatile的用法和原理

保证共享变量的可见性和有序性（避免重排列），写时刷新到主内存，读只能从主内存读

### #volatile和synchronized区别 CAS

* volatile是变量修饰符；不会造成线程堵塞；仅能实现变量的修改可见性，不能保证原子性
* synchronized是修饰方法或者代码段；可能会造成线程堵塞；都可以保证。
* volatile标记的变量不会被编译器优化，synchronized可以被优化

？源码atomic原理 CAS	

​	利用CAS（Compare And Swap）、volatile、native方法保证原子操作，从而避免synchronized的高开销，执行效率大为提升

### #怎么保证多线程运行安全:fish:

* 使用安全类，如并发集合java.util.concurrent包下的原子类、集合类
* 使用synchronized同步机制
* 使用手动锁Lock
* volatile

### #介绍ThreadLocal和使用场景:fish:

能够为每个线程关联一个数据，每个线程可以通过get/set存取关联数据，而不会影响其它线程所关联的数据。（每个线程有一个ThreadLocal.ThreadLocalMap，Map以**ThreadLocal变量为Key**，线程所关联的变量为value。ThreadLocal调用set是把数据存放到ThreadLocalMap中，get也是从里面取）

用于数据库连接（每个线程关联一个来连接，防止当前线程所用连接被其它线程关闭）和session管理。



### #线程池:fish:

作用：线程的复用

 	ExecutorService接口接收
 	ExecutorService e = Executors.xxxPool(num)  Executors工厂类用于生产线程池
 	e.submit(task1)提交任务
 	e.shutdown()关闭池子

创建方式：

```
new ThreadPoolExecutors()，最原始的创建方式，下面三种都是对它进行封装。可创建指定范围数的线程池
new FixedThreadPool()--线程数固定
new SingleThreadExecutor()--仅单线程执行，相当于new FixedThreadPool(1)
new CachedThreadPool()--线程数根据任务动态调整,线程会被缓存到线程池。线程闲置时间超时则会被移除。默认多少任务就创建多少线程

newScheduledThreadPool()--可以进行定时或周期性的工作调度。适合需要定期反复执行的任务
J8新增
newWorkStealingPool()--基于ForkJoinPool,提高并行效率
```

状态：
RUNNING	能接受新提交任务，也能处理阻塞队列任务
SHUTDOWN	不再接受新提交任务，但能处理阻塞队列任务
STOP	中断正在执行任务的线程
TIDYING	所有任务都终止，有效线程数为0
TERMINATED	TIDYING状态的线程池会调用terminated()进入TERMINATED状态

execute和submit区别:

execute只能执行Runnable类型的任务，submit还能执行Callable类型的任务，且能返回Future对象。



## 反射

程序运行时，通过类的Class对象获取类的信息。

### #动态代理

介绍：程序运行时动态生成代理类。

应用：spring aop

实现：jdk原生动态代理和cglib动态代理，前者基于接口实现，后者基于继承当前类的子类实现。具体参见代理模式



## 序列化和反序列化:fish:

把内存的中的Java对象转换成二进制流，从而允许把这种二进制流持久的保存在磁盘上，或者通过网络将这种二进制流传输到另一个网络节点。当其它程序获取了这种二进制流，就可以恢复成原来的Java对象。



## 拷贝

*为什么要使用拷贝(克隆)：能够保存当前对象的状态。

*如何实现：实现Cloneable接口和重写clone()--调用Object；

深拷贝和浅拷贝区别：拷贝时是否包含对象内的对象。

:bulb:序列化和反序列化也能实现深拷贝 



## JavaWeb

### #JSP和Servlet区别

​	jsp本质上也是一个Servlet程序(由服务器翻译)，代替Servlet程序回传html页面数据。两者的区别在于servlet与html是分离的，仅作用于java文件，而jsp则可以与html结合为一个jsp文件。

### #JSP九大内置对象

​	jsp九大内置对象，指Tomcat翻译jsp页面成为 Servlet 源代码后，内部提供的对象
​    pageContext（上下文对象，可获取其它对象）	out（输出流对象）	page（当前jsp对象）

​	application（ServletContext对象）	config（ServletConfig对象）	

​	session	request	response	exception(异常对象--需要在头部page配isErrorPage属性)

### #JSP四大作用域

​	pageContext (PageContex类)	当前 jsp 页面范围内有效

​	request (HttpServletRequest类)	一次请求内有效

​	session (HttpSession类)	一个会话范围内有效（打开浏览器访问服务器，直到关闭浏览器）

​	application (ServletContext类)	整个web 工程范围内都有效（只要web工程不停止，数据都在, 重新部署就没了）

### #Cookie和Session区别:fish:

* 存储位置：cookie存放在浏览器端；session存放在在服务端，存放位置多样，如Redis，数据库
* 安全性：cookie安全性相对较低，不适合存放隐私的数据
* 容量：cookie的个数和大小都有限制，单个大小限制是3k

### #Session原理:fish:

用户登录之后，服务端会创建对应的session，并把session的id发送给客户端，客户端再存到浏览器的cookie中，当用户再次访问服务器时，会携带cookie，服务器会获取里面的sessionid，并找到匹配的session，从而确定用户身份。

:zap:客户端禁用cookie后，如何实现session：可通过url添加sessionid的方式



### Tomcat

解压缩后的文件

![image-20220717194403011](%E9%9D%A2%E8%AF%95copy/image-20220717194403011.png)

## 数据库

### 1、记忆题

#### #如何避免sql注入

* 使用预编译，也就是preparedStatement代替Statement。
* 使用正则表达式过滤掉字符中的特殊字符。



#### #事务隔离存在的问题

* 脏读：读取未提交事务的数据（事务A添加数据，但未提交，此时却能被事务B读取到）
* 不可重复读：一个事务内，相同查询的结果不一致（表无数据，事务B查询条数为0，未提交，然后事务A添加数据并提交，接着事务B查询数据库条数变为1）
* 幻读：事务B查询不到某条数据，此时事务A添加了这条数据，事务B再次查询，同样查不到，于是进行添加操作，但发现无法添加（明明不存在的数据，居然无法添加）



#### 事务ACID

* 原子性：事务内的所有操作是不可分割的，要么都成功，要么都失败。
* 一致性：事务执行前后，数据库的完整性没有被破坏（例如A向B转账，A+B的总额不变）
* 隔离性：事务并发执行时，互不干扰。
* 持久性：事务一旦提交，对数据的修改就是永久的，即使系统故障也不会丢失。



#### 说说数据库的事务隔离级别

:zap:存在问题：A脏读、B不可重复读、C幻读
**READ_UNCOMMITTED**	未提交读，最低的隔离级别，允许读取尚未提交的数据变更；ABC **READ_COMMITTED**	提交读，一个事务提交之后才能被其它事务被读取。；BC 
**REPEATABLE_READ**  可重复读，一个事务多次查询的结果相同，除非自己改自己；C 
**SERIALIZABLE**	可串行化，最高隔离级别，完全符合acid，但最慢；
隔离级别可通过配置文件(mysql.ini)修改，默认是可重复读（transaction-isolation=REPEATABLE_READ）



#### 常用存储引擎

* InnoDB是默认的事务型存储引擎，具有提交、回滚功能；使用行级锁提高并发写效率；支持外键功能；支持数据库崩溃恢复功能；
* MyISAM引擎的表占用空间较小,表级锁定限制写操作效率, 通常用于只读或以读取为主的场景。
* Memory引擎：置于内存的表

:zap:5.5默认引擎由MyISAM转InnoDB



#### InnoDB、MyISAM的区别和选用场景

- 事务：InnoDB支持事务；MyISAM不支持。
- 并发：MyISAM 只支持表级锁，而 InnoDB 还支持行级锁。
- 外键：InnoDB 支持外键。
- 崩溃恢复：InnoDB有崩溃恢复机制；MyISAM没有
- 存储文件：
  表名.frm 存储表结构（MySQL8.0时，合并在表名.ibd中） 表名.ibd 存储数据和索引（InnoDB）
  表名.frm 存储表结构  表名.MYD 存储数据  表名.MYI 存储索引（MyISAM）
- 索引：InnoDB数据和索引存放在一起，而MyISAM是分开的

选用：需要事务选InnoDB；只读或以读取为主选MyISAM；是否在乎系统崩溃恢复；

> [推荐阅读](https://blog.csdn.net/qq_35642036/article/details/82820178)

#### 行级锁和表锁

* 表锁，即使操作一条记录也会锁住整个表，不适合高并发的操作 

* 行锁，操作时只锁住某一行，不对其它行有影响，适合高并发的操作

:zap:InnoDB行级锁是通过给索引上的索引项加锁来实现的。只有通过索引条件检索数据,InnoDB才使用行级锁,否则,InnoDB将使用表锁。



#### 乐观锁和悲观锁（同redis）

悲观锁：每次操作数据都会上锁，保证其他人无法操作该数据。

乐观锁：不上锁。而是在修改数据之前，会检查该数据是否被修改。

:zap:乐观锁可通过给表添加一个version字段，每次修改数据前进行比较，若version不一致，则不修；一致则修改且更新version为version+1（版本号机制），适合多读场景。



#### 谈谈索引以及它的好处和坏处

索引是一种的数据结构，能够快速的找到数据。

好处：索引能减少对数据库的IO操作，从而提高检索效率；唯一索引能保证表的记录是唯一的；**减少连接、分组、排序的查询时间。**

坏处：索引的创建和维护需要耗费时间。虽然索引提高了查询速度，但对于数据的增删改操作， 索引需要跟着一起修改，这就导致sql执行效率下降；索引会占用一定的存储空间。

#### 索引原理

MySQL中使用InnoDB和MyISAM引擎时都使用了B+树实现索引。（要能画出B+树的图，只要能画出来，啥都能想起来）

b+树：待补充

#### 为什么不使用Hash索引或红黑树或B树

* hash索引不适合范围数的查找，也不适合用于排序和分组（存储是无序的）
* 红黑数是二叉树，说明每个节点只有两个分支，当数据多时，会导致树很高，树高会导致磁盘IO次数变多
* B树的中间节点也存储数据，这就使得在同样大小的节点，B树存储的关键字比B+树少，也就是B+树的高度会更低，也就IO次数更少；除此之外，B+树的叶子节点通过指针连接，且数据都是有序的，范围查找的速度更快，而B树的查找需要通过对树的中序遍历

:zap:索引树的节点对应每个磁盘页，查询时，逐一加载每个磁盘页，而不是一次性加载整颗搜索树。



#### B+树存储能力？一次查找，最多只需2-4次IO?

InnoDB的数据页大小为16kb, 假如一条记录160字节，一条目录项记录为16字节，则一个数据页可存100条记录，1000条目录项记录，当b+树为3层时，数据就能达到1亿。可见b+树的高度一般在2-4层，而树的高度决定了磁盘IO次数。这也是为什么一次查找，最多只需2-4次IO的原因

:zap:innoDB存在自适应哈希，也就是当你重复查询时，会把数据地址放到哈希表中，下次查询直接通过哈希表返回。

```msyql
 show variables like '%adaptive_hash_index' #默认开启
```



#### B树和B+树（同样需要能画出来）

待补充



#### 请你说说聚簇索引和非聚簇索引

聚簇索引就是是将数据与索引存储到一起, 找到索引也就找到了数据；而非聚簇索引是将数据和索引存储分离开。。

在InnoDB中，对于聚簇索引，B+树的叶子节点存放主键和数据；而非聚簇索引则存放索引列和主键，查询时，需要找到对应的主键，然后再通过聚簇索引找到数据。

在MyISAM中，只有非聚簇索引，其B+树的叶子节点存放索引列+数据记录的地址。查询时通过地址到数据文件中找到数据。

:zap:非聚簇索引的查找过程称为回表

:zap:联合索引就是为多个列建立索引



#### 主从复制（主从同步）

1. 主服务器（master）把数据更改记录到二进制日志（binlog）中。(binlog线程)
2. 从服务器（slave）把主服务器的二进制日志复制到自己的中继日志（relay log）中。 
3. 从服务器读取中继日志，并根据日志更新服务器上的数据，从而实现数据同步。



数据库优化

联合索引的最左侧原则（哈希不支持）

数据库三范式



### 2、场景题：

#### char和varchar的区别？

* 固定长度，效率高，但浪费空间（例如char(10)存储“abc”，仍占用10字节）
* 可变长度，节省空间（存放的是串+串的长度）

#### float和double的区别？

float占4字节，double占16个字节

#### 自增表有7条数据，删除最后两条，重启数据库，插入一条，id为几？

取决表使用的储存引擎，MyISAM为8，InnoDB为6（自增主键的最大id只存放在内存，重启丢失）

#### 如何验证mysql索引是否满足需求？

使用explain查看sql语句是如何执行的，语法：explain sql语句;

### 异常

throw和throws区别：抛出一个异常；是异常处理的一种方式，声明可能会抛出一个异常。

final、finally、finalize区别

*  见《 final 的作用》
 * try-catch-finally的最后一部分，无论是否捕获或处理异常，都会执行该块的语句
 * Object中的方法，垃圾回收时会调用回收对象的这个方法。

try-catch-finally那部分可省略

​	catch和finally可以省略，但不能同时省略。

try-catch-finally的使用

​	try块：用于捕获异常
​	catch块：用于处理异常
​	finally:无论是否捕获或处理异常，都会执行该块的语句



常见的异常类

* 编译时异常：
  IOException---FileNoFoundException
  ClassNotFoundException、NoSuchMethodException 方法不存在异常、SQLException

* 运行时异常

  NullPointerException、ArrayIndexOutOfException、ClassCastException
  ArithmeticException  算术异常、NumberFormatException  数字格式异常
  InputMismatchException  输入不匹配异常

Throwable分类（异常分类）

1. Error：错误是程序无法处理的----JVM系统内部错误，例如虚拟机内存不足

2. Exception：异常是能被程序处理的，分为检查和非检查异常（检不检查是指在编译时期 ）
   ①检查异常：编译时异常
   ②非检查： 运行时异常(RuntimeException及其子类)



### 网络

**http响应码301和302：**
	永久重定向和暂时重定向，前者对搜索引擎优化（SEO）有利，后者可能会被提示为网络拦截。

**forward和redirect：**

请求：服务器接收到请求，然后跳转到其它资源的过程。

重定向：客户端给服务器发请求，服务器提供新地址给客户端去访问。

* url：请求的地址栏url不变，重定向会改变
* 域数据：请求能共享域数据，例如request域中的数据，重定向不能	
* 效率：请求比重定向高

**get/post区别**

它们是http协议中两种请求方式，本质上并没有什么区别。可以把 get 和 post 当作两个不同的行为，底层都是 TCP 连接。get用来从服务器获取信息，post用于向服务器提交数据

无区别理解：

get参数写法按约定是固定的，例如?a=2&b=2。但其实只要服务器能解释，什么写法都可以。

get url长度限制的原因是浏览器和服务器的原因，http协议并没有限制，例如浏览器处理长url需要消耗很多资源

post跟get都不安全，虽然post的数据在地址栏看不到，但http都是明文传输，要安全就要加-https



**osi七层：**

物理层、数据链路层、网络层、传输层、会话层、表示层、应用层。

五层网络体系结构

网络层：提供主机之间的通信

运输层：负责主机应用进程之间的通信

* 功能：复用和分用（报文加上首部（源端口号和目的端口号））

  复用指发送方不同的进程能在同一个协议传送数据。
  分用指接受方能把收到的数据正确的交付到对应进程。

* 端到端协议：TCP（传输控制协议）、UDP（用户数据报格式）

TCP和UDP区别

都是运输层协议。

udp是无连接、尽最大努力交付、没有拥塞控制（实时应用）、传输慢、面向报文、支持一对一，一对多、多对一和多对多的交互通信，首部开销为8字节（源端口，目的端口，数据长度，检验和==差错检验码）

tcp是面向连接、提供全双工和可靠交付服务、传输快、面向字节流、只能是一对一通信、首部开销20个字节



理解：tcp首部
源端口、目的端口、序号seq-4字节：报文段数据的第一个字节序号、确认号ack-4字节（下一个报文段数据的第一个字节序号）序号加上携带数据大小==确认号、数据偏移-4字（半个字节，记录首部长度）、保留-6字、(紧急URG,确认ACK,推送PSH,复位RST,同步SYN,终止FIN)-1字、窗口-2字节（接收窗口大小,告诉发送方我的接收能力，从而控制发送方的发送量）、检验和、选项

ACK：为1代表确认好有效，为0无效

SYN：用于建立连接，为1表示一个连接请求或连接接受报文 ,为0代表普通报文

FIN：用于释放连接，为1表示发送方的报文发送完，请求释放



**tcp3次握手--建立连接**

客户端向服务器发送连接请求报文段，SYN=1,序号seq=x

服务器收到后，发送连接请求确认，SYN=1,ACK=1,序号seq=y,ack=x+1

客户端收到后，还需要给服务器发送确认，SYN=0,ACK=1,seq=x+1,ack=y+1

**tcp为什么要3次握手，或者说为什么要有最后一次ACK**

防止客户端失效的连接请求突然被服务器收到，例如客户端向服务器发送连接请求，但该请求可能一开始没被服务器收到，此时客户端再发送连接请求，这次一切正常，直到连接结束，但此时最初的那个请求被服务器收到，此时服务器以为是建立新连接，于是向客户端发送确认报文，但客户端并没有搭理，这就可能导致服务器陷入等待状态，白白浪费了许多资源，所以才需要客户端发送第三个确认。

tcp粘包和拆包问题

发送端需要等缓冲区满才发送出去，造成粘包，拆包相反（写入数据大于缓冲区大小）。

接收方不及时读取缓冲区的数据，造成粘包



**4次挥手--关闭连接**

相当于两个二次挥手，因为需要关闭两个方向的连接，所以需要四次

客户端向服务器发送连接释放请求报文段，FIN=1,序号seq=u

服务器收到后发出确认，ACK=1,序号seq=v,ack=u+1

服务器向客户端发送连接释放请求报文段，FIN=1, 序号seq=w,ack=u+1

客户端收到后发出确认，ACK=1,序号seq=u+1, ack=w+1



# 框架

## mybatis

#### #{}和${}的区别*

* #{}是预编译处理，mybatis处理sql时会将#{}替换为问号，调用PreparedStatement来赋值。执行效率高，可以防止sql注入
* ${}是字符串替换，mybatis处理sql时把${}替换为变量的值，相当于sql拼接，虽然不安全。但却适合一些其它场景，例如传递不同列名给sql，而预编译无法解决。
  ${}也可以用于获取引入配置文件（properties）中的值，例如数据库

理解：预编译会把传入的参数当成一个字符串，也就是会加上引号，这样就不会发生sql注入。

#### **分页方式**

逻辑分页：使用RowBounds进行分页，它从数据库是一次性查询很多数据，然后再对数据进行分页。

物理分页：手写sql或者分页插件，直接从数据库查询指定条数的数据。

理解：RowBounds是一次性查询很多数据，但不是全部，首先mybatis是对jdbc的封装，而jdbc中规定了每次从数据库查询的最大条数，也就是内部也是分多次查询的。逻辑分页是再内存层面上的操作，存在消耗内存的弊端，

#### **延迟加载**

设置lazyLoadingEnabled=true，仅支持关联查询

懒加载原理是调用的时候才触发，而不是初始化时就加载，例如a.getStudent()用于获取学生及对应导师的信息，但如果使用懒加载，调用a.getStudent的时候只返回学生的信息，当调用a.getSutdent().getTeacher()才返回学生和教师的信息。

一级和二级缓存

默认只开启一级缓存，SqlSession级别的，同一SqlSession查询，MyBatis 会把执行的方法和参数通过算法生成缓存的键值，将键值和查询结果存入一个Map对象中。如果同一个SqlSession 中执行的方法和参数完全一致，那么通过算法会生成相同的键值，当Map 缓存对象中己经存在该键值时，则会返回缓存中的对象。

一级缓存失效：增删改操作（刷新缓存）；手动清除缓存---sqlSession.clearCache()，仅对一级有效；

二级缓存是基于命名空间级别的缓存，不同会话可以通过二级缓存获取内容。需要配置在Mapper.xml中配置<cache/>，同一mapper有效，一级缓存先作用，一级失效，才使用二级

二级缓存失效：增删改操作（刷新缓存）；

#### #三种基本的执行器Executor

* SimpleExecuter：每执行一次 update 或 select，就开启一个 Statement 对象，用完立刻关闭 Statement 对象。
* ReuseExecutor：执行 update 或 select，以 sql 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后，不关闭 Statement 对象，而是放置于 Map<String, Statement>内，供下一次使用。简言之，就是重复使用 Statement 对象。
* BatchExecutor：执行 update（没有 select，JDBC 批处理不支持 select），将所有 sql 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个 Statement 对象，每个 Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理。与 JDBC 批处理相同。

#### #分页插件实现原理

分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql，根据不同数据库，添加对应的物理分页语句和物理分页参数。

#### #自定义插件实现原理

MyBatis自定义插件针对 `ParameterHandler` 、 `ResultSetHandler` 、 `StatementHandler` 、 `Executor` 这 4 种接口进行拦截，MyBatis 使用 JDK 的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能。

#### #如何编写自定义插件

实现 MyBatis 的 Interceptor 接口并重写 `intercept()` 方法，然后再给插件添加注解@Intercepts，指定要拦截哪一个接口的哪些方法即可，最后在配置文件中配置你编写的插件。



## spring

#### #什么是spring框架、为什么要用spring（优点）

spring一个轻量级的、非入侵式的Java开发框架；
提供ioc和aop技术，用于管理对象和支持面向切面编程。
spring也支持事务的处理、可以很方便的集成其它框架。

#### #spring有那些主要模块(这里以依赖进行说明)

spring core 	核心模块，提供ioc依赖注入功能
spring context 
spring data access  提供了jdbc的抽象层、事务、其它框架支持
spring aop  提供面向切面编程
spring web  提供mvc功能等

#### #IOC

即控制反转，由框架负责对象的创建、管理，而不由程序本身负责。
我们需要那个对象就去ioc容器中获取，而不是自己使用new去创建。

#### #AOP

通过面向切面的方式来编程，把项目中具有公共功能的部分抽取出来，例如日志、事务、权限等。
aop基于动态代理，可以是jdk动态代理，也可以是cglib。

#### #Bean常用注入方式

setter、构造器、注解

#### #Bean作用域

通过scope属性或@Scope注解配置，默认是单例模式（singleton），原型模式（prototype，同个controller的bean是同一个，getBean获得的则每次都不相同）
request：每次http请求都会创建一个bean
session：一次会话对应一个bean

#### #Bean是线程安全么

默认单例的bean存在线程安全，不过还需要看该bean是否用于存放数据，大部分bean都是不用于存放数据的，例如dao，service。如果需要，可通过修改作用域或使用ThreadLocal来实现线程安全。

#### #Spring自动装配bean方式

byName，ByType，注解@Autowired等（得懂这些是什么意思）

#### #事务实现方式有那些

* 声明式事务：两种，基于xml配置、注解（@Transactional）
* 编程式事务：通过编码实现

#### #事务的五大隔离级别

存在问题：A脏读、B不可重复读、C幻读

**DEFAULT**  使用数据库默认的隔离级别，数据库设置什么就是什么，其它四个与数据库的隔离级别一样。 
**READ_UNCOMMITTED**	未提交读，最低的隔离级别，允许读取尚未提交的数据变更；ABC **READ_COMMITTED**	提交读，一个事务提交之后才能被其它事务被读取。；BC 
**REPEATABLE_READ**  可重复读，一个事务多次查询的结果相同，除非自己改自己；C 
**SERIALIZABLE**	可串行化，最高隔离级别，完全符合acid，但最慢；

:zap:后四个同mysql

## SpringMVC

#### #mvc运行流程



#### #mvc有那些组件

DispatcherServlet、HandlerMapping、Controller、ModelAndView、ViewResolver

#### #@RequestMapping作用



## SpringBoot

#### #什么是springboot

它是spring的脚手架，能够快速构建项目，提高开发效率。

#### #优点

配置简单，自动装配，依赖管理、项目可独立运行，无需外部依赖Servlet容器，提供应用监控。

#### #核心配置文件

bootstrap和application，前者比后者优先加载，且里面的属性不能被覆盖，application则是用于项目的自动配置

#### #常用注解

@SpringBootApplication注解：声明这是一个sb应用

* 子注解 @SpringBootConfiguration
  该注解表示，可以用 Java 代码的形式来实现 Spring 中 xml 配置文件配置的效果。
* 子注解 @EnableAutoConfiguration
  Springboot 的核心注解。该注解表示开启自动配置，Spring Boot 能够根据项目中依赖的 jar 包，自动配置依赖需要的基本配置。比如我们的项目引入了spring-boot-starter-web 依赖，springboot 会自动配置 tomcat 和 springmvc。

:bulb:底层都是通过Spring注解实现。

#### #自动装配流程

通过@EnableAutoConfiguration注解开启自动配置，具体通过@Import({AutoConfigurationImportSelector.class}) 注解，通过加载spring-boot-autoconfigure jar包下的spring.factories文件，向容器中注入各种AutoConfiguration类；而且这些类都是按需配置的，只有满足@Conditional注解的条件，才配置；其它类的注入则由这些AutoConfiguration实现，例如DispatcherServlet的注入由DispatcherServletAutoConfiguration实现。

简化：通过`@EnableAutoConfiguration`注解在类路径的META-INF/spring.factories文件中找到所有的对应配置类，然后将这些自动配置类加载到spring容器中。



#### #Springboot启动流程

在启动类中，通过SpringApplication的静态run方法进行SpringApplication的实例化，然后实例化对象调用另一个run方法，创建应用监听器SpringApplicationRunListeners开始监听，接着加载SpringBoot配置环境(ConfigurableEnvironment)，然后把配置环境(Environment)加入监听对象中，然后加载应用上下文(ConfigurableApplicationContext)，当做run方法的返回对象，最后创建Spring容器，refreshContext(context)，实现starter自动化配置和bean的实例化等工作。

简化：在启动类中，通过SpringApplication的静态run方法进行SpringApplication的实例化，然后实例化对象调用另一个run方法实现项目的初始化和启动。



## SpringCloud

#### #介绍

多种技术的总称、是一个框架集合体，能实现微服务。

:zap:微服务：一种架构风格，项目由多个独立服务构成，每个服务独立运行(独立进程)。

#### #熔断器的作用

调用过程中，如果服务器挂了，执行熔断机制，断掉连接，停止请求。

#### #核心组件有哪些

* 服务注册和发现(Eureka)  ：服务都注册到注册中心，如Nacos
* 服务调用(Feign) ：根据服务名找到对应的服务和调用的方法
* 熔断器(Hystrix)：调用过程中，如果服务器挂了，执行熔断机制，断掉连接，停止请求。
* 负载均衡(Ribbon)：对请求做负载均衡，让请求分担到不同服务器上。
* 服务网关(GateWay) ：
* 分布式配置(Config)：能够抽取统一配置，进行独立管理，修改配置时无需重启服务器，nacos会刷新

:bulb:旧的网关--Zuul

## Linux & 操作系统

进程间的通信方式

一、**进程间的通信方式**

管道( pipe )：

管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。

有名管道 (namedpipe) ：

有名管道也是半双工的通信方式，但是它允许无亲缘关系进程间的通信。

信号量(semophore ) ：

信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。

消息队列( messagequeue ) ：

消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。

信号 (sinal ) ：

信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。

共享内存(shared memory ) ：

共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号两，配合使用，来实现进程间的同步和通信。

套接字(socket ) ：

套接口也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同设备及其间的进程通信。



如何查看进程内存

* 进程状态 top
* 当前进程信息 ps
* 进程数pstree

常用命令



多线程

[Java多线程学习之线程间通信 - 简书 (jianshu.com)](https://www.jianshu.com/p/9c3fe3522b2b)

[面试必问：Java 是如何实现线程间通信的？_Java小咖秀的博客-CSDN博客](https://blog.csdn.net/qq_14958051/article/details/119770024)



# JVM

#### 什么是虚拟机？

一个用于解析和运行java程序的虚拟计算机。

什么是类加载

#### 何时类加载？（类加载时机）

* 构造实例
* 调用该类的静态属性和静态方法
* 使用子类时会触发父类加载



#### $有哪些类加载器？（ClassLoader）

分类：引导类加载器(Bootstrap ClassLoader)(无法获取)、扩展类加载器(Extension ClassLoader)、系统类加载器(Application ClassLoader)、自定义类加载器
作用：把class文件加载到jvm内存，然后转为class对象
关系：引导类（Bootstrap）加载器主要负责加载java的核心类库jre\lib\，无法加载自定义类，即便通过核心库类也无法获取该加载器。
			扩展类加载器主要加载jre\lib\ext
			应用程序类加载器主要加载当前类路径下的类

> [代码](https://gitee.com/-/ide/project/zhandiming/java-code/edit/master/-/src/main/java/com/jvm/classload/ClassLoader1.java)

![image-20220718100922988](%E9%9D%A2%E8%AF%95copy/image-20220718100922988.png)

#### $类加载的流程：（遇到需要运行的类才去加载，且只加载一次）

1. 加载：类加载器将各个来源的字节码文件装载到内存中
2. 验证：检查字节码文件的正确性
3. 准备：类变量分配内存，并设置默认初始值
4. 解析：将常量池内的符号引用（标示）替换为直接引用（指向内存）
5. 初始化：初始化静态变量和静态代码块

链接阶段为234

[类加载](https://zhuanlan.zhihu.com/p/33509426)



#### $双亲委派机制?

加载器收到加载类请求时，会把请求委派给父加载器去完成，每个加载器都是如此，不断向上委托（直到引导类加载器，再向下判断），只有父加载器没法加载时才自己处理。
作用：避免JDK类被覆盖
例如你定义了一个同包的String类，但是类加载时，JDK中的会被先加载，
而且只要某个环节成功加载，后面的都不需要加载



#### jvm组成部分（java程序的运行流程）

1.**类加载器**把字节码文件加载到内存
2.然后存放到**运行时数据区**
3.**执行引擎**（解释器）将字节码翻译为能被操作系统识别的机器指令，由cpu去执行
4.这个过程需要调用其它语言的**本地库接口**来实现完整的功能。



#### $jvm内存区域划分（运行时数据区）

* 程序计数器：存储的是地址，描述当前线程接下来要执行的指令在什么地方）
* java虚拟机栈：根据栈帧实现方法的调用（每一个方法从调用到执行结束的过程，对应着一个栈帧从入栈到出栈的过程），可存放基本类型，引用等。
* 本地方法栈：类似虚拟机栈，本地方法（native）运行时占用的内存即为本地方法栈。
* java堆：是jvm内存中最大的一块，且内存的大小可调节（通过参数-Xmx和-Xms设定），用于存放对象实例；是垃圾回收的主要区域。
* 方法区（Method Area）：存放加载好的类、静态变量等。

其中**程序计数器、java虚拟机栈、本地方法栈**是线程私有的

而**java堆、方法区**是线程共享。

图：<img src="https://img-blog.csdnimg.cn/6f74b39b0a0a42ea9281ac32869af118.png" alt="在这里插入图片描述" style="zoom: 67%;" />

jdk1.6和1.7：方法区的实现都是永久代，不过字符串常量池、静态变量由永久代移到了堆，保留了运行时常量池

jdk1.8：无永久代，方法区的实现是元空间（本地内存），运行时常量池和类型信息等都移到了元空间

[元空间和永久代的区别](https://zhuanlan.zhihu.com/p/505516868)

#### native方法

用于调用其它语言实现的方法，因为java无法操作操作系统底层，
但是可以通过jni(本地方法接口)调用其它语言的方法来实现。

#### 本地方法栈

本地方法运行时占用的内存即为本地方法栈

#### 堆栈的区别

* 功能：堆用于存放对象、栈用于执行程序
* 线程：堆线程共享，栈私有
* 空间：堆远大于栈

#### Error

**java虚拟机栈、本地方法栈**：在栈深度溢出（死循环递归）和栈扩展失败（）时分别抛出StackOverflowError和OutOfMemoryError异常。

**java堆**：如果在Java堆中没有内存完成实例分配，并且堆也无法再扩展时，Java虚拟机将会抛出OutOfMemoryError异常。

栈扩展（-Xss）

### GC 垃圾回收

#### GC特点

GC中主要回收的是**堆和方法区**中的内存，栈中内存的释放要等到线程结束或者是栈帧被销毁，而程序计数器中存储的是地址不需要进行释放。



#### 如何判断对象是否死亡（是否可以被回收）

* 引用计数器算法：为每个对象创建一个引用计数，有对象引用时计数器+1，引用被释放时计数-1，当计数器为0时就可以被回收，缺点是不能解决循环引用问题。

* 可达性分析算法：通过一系列称为“GC Roots”的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为“引用链”，如果某个对象到GC Roots间没有任何引用链相连，则证明此对象是可以被回收的。
  ![image-20220509223206919](https://img-blog.csdnimg.cn/8e48d44850ee49dfbd6dfcbe9a25be6e.png)

  从哪里开始遍历（即什么可以作为GC Roots）：栈帧局部变量表、常量池中引用的对象、方法区静态变量引用的对象。



#### Java中有那些引用类型

* 强引用：gc时不会被回收，即便内存溢出

* 软引用：有用但非必须的对象，内存溢出前会被回收

* 弱引用：有用但非必须的对象，gc时被回收

* 虚引用：无法获取对象，用于gc时返回一个通知。

  

#### $垃圾回收有那些算法（GC方法）

* 标记-清除算法：标记出所有需要回收的对象，在标记完成后，统一清除被标记对象，也可以反过来。
  缺点：效率不稳定（对象存活率底时效率低）；清除之后会产生不连续的内存碎片
* 复制算法：将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把当前块的内存空间一次清理掉。
  优点：高效，无内存碎片。
  缺点：空间浪费（只用一半），复制开销（对象存活率高时）
* 标记-整理算法：标记出所有需要回收的对象，让所有存活的对象都向内存空间一端移动，然后直接清理掉边界以外的内存。
  优点：不浪费空间，无内存碎片
  缺点：移动存活对象开销（对象存活率高时）
* 分代收集理论：根据对象存活周期的不同将内存划分为几块，一般是新生代和老年代，新生代基本采用复制算法，老年代标记-清除算法或者标记-整理算法



#### JVM有那些垃圾回收器（GC具体实现）

Serial:	最早的单线程垃圾收集器。垃圾收集时会暂停其它所有线程，新生代用复制算法，老年代用标记-整理算法

ParNew：Serial的多线程版本。同上

Parallel Scavenge： 类似ParNew，基于复制算法的多线程收集器，以吞吐量为优先（运行代码时间/运行代码时间+垃圾回收时间）。

Serial Old：Serial的老年代版本，使用标记-整理算法

Parallel Old：Parallel Scavenge的老年代版本，使用标记-整理算法。

CMS：以最短回收停顿时间为目标的收集器。(缩短垃圾收集时用户线程的停顿时间)，使用标记清除算法。

G1（Garbage First）：面向服务端应用的垃圾收集器，兼容了吞吐量和停顿时间，是jdk9之后的默认gc选项



#### 新生代和老年代使用的垃圾收集器分类和区别

* 新生代收集器：Serial、ParNew、Parllel Scavenge
* 老年代收集器：Serial Old、Parlled Old、CMS
* 整堆收集器：G1

新生代收集器一般采用复制算法，老年代一般采用标记-整理或标记清除算法。



#### 详细介绍CMS垃圾回收器

以最短停顿时间为目标的收集器，适合那些对响应速度有要求的应用。整个过程分四步：初始标记、并发标记、重新标记、并发清除。初始标记仅标记GC Roots能直接关联到的对象；并发标记就是从这些直接关联对象开始遍历整个对象图；重新标记是为了修正那些在并发标记时发生标记变动的对象；最后通过并发清除回收对象。CMS基于标记-清除算法，在gc时会产生内存碎片；

理解：初始标记和重新标记需要暂停其它所有线程;
			并发标记和并发清除耗时长，但可与用户线程并发进行；
			最短停顿时间：GC时用户线程的等待时间；



#### 详细介绍G1垃圾回收器

面向服务端应用的垃圾收集器，兼容了吞吐量和停顿时间，是jdk9之后的默认gc选项。整个过程分四步：初始标记、并发标记、最终标记、筛选回收。
g1和cms区别：cms会一直执行下去、G1发现老年代没有存活的对象之后就会直接回收。



#### 垃圾回收的流程（内存分配和回收策略）（分代回收的具体过程）

对象优先在伊甸园区分配，当伊甸园区没有足够空间进行分配时，jvm发起一次轻GC，会把伊甸园区和from区中存活的对象移动到to区，然后清空伊甸园区和from区，为保证to区为空，form区和to区交换，从伊甸园区到to区的对象会初始化年龄为1，而从from区到to区的对象则年龄加一，当年龄达到15时，则将对象移入老年区。如果是大对象的话也会直接进入老年代，老年代当空间达到某个值时，就会触发重GC，一般使用标记整理算法处理。

注：分代回收具体分代

根据分代收集理论，堆可分为新生代和老年代。默认占比约1：2，新生代使用复制算法，有伊甸园区、To幸存区、From幸存区，三区默认占比为8：1：1。to区和from区是不断交换的，且to区一直为空。

注：jvm调优参数

* 新生代和老年代占比设置：-XX:NewRatio=2			1：2
* 伊甸园区和幸存区占比设置：-XX:SurvivorRatio=8		8:1
* 年龄设置：-XX:MaxTenuringThreshold=15		（默认15）
* -Xmn1m 设置新生代大小为1m
* -Xms10m  -Xmx10m  设置堆最小和最大空间为10m
* -XX:+HeapDumpOnOutOfMemoryError打印堆内存满时gc信息
* -XX:+PrintGCDetails  打印gc信息

注：了能更好地适应不同程序的内存状况，HotSpot虚拟机并不是永远要求对象的年龄必须达到-XX:MaxTenuringThreshold才能晋升老年代，如果在Survivor空间中，相同年龄对象的大小总超过Survivor的一半空间，则该年龄以上的对象就可以直接进入老年代，无须等到-XX:MaxTenuringThreshold中要求的年龄。

理解：为什么要划分

在Java堆划分出不同的区域之后，垃圾收集器才可以每次只回收其中某一个或者某些部分的区域 ——因而才有了“Minor GC” 轻gc，“Major GC”，“Full GC” 重gc这样的回收类型的划分；也才能够针对不同的区域安排与里面存储对象存亡特征相匹配的垃圾收集算法

#### 大对象为什么直接进入老年区

由于大对象需要分配大量连续内存空间，容易触发垃圾回收；而且大对象意味着在复制算法中，复制开销大



# 中间件



## Redis



### #说说你对Redis的了解

Redis是基于键值对的非关系型数据库，提供多种数据类型，支持数据持久化。

:zap:适合做缓存(例如热点数据，即访问量大的数据)，计数器（incr/decr），分布式锁（setnx），消息队列（list双向链表），统一存放不同服务器的会话信息（session共享）



#### #Redis和Memcache

* 数据类型：Memcache只支持字符串类型。Redis有...
* 持久化：Memcache不支持持久化，数据仅存于内存。Redis有...



#### #Redis支持的数据类型

Redis主要提供了5种数据结构：String字符串、Hash键值对集合、List双向链表、Set无序不重复集合、Zset有序不重复集合。 

:bulb:需要掌握简单的命令操作

#### #说说Redis的单线程架构

单线程一般指Redis的网络IO和键值对读写是由一个线程来完成的，但Redis采用了IO多路复用机制，使得Redis具备并发处理能力。而且Redis的其它功能，如持久化、异步删除、集群数据同步等操作都是依赖其他线程来执行。

:zap:单线程不仅实现简单，还可以避免线程切换和竞争所带来的消耗（为什么用单线程）。
:zap:Redis的大部分操作是在内存上完成的,这是它实现高性能的一个重要原因。(为什么快)
:bulb:zap说Redis是单线程的只是一种习惯的说法,事实上它的底层不是单线程的。



#### #Redis的持久化策略

#### RDB(Redis Database)

是Redis默认采用的持久化方式，以快照的形式将数据持久化到硬盘中。

##### 触发方式:

- 手动：save或`bgsave`命令

- 自动触发：配置redis.conf，让服务器在一定条件下自动执行bgsave命令

  ```
  stop-write-on-bgsave-error yes  磁盘无法写入时则关闭redis(例如磁盘满了)
  rdbcompression yes		rdb文件是否压缩
  save 20 3  				表示10秒内如果有大于等于3个key发生变化，就执行bgsave命令
  dbfilename dump.rdb  	rdb文件名
  dir ./					持久化文件保存路径，包括aof也是如此
  ```

  注：shutdown关闭服务，清空数据库时也会自动触发。

##### RDB原理:

save命令会让redis服务器阻塞，直到持久化过程结束。 bgsave命令则采用异步的方式生成快照，Redis首先会通过fork获得一个跟父进程一样的子进程，fork过程父进程会被阻塞，fork之后父进程可以继续响应其它命令，而子进程会将数据保存到临时文件，然后再用临时文件去替换dump.rdb

（bgsave底层：COW(copy on write)写时复制，通过复制一个副本并对副本修改，而不是直接修改原先的）

##### 优缺点:

rdb文件体积小，使得恢复数据速度快；但由于要创建子进程，很消耗内存，没法做到实时的持久化。

#### AOF(Append Only File)

以日志的方式（追加）记录每次的写操作(有修改就记录)。Reids重启时，通过重新执行这些命令来恢复数据。

保证数据持久化的实时性。

##### 使用：

配置redis.conf

```
appendonly yes  //开启，redis开启时优选读取的是aof文件，而不是rdb
appendfilename "appendonly.aof"  //aof文件名
```

##### 异常恢复：

文件损坏时，通过 `redis-check-aof --fix appendonly.aof `命令恢复（原理就是把报异常的命令包括后面的命令都去除）,

##### 持久化策略（同步频率）：

```
# 配置redis.conf
appendfsync always	   始终同步，有写入就记入日志
appendfsync everysec	每秒同步（默认）
appendfsync no	把同步时机交给操作系统
```

##### 重写策略（Rewire压缩）

aof采用文件追加的方式，会使文件越来越大，所以新增了重写机制，当aof文件的大小是上一次aof文件的2倍，且aof文件大于64mb，则进行压缩。

手动重写命令：bgrewriteaof

默认配置：

```
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

原理：使用rdb快照，以二进制的形式附在aof文件的头部

```
no-appendfsync-on-rewrite
```

##### 持久化流程：

写命令会被追加到AOF缓冲区；缓冲区根据AOF持久化策略将命令同步到磁盘AOF文件； 文件超过一定大小时，会进行文件压缩; Redis重启时，通过加载AOF文件恢复数据。



#### #主从同步（复制）

一台主机负责写操作，多台从机负责读操作，数据由主机同步到从机。
:zap:复制原理：从机启动成功连接到主机后会发送一个同步命令；主机收到命令后进行持久化操作，并发送持久化文件并发送给从机，从机接收到文件并读取（首次全量复制），发送期间，主机会在缓冲区中保存新执行的写命令，文件发送完就向从机发送这些写命令；之后主机一旦有写命令，就向从机发送。



#### #缓存穿透、击穿、雪崩

##### 缓存穿透

问题：向数据库请求不存在的数据，由于不存在，缓存也不会有，导致请求不通过缓存直接访问数据库。（漏洞攻击） 解决：

- 空值缓存：查询返回为空时也进行缓存
- 设置白名单：那些可访问，那些不行。
- 布隆过滤器：集合判断访问数据是否存在，不存在则拦截，同白名单
- 实时监控：

##### 缓存击穿

问题：缓存中存在请求数据，但key过期了，此时有大量并发请求该数据，可能导致数据库被瞬间压垮。（热点数据） 解决：

- 预先设置热门数据，延长缓存时间
- 实时调整过期时长
- 使用锁：排队请求，例如使用setnx设置锁（存在则返回失败），如果返回成功，放行去查数据库并缓存；如果返回失败，则让但前线程sleep一会再重试。

##### 缓存雪崩

问题：缓存中存在请求数据，但大量的key集中过期了，此时有大量并发请求该数据。类似缓存击穿，不同的是大过期key的数量。 解决：

- 使用锁和队列：同上
- 构建多级缓存：nginx缓存+redis缓存+等
- 设置标志更新缓存：记录key是否过期，过期的话则触发其它线程去更新缓存
- 分散缓存失效：对每个key的缓存失效时间增加1-5分钟的随机值，这样就很难引发大量key集中过期

#### #如何实现分布式锁

分布式的环境下,会发生多个server并发修改同一个资源的情况,这种情况下,由于多个server是多个不同的JRE环境,而Java自带的锁局限于当前JRE,所以Java自带的锁机制在这个场景下是无效的,那么就需要我们自己来实现一个分布式锁。 采用Redis实现分布式锁,我们可以在Redis中存一份代表锁的数据,数据格式通常使用字符串即可。 首先加锁的逻辑可以通过`setnx key value`来实现,但如果客户端忘记解锁,那么这种情况就很有可能造成死锁,但如果直接给锁增加过期时间即新增`expire key seconds`又会发生其他问题,即这两个命令并不是原子性的,那么如果第二步失败,依然无法避免死锁问题。考虑到如上问题,我们最终可以通过`set...nx...`命令,将加锁、过期命令编排到一起,把他们变成原子操作,这样就可以避免死锁。写法为`set key value nx ex seconds` 。 解锁就是将代表锁的那份数据删除,但不能用简单的`del key`,因为会出现一些问题。比如此时有进程A,如果进程A在任务没有执行完毕时,锁被到期释放了。这种情况下进程A在任务完成后依然会尝试释放锁,因为它的代码逻辑规定它在任务结束后释放锁,但是它的锁早已经被释放过了,那这种情况它释放的就可能是其他线程的锁。为解决这种情况,我们可以在加锁时为key赋一个随机值,来充当进程的标识,进程要记住这个标识。当进程解锁的时候进行判断,是自己持有的锁才能释放,否则不能释放。另外判断,释放这两步需要保持原子性,否则如果第二步失败,就会造成死锁。而获取和删除命令不是原子的,这就需要采用Lua脚本,通过Lua脚本将两个命令编排在一起,而整个Lua脚本的执行是原子的。综上所述,优化后的实现分布式锁命令如下： # 加锁 set key random-value nx ex seconds # 解锁 if redis.call("get",KEYS[1]) == ARGV[1] then return redis.call("del",KEYS[1]) else return 0 end 加分回答 上述的分布式锁实现方式是建立在单节点之上的,它可能存在一些问题,比如有一种情况,进程A在主节点加锁成功,但主节点宕机了,那么从节点就会晋升为主节点。那如果此时另一个进程B在新的主节点上加锁成功而原主节点重启了,成为了从节点,系统中就会出现两把锁,这违背了锁的唯一性原则。 总之,就是在单个主节点的架构上实现分布式锁,是无法保证高可用的。若要保证分布式锁的高可用,则可以采用多个节点的实现方案。这种方案有很多,而Redis的官方给出的建议是采用RedLock算法的实现方案。该算法基于多个Redis节点,它的基本逻辑如下： - 这些节点相互独立,不存在主从复制或者集群协调机制； - 加锁：以相同的KEY向N个实例加锁,只要超过一半节点成功,则认定加锁成功； - 解锁：向所有的实例发送DEL命令,进行解锁； 我们可以自己实现该算法,也可以直接使用Redisson框架。

:bulb:演化过程也指出了分布式锁的缺陷

#### #如何实现Redis高可用

##### ##集群

作用

当redis存储容量不够时，可通过集群扩容；当对服务器进行并发写操作时，可通过集群分担压力。

特点

无中心化的集群配置：任何一台机都可以作为入口访问，如果当前服务器不是处理该命令的，则切换到处理该命令的服务器。

原理

集群实现了对Redis的水平扩容，即启动N个Redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数据的1/N，通过分区，使得集群中若有部分节点失效，集群也能继续处理命令请求。

不足：多键操作需要添加组才可以操作；不支持事务；

服务器切换解释

slots--插槽，用于集群中保存键，有16384个，每个键都会对应一个插槽（哈希），可用数组去理解。每个服务器对应一部分插槽，每次添加的键通过计算落在那一部分插槽就切换到哪个服务。

##### ##哨兵

主机挂时，从机变主机这种操作肯定需要能自动触发，哨兵模式就是利用哨兵（也是服务器）监控后台主机是否故障，如果故障，就选择从机替代主机



#### #Redis数据淘汰策略

`maxmemory-policy  策略`

当Redis内存达到最大值`maxmemory`，会对数据进行淘汰，有6种策略，大致可分三种，默认策略noeviction，不淘汰数据；对设置了过期时间的数据，有volatile-lru，volatile-ttl，volatile-random，lru代表淘汰最近最少使用的数据，ttl代表淘汰将要过期的数据，random代表随机选择数据淘汰；对于所有数据，有allkeys-lru，allkeys-random，原则同上，只是数据范围不同。

:bulb:已设置过期时间的数据：server.db[i].expires	所有数据：server.db[i].dict
:bulb:最近最少使用(LRU)：不常用的
:zap:noeviction：达到最大内存时，再执行命令就报错



#### #如何保证缓存和数据库数据一致

* 缓存设置合理过期时间
* 增删改时，同步更新redis



#### #Redis如何做内存优化



牛客面试题考验

https://www.nowcoder.com/exam/interview







可以把任何一种数据类型的变量赋给Object类型的变量。基本类型会自动转包装类

从System.out.println(x)  该方法的参数类型是Object也可看出，

对于输出，总是先得到引用类型，再调用String.valueOf() 去调用该类的toString()

```
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}
```

## Nginx

### Nginx和Tomcat区别？

总的来说，Apache或者Nginx 是**HTTP Server**，Tomcat 则是一个Application Server也有人说是**Web Server**，但本质上它**是一个Servlet/JSP应用的容器**，顺便说一句，GlassFish以前版本的Servlet容器实现就直接用的Tomcat。一个 HTTP Server 关心的是 **HTTP 协议层面**的传输和访问控制，所以在 Apache/Nginx 上你可以看到**代理、负载均衡**等功能。客户端通过 HTTP Server 访问服务器上存储的资源（HTML 文件、图片文件等等）。通过 CGI 技术，也可以将处理过的内容通过 HTTP Server 分发，但是一个 HTTP Server 始终只是把服务器上的文件如实的通过 HTTP 协议传输给客户端。

而应用服务器，顾名思义是一个应用的容器。它首先需要支持开发语言的 Runtime环境（对于 Tomcat 来说，就是 Java），**保证应用能够在应用服务器上正常运行**。其次，需要支持应用相关的规范，例如类库、安全方面的特性。对于 Tomcat 来说，就是需要提供 JSP/Sevlet 运行需要的标准类库、Interface 等。为了方便，应用服务器往往也会集成 HTTP Server 的功能，但是不如专业的 HTTP Server 那么强大，所以应用服务器往往是运行在 HTTP Server 的背后，执行应用，将动态的内容转化为静态的内容之后，通过 HTTP Server 分发到客户端。

这也是为什么Nginx往往与Tomcat配合使用的原因。

### 



















