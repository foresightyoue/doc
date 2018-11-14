类的实例化顺序
- 先静态、先父后子
- 先静态：父静态 > 子静态
- 优先级：父类 > 子类 静态代码块 > 非静态代码块 > 构造函数

一个类的实例化过程：
- 1，父类中的static代码块，当前类的static
- 2，顺序执行父类的普通代码块
- 3，父类的构造函数
- 4，子类普通代码块
- 5，子类（当前类）的构造函数，按顺序执行
- 6，子类方法的执行

JVM内存分配

![image](https://img-blog.csdn.net/20170320154657018?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemQ4MzY2MTQ0Mzc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![image](https://img-blog.csdn.net/20170320154620080?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemQ4MzY2MTQ0Mzc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 Java 8的内存分代改进
 - 从永久代到元空间，在小范围自动扩展永生代避免溢出

JVM垃圾回收机制，何时触发MinorGC等操作
- 分代垃圾回收机制
    - 不同的对象生命周期不同。把不同生命周期的对象放在不同代上，不同代上采用最合适它的垃圾回收方式进行回收。
JVM中共划分为三个代：年轻代、年老代和持久代
    - 年轻代：存放所有新生成的对象；新生代的垃圾收集器命名为“minor gc”
    - 年老代：在年轻代中经历了N次垃圾回收仍然存活的对象，将被放到年老代中，故都是一些生命周期较长的对象；老生代的GC命名为”Full Gc 或者Major GC”.其中用System.gc()强制执行的是Full Gc.
    - 持久代：用于存放静态文件，如Java类、方法等。

- 判断对象是否需要回收的方法有两种  
   - 引用计数
     - 当某对象的引用数为0时，便可以进行垃圾收集。

  - 对象引用遍历
    - 某对象不能从这些根对象的一个（至少一个）到达，则将它作为垃圾收集。
    - 在对象遍历阶段，gc必须记住哪些对象可以到达，以便删除不可到达的对象，这称为标记（marking）对象。

- 触发GC（Garbage Collector）的条件：
  - GC在优先级最低的线程中运行，一般在应用程序空闲即没有应用线程在运行时被调用。
  - Java堆内存不足时，GC会被调用。


 jvm中一次完整的GC流程（从ygc到fgc）是怎样的，重点讲讲对象如何晋升到老年代等
  - 对象优先在新生代区中分配，若没有足够空间，Minor GC；
  - 大对象（需要大量连续内存空间）直接进入老年态；
  - 长期存活的对象进入老年态。
  - 如果对象在新生代出生并经过第一次MGC后仍然存活，年龄+1，若年龄超过一定限制（15），则被晋升到老年态。


你知道哪几种垃圾收集器，各自的优缺点，重点讲下cms，g1

![image](https://img-blog.csdn.net/20170320154529094?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemQ4MzY2MTQ0Mzc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


Eden和Survivor的比例分配等
 - 默认比例8:1
 - 大部分对象都是朝生夕死

复制算法的基本思想
 - 将内存分为两块，每次只用其中一块，当这一块内存用完，就将还活着的对象复制到另外一块上面
 - 复制算法不会产生内存碎片

类的加载
- 指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构。
- 类的加载的最终产品是位于堆区中的Class对象，Class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。

jvm内存结构

![image](http://www.ityouknow.com/assets/images/2017/jvm/structure.png)

类的生命周期
- 加载、连接、初始化、使用和卸载，其中前三部是类的加载的过程
![image](http://www.ityouknow.com/assets/images/2017/jvm/class.png)
- 加载，查找并加载类的二进制数据，在Java堆中也创建一个java.lang.Class类的对象
- 连接，连接又包含三块内容：验证、准备、初始化。1）验证，文件格式、元数据、字节码、符号引用验证；2）准备，为类的静态变量分配内存，并将其初始化为默认值；3）解析，把类中的符号引用转换为直接引用
- 初始化，为类的静态变量赋予正确的初始值
- 使用，new出对象程序中使用
- 卸载，执行垃圾回收


深入分析了Classloader，双亲委派机制
  - 类加载器（class loader）用来加载 Java 类到 Java 虚拟机中
  - Java 源程序（.java 文件）在经过 Java 编译器编译之后就被转换成 Java 字节代码（.class 文件）。类加载器负责读取 Java 字节代码，并转换成 java.lang.Class 类的一个实例。
  - 双亲委派机制
    - 某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；
    - 只有父类加载器无法完成此加载任务时，才自己去加载。

-  Bootstrap ClassLoader，负责加载存放在JDK\jre\lib(JDK代表JDK的安装目录，下同)下，或被-Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库（如rt.jar，所有的java.\*开头的类均被Bootstrap ClassLoader加载）。启动类加载器是无法被Java程序直接引用的。
- Extension ClassLoader，该加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载DK\jre\lib\ext目录中，或者由java.ext.dirs系统变量指定的路径中的所有类库（如javax.\*开头的类），开发者可以直接使用扩展类加载器。
- Application ClassLoader，该类加载器由sun.misc.Launcher$AppClassLoader来实现，它负责加载用户类路径（ClassPath）所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。
- 加入自定义的类加载器。因为JVM自带的ClassLoader只是懂得从本地文件系统加载标准的java class文件，因此如果编写了自己的ClassLoader，便可以做到如下几点：
 - 在执行非置信代码之前，自动验证数字签名。
 - 动态地创建符合用户特定需要的定制化构建类。
 - 从特定的场所取得java class，例如数据库中和网络中。


![image](https://img-blog.csdn.net/20170320154307607?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemQ4MzY2MTQ0Mzc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

JVM的编译优化

对Java内存模型的理解，以及其在并发中的应用
- Java内存模型的主要目标: 定义程序中各个变量的访问规则。
-  Java线程之间的通信由Java内存模型（本文简称为JMM）控制。
- 所有变量的存储都在主内存，每条线程还都有自己的工作内存
- 线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作必须在工作内存完成，而不能直接读取主内存中的变量
- 不同的线程直接无法访问对方工作内存中的变量，线程间变量的传递均需要通过主内存来完成。

![image](https://img-blog.csdn.net/20170320154451607?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemQ4MzY2MTQ0Mzc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

- 线程间通信
 - 首先，线程A把本地内存A中更新过的共享变量刷新到主内存中去。
 - 然后，线程B到主内存中去读取线程A之前已更新过的共享变量。

指令重排序，内存栅栏等
 - 指令重排序
  - 编译器或运行时环境为了优化程序性能而采取的对指令进行重新排序执行的一种手段
  - 在单线程程序中，对存在控制依赖的操作重排序，不会改变执行结果
  - 在多线程程序中，对存在控制依赖的操作重排序，可能会改变程序的执行结果

OOM错误，stackoverflow错误，permgen space错误

JVM常用参数
- 堆设置、回收器选择（串行、并行、并发收集器）

tomcat结构，类加载器流程

 ![image](https://img-blog.csdn.net/20170320154346997?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemQ4MzY2MTQ0Mzc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 volatile的语义，它修饰的变量一定线程安全吗
 - 保证此变量对所有线程的可见性，即当一条线程修改了这个值，新值对于其他所有线程来说是立即得知的，普通变量需要通过主内存传递。
 - 禁止指令重排序优化。
 - Volatile修饰的变量不一定是线程安全的，eg非原子操作a++等


 g1和cms区别,吞吐量优先和响应优先的垃圾收集器选择
 - CMS收集器：一款以获取最短回收停顿时间为目标的收集器，是基于“标记-清除”算法实现的，分为4个步骤：初始标记、并发标记、重新标记、并发清除。
 - G1收集器：面向服务端应用的垃圾收集器，过程：初始标记；并发标记；最终标记；筛选回收。整体上看是“标记-整理”，局部看是“复制”，不会产生内存碎片。
 - 吞吐量优先的并行收集器：以到达一定的吞吐量为目标，适用于科学技术和后台处理等。
 - 响应时间优先的并发收集器：保证系统的响应时间，减少垃圾收集时的停顿时间。适用于应用服务器、电信领域等。


 说一说你对环境变量classpath的理解？如果一个类不在classpath下，为什么会抛出ClassNotFoundException异常，如果在不改变这个类路径的前期下，怎样才能正确加载这个类？
- classpath是javac编译器的一个环境变量。它的作用与import、package关键字有关。
- package的所在位置，就是设置CLASSPATH当编译器面对import packag这个语句时，它先会查找CLASSPATH所指定的目录，并检视子目录java/util是否存在，然后找出名称吻合的已编译文件（.class文件）。如果没有找到就会报错！
- 动态加载包

强引用、软引用、弱引用、虚引用以及他们之间和gc的关系
- 强引用：new出的对象之类的引用，只要强引用还在，永远不会回收
- 软引用：引用但非必须的对象，内存溢出异常之前，回收
- 弱引用：非必须的对象，对象能生存到下一次垃圾收集发生之前。
- 虚引用：对生存时间无影响，在垃圾回收时得到通知。
