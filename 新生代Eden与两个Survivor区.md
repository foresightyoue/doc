 HotSpot JVM把年轻代分为了三部分：1个Eden区和2个Survivor区（分别叫from和to）

 默认比例为8：1
  - 新创建的对象都会被分配到Eden区(一些大对象特殊处理)
  - 这些对象经过第一次Minor GC后，如果仍然存活，将会被移到Survivor区。
  - 对象在Survivor区中每熬过一次Minor GC，年龄就会增加1岁，当它的年龄增加到一定程度时，就会被移动到年老代中。

年轻代中的对象基本都是朝生夕死的(80%以上)，所以在年轻代的垃圾回收算法使用的是复制算法
- 基本思想就是将内存分为两块，每次只用其中一块，当这一块内存用完，就将还活着的对象复制到另外一块上面
- 复制算法不会产生内存碎片

在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor区“To”是空的

紧接着进行GC，Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。

年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。

经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”。

不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中。

![image](http://ifeve.com/wp-content/uploads/2014/07/young_gc.png)


有关年轻代的JVM参数
- -XX:NewSize和-XX:MaxNewSize
 - 用于设置年轻代的大小，建议设为整个堆大小的1/3或者1/4,两个值设为一样大。
- -XX:SurvivorRatio
  - 设置Eden和其中一个Survivor的比值，这个值也比较重要。
- -XX:+PrintTenuringDistribution
  - 显示每次Minor GC时Survivor区中各个年龄段的对象的大小。
- -XX:InitialTenuringThreshol和-XX:MaxTenuringThreshold
 - 设置晋升到老年代的对象年龄的最小值和最大值，每个对象在坚持过一次Minor GC之后，年龄就加1
