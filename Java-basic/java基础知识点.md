## Java-Basic ## 
### 谈谈final、 finally、 finalize有什么不同？
**典型回答:**
final可以用来修饰类、方法、变量，分别有不同的意义， final修饰的class代表不可以继承扩展， final的变量是不可以修改的，而final的方法也是不可以重写的（ override）。

finally则是Java保证重点代码一定要被执行的一种机制。我们可以使用try-finally或者try-catch-finally来进行类似关闭JDBC连接、保证unlock锁等动作。

finalize是基础类java.lang.Object的一个方法，它的设计目的是保证对象在被垃圾收集前完成特定资源的回收。 finalize机制现在已经不推荐使用，并且在JDK 9开始被标记
为deprecated。 


### 强引用、软引用、弱引用、幻象引用有什么区别？具体使用场景是什么？
不同的引用类型，主要体现的是对象不同的可达性（ reachable）状态和对垃圾收集的影响。

所谓强引用（ "Strong" Reference），就是我们最常见的普通对象引用，只要还有强引用指向一个对象，就能表明对象还“活着”，垃圾收集器不会碰这种对象。对于一个普通的对
象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为null，就是可以被垃圾收集的了，当然具体回收时机还是要看垃圾收集策略。

软引用（ SoftReference），是一种相对强引用弱化一些的引用，可以让对象豁免一些垃圾收集，只有当JVM认为内存不足时，才会去试图回收软引用指向的对象。 JVM会确保在抛
出OutOfMemoryError之前，清理软引用指向的对象。软引用通常用来实现内存敏感的缓存，如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓
存的同时，不会耗尽内存。
andriod中的图片缓存是软引用的例子.

弱引用（ WeakReference）并不能使对象豁免垃圾收集，仅仅是提供一种访问在弱引用状态下对象的途径。这就可以用来构建一种没有特定约束的关系，比如，维护一种非强制性
的映射关系，如果试图获取时对象还在，就使用它，否则重现实例化。它同样是很多缓存实现的选择。
ThreadLocal中entry的Key是弱引用的例子.

对于幻象引用，有时候也翻译成虚引用，你不能通过它访问对象。幻象引用仅仅是提供了一种确保对象被fnalize以后，做某些事情的机制，比如，通常用来做所谓的PostMortem清理机制，我在专栏上一讲中介绍的Java平台自身Cleaner机制等，也有人利用幻象引用监控对象的创建和销毁。


### 理解Java的字符串， String、 StringBufer、 StringBuilder有什么区别？
String是Java语言非常基础和重要的类，提供了构造和管理字符串的各种基本逻辑。它是典型的Immutable类，被声明成为fnal class，所有属性也都是fnal的。也由于它的不可
变性，类似拼接、裁剪字符串等动作，都会产生新的String对象。由于字符串操作的普遍性，所以相关操作的效率往往对应用性能有明显影响。

StringBufer是为解决上面提到拼接产生太多中间对象的问题而提供的一个类，我们可以用append或者add方法，把字符串添加到已有序列的末尾或者指定位置。 StringBufer本
质是一个线程安全的可修改字符序列，它保证了线程安全，也随之带来了额外的性能开销，所以除非有线程安全的需要，不然还是推荐使用它的后继者，也就是StringBuilder。

StringBuilder是Java 1.5中新增的，在能力上和StringBufer没有本质区别，但是它去掉了线程安全的部分，有效减小了开销，是绝大部分情况下进行字符串拼接的首选。

String是Immutable类的典型实现，原生的保证了基础线程安全，因为你无法对它内部数据进行任何修改，这种便利甚至体现在拷贝构造函数中，由于不可
变， Immutable对象在拷贝时不需要额外复制数据。

为了实现修改字符序列的目的， StringBufer和StringBuilder底层都是利用可修改的（ char， JDK 9以后是byte）数组，二者都继承了AbstractStringBuilder，里面包含了基本
操作，区别仅在于最终的方法是否加了synchronized。

### 谈谈Java反射机制，动态代理是基于什么原理？
**典型回答:**
反射机制是Java语言提供的一种基础功能，赋予程序在运行时自省（ introspect，官方用语）的能力。通过反射我们可以直接操作类或者对象，比如获取某个对象的类定义，获取类
声明的属性和方法，调用方法或者构造对象，甚至可以运行时修改类定义。

动态代理是一种方便运行时动态构建代理、动态处理代理方法调用的机制，很多场景都是利用类似机制做到的，比如用来包装RPC调用、面向切面的编程（ AOP）。

实现动态代理的方式很多，比如JDK自身提供的动态代理，就是主要利用了上面提到的反射机制。还有其他的实现方式，比如利用传说中更高性能的字节码操作机制，类
似ASM、 cglib（基于ASM）、 Javassist等。

我们知道Spring AOP支持两种模式的动态代理， JDK Proxy或者cglib，如果我们选择cglib方式，你会发现对接口的依赖被克服了。

cglib动态代理采取的是创建目标类的子类的方式，因为是子类化，我们可以达到近似使用被调用者本身的效果。


### int和Integer有什么区别？谈谈Integer的值缓存范围。
**典型回答:**
int是我们常说的整形数字，是Java的8个原始数据类型（ Primitive Types， boolean、 byte 、 short、 char、 int、 foat、 double、 long）之一。 Java语言虽然号称一切都是对象，
但原始数据类型是例外。

Integer是int对应的包装类，它有一个int类型的字段存储数据，并且提供了基本操作，比如数学运算、 int和字符串之间转换等。在Java 5中，引入了自动装箱和自动拆箱功能
（ boxing/unboxing）， Java可以根据上下文，自动进行转换，极大地简化了相关编程。

关于Integer的值缓存，这涉及Java 5中另一个改进。构建Integer对象的传统方式是直接调用构造器，直接new一个对象。但是根据实践，我们发现大部分数据操作都是集中在有
限的、较小的数值范围，因而，在Java 5中新增了静态工厂方法valueOf，在调用它的时候会利用一个缓存机制，带来了明显的性能改进。按照Javadoc， 这个值默认缓存
是-128到127之间。

这种缓存机制并不是只有Integer才有，同样存在于其他的一些包装类，比如：
- Boolean，缓存了true/false对应实例，确切说，只会返回两个常量实例Boolean.TRUE/FALSE。
- Short，同样是缓存了-128到127之间的数值。
- Byte，数值有限，所以全部都被缓存。
- Character，缓存范围'\u0000' 到 '\u007F'。

**注意事项:**

[1] 基本类型均具有取值范围，在大数*大数的时候，有可能会出现越界的情况。

[2] 基本类型转换时，使用声明的方式。例： int result= 1234567890 * 24 * 365；结果值一定不会是你所期望的那个值，因为1234567890 * 24已经超过了int的范围，如果修改为： long result= 1234567890L * 24 * 365；就正常了。

[3] 慎用基本类型处理货币存储。如采用double常会带来差距，常采用BigDecimal、整型（如果要精确表示分，可将值扩大100倍转化为整型）解决该问题。

[4] 优先使用基本类型。原则上，建议避免无意中的装箱、拆箱行为，尤其是在性能敏感的场合，

[5] 如果有线程安全的计算需要，建议考虑使用类型AtomicInteger、 AtomicLong 这样的线程安全类。部分比较宽的基本数据类型，比如 foat、 double，甚至不能保证更新操作的原子性，
可能出现程序读取到只更新了一半数据位的数值。



[4].原则上， **建议避免无意中的装箱、拆箱行为**，尤其是在性能敏感的场合，创建10万个Java对象和10万个整数的开销可不是一个数量级的，不管是内存使用还是处理速度，光是对象头
的空间占用就已经是数量级的差距了。

以我们经常会使用到的计数器实现为例，下面是一个常见的线程安全计数器实现。
```java
class Counter {
	private fnal AtomicLong counter = new AtomicLong();
		public void increase() {
		counter.incrementAndGet();
	}
}
```

如果利用原始数据类型，可以将其修改为
```java
class CompactCounter {
	private volatile long counter;
	private satic fnal AtomicLongFieldUpdater<CompactCounter> updater = AtomicLongFieldUpdater.newUpdater(CompactCounter.class, "counter");
	public void increase() {
		updater.incrementAndGet(this);
	}
}
```

**Java原始数据类型和引用类型局限性:**

前面我谈了非常多的技术细节，最后再从Java平台发展的角度来看看，原始数据类型、对象的局限性和演进。
对于Java应用开发者，设计复杂而灵活的类型系统似乎已经习以为常了。但是坦白说，毕竟这种类型系统的设计是源于很多年前的技术决定，现在已经逐渐暴露出了一些副作用，例
如：
- 原始数据类型和Java泛型并不能配合使用
这是因为Java的泛型某种程度上可以算作伪泛型，它完全是一种编译期的技巧， Java编译期会自动将类型转换为对应的特定类型，这就决定了使用泛型，必须保证相应类型可以转换
为Object。

- 无法高效地表达数据，也不便于表达复杂的数据结构，比如vector和tuple我们知道Java的对象都是引用类型，如果是一个原始数据类型数组，它在内存里是一段连续的内存，而对象数组则不然，数据存储的是引用，对象往往是分散地存储在堆的不同位
置。这种设计虽然带来了极大灵活性，但是也导致了数据操作的低效，尤其是无法充分利用现代CPU缓存机制。


### Vector、 ArrayList、 LinkedList有何区别？
**典型回答:**
Vector是Java早期提供的线程安全的动态数组，如果不需要线程安全，并不建议选择，毕竟同步是有额外开销的。 Vector内部是使用对象数组来保存数据，可以根据需要自动的增加
容量，当数组已满时，会创建新的数组，并拷贝原有数组数据。

ArrayList是应用更加广泛的动态数组实现，它本身不是线程安全的，所以性能要好很多。与Vector近似， ArrayList也是可以根据需要调整容量，不过两者的调整逻辑有所区
别， Vector在扩容时会提高1倍，而ArrayList则是增加50%。

LinkedList顾名思义是Java提供的双向链表，所以它不需要像上面两种那样调整容量，它也不是线程安全的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190930143507252.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

我们可以看到Java的集合框架， Collection接口是所有集合的根，然后扩展开提供了三大类集合，分别是：
- List，也就是我们前面介绍最多的有序集合，它提供了方便的访问、插入、删除等操作。
- Set， Set是不允许重复元素的，这是和List最明显的区别，也就是不存在两个对象equals返回true。我们在日常开发中有很多需要保证元素唯一性的场合。
- Queue/Deque，则是Java提供的标准队列结构的实现，除了集合的基本功能，它还支持类似先入先出（ FIFO， First-in-First-Out）或者后入先出（ LIFO， Last-In-FirstOut）等特定行为。这里不包括BlockingQueue，因为通常是并发编程场合，所以被放置在并发包里。

今天介绍的这些集合类，都不是线程安全的，对于java.util.concurrent里面的线程安全容器，我在专栏后面会去介绍。但是，并不代表这些集合完全不能支持并发编程的场景，
在Collections工具类中，提供了一系列的synchronized方法，比如
```java
static <T> List<T> synchronizedList(List<T> list)
```
我们完全可以利用类似方法来实现基本的线程安全集合：
```java
List list = Collections.synchronizedList(new ArrayList());
```
它的实现，基本就是将每个基本方法，比如get、 set、 add之类，都通过synchronizd添加基本的同步支持，非常简单粗暴，但也非常实用。注意这些方法创建的线程安全集合，都
符合迭代时fail-fast行为，当发生意外的并发修改时，尽早抛出ConcurrentModifcationException异常，以避免不可预计的行为。

另外一个经常会被考察到的问题，就是理解Java提供的默认排序算法，具体是什么排序方式以及设计思路等。

这个问题本身就是有点陷阱的意味，因为需要区分是Arrays.sort()还是Collections.sort() （底层是调用Arrays.sort()）；什么数据类型；多大的数据集（太小的数据集，复杂排
序是没必要的， Java会直接进行二分插入排序）等。

对于原始数据类型，目前使用的是所谓双轴快速排序（ Dual-Pivot QuickSort），是一种改进的快速排序算法，早期版本是相对传统的快速排序，你可以阅读源码。

而对于对象数据类型，目前则是使用TimSort，思想上也是一种归并和二分插入排序（ binarySort）结合的优化排序算法。 TimSort并不是Java的独创，简单说它的思路是查找

数据集中已经排好序的分区（这里叫run），然后合并这些分区来达到排序的目的。

另外， Java 8引入了并行排序算法（直接使用parallelSort方法），这是为了充分利用现代多核处理器的计算能力，底层实现基于fork-join框架，当处理的数据集比较小的时候，差距不明显，甚至还表现差一点；但是，当数据集增长到数万或百万以上时，提高就非常大了，具体还是取决于处理器和系统环境。



### Hashtable、 HashMap、 TreeMap有什么不同？ 
**典型回答:**
Hashtable、 HashMap、 TreeMap都是最常见的一些Map实现，是以键值对的形式存储和操作数据的容器类型。

Hashtable是早期Java类库提供的一个哈希表实现，本身是同步的，不支持null键和值，由于同步导致的性能开销，所以已经很少被推荐使用。

HashMap是应用更加广泛的哈希表实现，行为上大致上与HashTable一致，主要区别在于HashMap不是同步的，支持null键和值等。通常情况下， HashMap进行put或者get操
作，可以达到常数时间的性能，所以它是绝大部分利用键值对存取场景的首选，比如，实现一个用户ID和用户信息对应的运行时存储结构。

TreeMap则是基于红黑树的一种提供顺序访问的Map，和HashMap不同，它的get、 put、 remove之类操作都是O（ log(n)）的时间复杂度，具体顺序可以由指定
的Comparator来决定，或者根据键的自然顺序来判断。


![在这里插入图片描述](https://img-blog.csdnimg.cn/2019093015421066.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

Hashtable比较特别，作为类似Vector、 Stack的早期集合相关类型，它是扩展了Dictionary类的，类结构上与HashMap之类明显不同。

HashMap等其他Map实现则是都扩展了AbstractMap，里面包含了通用方法抽象。不同Map的用途，从类图结构就能体现出来，设计目的已经体现在不同接口上。

大部分使用Map的场景，通常就是放入、访问或者删除，而对顺序没有特别要求， HashMap在这种情况下基本是最好的选择。


** HashMap源码分析:**
前面提到， HashMap设计与实现是个非常高频的面试题，所以我会在这进行相对详细的源码解读，主要围绕：
- HashMap内部实现基本点分析。
- 容量（ capcity）和负载系数（ load factor）。
- 树化 。

首先，我们来一起看看HashMap内部的结构，它可以看作是数组（ Node[] table）和链表结合组成的复合结构，数组被分为一个个桶（ bucket），通过哈希值决定了键值对在这个
数组的寻址；哈希值相同的键值对，则以链表形式存储，你可以参考下面的示意图。这里需要注意的是，如果链表大小超过阈值（ TREEIFY_THRESHOLD, 8），图中的链表就会被
改造为树形结构。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190930155011802.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)




















