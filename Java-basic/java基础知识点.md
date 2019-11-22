## Java-Basic ## 
<br/>

### 谈谈final、 finally、 finalize有什么不同？
**典型回答:**
final可以用来修饰类、方法、变量，分别有不同的意义， final修饰的class代表不可以继承扩展， final的变量是不可以修改的，而final的方法也是不可以重写的（ override）。

finally则是Java保证重点代码一定要被执行的一种机制。我们可以使用try-finally或者try-catch-finally来进行类似关闭JDBC连接、保证unlock锁等动作。

finalize是基础类java.lang.Object的一个方法，它的设计目的是保证对象在被垃圾收集前完成特定资源的回收。 finalize机制现在已经不推荐使用，并且在JDK 9开始被标记
为deprecated。 

<br/>

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

<br/>

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

<br/>

### 谈谈Java反射机制，动态代理是基于什么原理？
**典型回答:**
反射机制是Java语言提供的一种基础功能，赋予程序在运行时自省（ introspect，官方用语）的能力。通过反射我们可以直接操作类或者对象，比如获取某个对象的类定义，获取类
声明的属性和方法，调用方法或者构造对象，甚至可以运行时修改类定义。

动态代理是一种方便运行时动态构建代理、动态处理代理方法调用的机制，很多场景都是利用类似机制做到的，比如用来包装RPC调用、面向切面的编程（ AOP）。

实现动态代理的方式很多，比如JDK自身提供的动态代理，就是主要利用了上面提到的反射机制。还有其他的实现方式，比如利用传说中更高性能的字节码操作机制，类
似ASM、 cglib（基于ASM）、 Javassist等。

我们知道Spring AOP支持两种模式的动态代理， JDK Proxy或者cglib，如果我们选择cglib方式，你会发现对接口的依赖被克服了。

cglib动态代理采取的是创建目标类的子类的方式，因为是子类化，我们可以达到近似使用被调用者本身的效果。

<br/>

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

[5] 如果有线程安全的计算需要，建议考虑使用类型AtomicInteger、 AtomicLong 这样的线程安全类。部分比较宽的基本数据类型，比如 float、 double，甚至不能保证更新操作的原子性，
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

<br/>

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

<br/>

### 对比Hashtable、 HashMap、 TreeMap有什么不同？

**典型回答**
Hashtable、 HashMap、 TreeMap都是最常见的一些Map实现，是以键值对的形式存储和操作数据的容器类型。

Hashtable是早期Java类库提供的一个哈希表实现，本身是同步的，**不支持null键和值**，由于同步导致的性能开销，所以已经很少被推荐使用。

HashMap是应用更加广泛的哈希表实现，行为上大致上与HashTable一致，主要区别在于HashMap不是同步的，支持null键和值等。通常情况下， HashMap进行put或者get操作，可以达到常数时间的性能，所以它是绝大部分利用键值对存取场景的首选，比如，实现一个用户ID和用户信息对应的运行时存储结构。

TreeMap则是基于红黑树的一种提供顺序访问的Map，和HashMap不同，它的get、 put、 remove之类操作都是O（log(n)）的时间复杂度，具体顺序可以由指定
的Comparator来决定，或者根据键的自然顺序来判断。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191006145504941.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

LinkedHashMap通常提供的是遍历顺序符合插入顺序，它的实现是通过为条目（键值对）维护一个双向链表。注意，通过特定构造函数，我们可以创建反映访问顺序的实例，所
谓的put、 get、 compute等，都算作“访问”。

对于TreeMap，它的整体顺序是由键的顺序关系决定的，通过Comparator或Comparable（自然顺序）来决定。

**HashMap：**
而对于负载因子，我建议：
- 如果没有特别需求，不要轻易进行更改，因为JDK自身的默认负载因子是非常符合通用场景的需求的。
- 如果确实需要调整，建议不要设置超过0.75的数值，因为会显著增加冲突，降低HashMap的性能。
- 如果使用太小的负载因子，按照上面的公式，预设容量值也进行调整，否则可能会导致更加频繁的扩容，增加无谓的开销，本身访问性能也会受影响。


**那么，为什么HashMap要树化呢？**
本质上这是个**安全**问题。 因为在元素放置过程中，如果一个对象哈希冲突，都被放置到同一个桶里，则会形成一个链表，我们知道链表查询是线性的，会严重影响存取的性能。而在现实世界，构造哈希冲突的数据并不是非常复杂的事情，恶意代码就可以利用这些数据大量与服务器端交互，导致服务器端CPU大量占用，这就构成了哈希碰撞拒绝服务攻击，国内一线互联网公司就发生过类似攻击事件。

**Hashtable、 HashMap、 TreeMap比较：**
三者均实现了Map接口，存储的内容是基于key-value的键值对映射，一个映射不能有重复的键，一个键最多只能映射一个值。
（1） 元素特性
HashTable中的key、 value都不能为null； HashMap中的key、 value可以为null，很显然只能有一个key为null的键值对，但是允许有多个值为null的键值对； TreeMap中当未实现Comparator 接口时， key 不可以为null；当实现 Comparator 接口时，若未对null情况进行判断，则key不可以为null，反之亦然。
（2）顺序特性
HashTable、 HashMap具有无序特性。 TreeMap是利用红黑树来实现的（树中的每个节点的值，都会大于或等于它的左子树种的所有节点的值，并且小于或等于它的右子树中的所有节点的
值），实现了SortMap接口，能够对保存的记录根据键进行排序。所以一般需要排序的情况下是选择TreeMap来进行，默认为升序排序方式（深度优先搜索），可自定义实现Comparator接口
实现排序方式。
（3）初始化与增长方式
初始化时： HashTable在不指定容量的情况下的默认容量为11，且不要求底层数组的容量一定要为2的整数次幂； HashMap默认容量为16，且要求容量一定为2的整数次幂。
扩容时： **Hashtable将容量变为原来的2倍加1； HashMap扩容将容量变为原来的2倍**。
（4）线程安全性
HashTable其方法函数都是同步的（采用synchronized修饰），不会出现两个线程同时对数据进行操作的情况，因此保证了线程安全性。也正因为如此，在多线程运行环境下效率表现非常低下。因为当一个线程访问HashTable的同步方法时，其他线程也访问同步方法就会进入阻塞状态。比如当一个线程在添加数据时候，另外一个线程即使执行获取其他数据的操作也必须被阻塞，大大降低了程序的运行效率，在新版本中已被废弃，不推荐使用。
HashMap不支持线程的同步，即任一时刻可以有多个线程同时写HashMap;可能会导致数据的不一致。如果需要同步（1）可以用 Collections的synchronizedMap方法；（2）使用ConcurrentHashMap类，相较于HashTable锁住的是对象整体， ConcurrentHashMap基于lock实现锁分段技术。首先将Map存放的数据分成一段一段的存储方式，然后给每一段数据分配一把锁，当一个线程占用锁访问其中一个段的数据时，其他段的数据也能被其他线程访问。 ConcurrentHashMap不仅保证了多线程运行环境下的数据访问安全性，而且性能上有长足的提升。
(5)一段话HashMap
HashMap基于哈希思想，实现对数据的读写。当我们将键值对传递给put()方法时，它调用键对象的hashCode()方法来计算hashcode，让后找到bucket位置来储存值对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。 HashMap使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。 HashMap在每个链表节点中储存键值对对象。当两个不同的键对象的hashcode相同时，它们会储存在同一个bucket位置的链表中，可通过键对象的equals()方法用来找到键值对。如果链表大小超过阈值（TREEIFY_THRESHOLD, 8），链表就会被改造为树形结构。

<br/>

### 如何保证集合是线程安全的? ConcurrentHashMap如何实现高效地线程安全？
**典型回答**:
Java提供了不同层面的线程安全支持。在传统集合框架内部，除了Hashtable等同步容器，还提供了所谓的同步包装器（Synchronized Wrapper），我们可以调用Collections工具类提供的包装方法，来获取一个同步的包装容器（如Collections.synchronizedMap），但是它们都是利用非常粗粒度的同步方式，在高并发情况下，性能比较低下。
另外，更加普遍的选择是利用并发包提供的线程安全容器类，它提供了：

- 各种并发容器，比如ConcurrentHashMap、 CopyOnWriteArrayList。
- 各种线程安全队列（Queue/Deque），如ArrayBlockingQueue、 SynchronousQueue。
- 各种有序容器的线程安全版本等。
- 
具体保证线程安全的方式，包括有从简单的synchronize方式，到基于更加精细化的，比如基于分离锁实现的ConcurrentHashMap等并发实现等。具体选择要看开发的场景需求，
总体来说，并发包内提供的容器通用场景，远优于早期的简单同步实现。

**知识扩展**
**1.为什么需要ConcurrentHashMap？**
Hashtable本身比较低效，因为它的实现基本就是将put、 get、 size等各种方法加上“synchronized”。简单来说，这就导致了所有并发操作都要竞争同一把锁，一个线程在进行同步操作时，其他线程只能等待，大大降低了并发操作的效率。

前面已经提过HashMap不是线程安全的，并发情况会导致类似CPU占用100%等一些问题，那么能不能利用Collections提供的同步包装器来解决问题呢？

看看下面的代码片段，我们发现**同步包装器只是利用输入Map构造了另一个同步版本，所有操作虽然不再声明成为synchronized方法，但是还是利用了“this”作为互斥的mutex，没有真正意义上的改进！**

```java
private static class SynchronizedMap<K,V> implements Map<K,V>, Serializable {
	private final Map<K,V> m; // Backing Map
	final Object mutex; // Object on which to synchronize
	// …
	public int size() {
	synchronized (mutex) {return m.size();}
	}
	// …
}
```
所以， Hashtable或者同步包装版本，都只是适合在非高度并发的场景下。

**2.ConcurrentHashMap分析**
我们再来看看ConcurrentHashMap是如何设计实现的，为什么它能大大提高并发效率。
首先，我这里强调， ConcurrentHashMap的设计实现其实一直在演化，比如在Java 8中就发生了非常大的变化（Java 7其实也有不少更新），所以，我这里将比较分析结构、实现机制等方面，对比不同版本的主要区别。

**早期ConcurrentHashMap**，其实现是基于：
- 分段锁，也就是将内部进行分段（Segment），里面则是HashEntry的数组，和HashMap类似，哈希相同的条目也是以链表形式存放。
- HashEntry内部使用volatile的value字段来保证可见性，也利用了不可变对象的机制以改进利用Unsafe提供的底层能力，比如volatile access，去直接完成部分操作，以最优化性能，毕竟Unsafe中的很多操作都是JVM intrinsic优化过的。

**ConcurrentHashMap 1.7中的get操作：get操作需要保证的是可见性，所以并没有什么同步逻辑。**
**get**:
```java
public V get(Object key) {
	Segment<K,V> s; // manually integrate access methods to reduce overhead
	HashEntry<K,V>[] tab;
	int h = hash(key.hashCode());
	//利用位操作替换普通数学运算
	long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
	// 以Segment为单位，进行定位
	// 利用Unsafe直接进行volatile access
	if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
	(tab = s.table) != null) {
		//省略
	}
	return null;
}
```
**put**:而对于put操作，首先是通过二次哈希避免哈希冲突，然后以Unsafe调用方式，直接获取相应的Segment，然后进行线程安全的put操作：

```java
public V put(K key, V value) {
	Segment<K,V> s;
	if (value == null)
		throw new NullPointerException();
	// 二次哈希，以保证数据的分散性，避免哈希冲突
	int hash = hash(key.hashCode());
	int j = (hash >>> segmentShift) & segmentMask;
	if ((s = (Segment<K,V>)UNSAFE.getObject // nonvolatile; recheck
		(segments, (j << SSHIFT) + SBASE)) == null) // in ensureSegment
		s = ensureSegment(j);
	return s.put(key, hash, value, false);
}
```
所以，从上面的源码清晰的看出，在进行并发写操作时：
- ConcurrentHashMap会获取再入锁，以保证数据一致性， Segment本身就是基于ReentrantLock的扩展实现，所以，在并发修改期间，相应Segment是被锁定的。
- 在最初阶段，进行重复性的扫描，以确定相应key值是否已经在数组里面，进而决定是更新还是放置操作，你可以在代码里看到相应的注释。重复扫描、检测冲突
是ConcurrentHashMap的常见技巧。
- HashMap可能发生扩容问题，在ConcurrentHashMap中同样存在。不过有一个明显区别，就是它进行的不是整体的扩容，而是单独对Segment进行扩容。

**size:**
分段计算两次，两次结果相同则返回，否则对所以段加锁重新计算

**在Java 8和之后的版本中， ConcurrentHashMap发生了哪些变化呢？**
- 总体结构上，**它的内部存储变得和HashMap结构非常相似，同样是大的桶（bucket）数组，然后内部也是一个个所谓的链表结构（bin），同步的粒度要更细致一些。**
- 其**内部仍然有Segment定义，但仅仅是为了保证序列化时的兼容性而已，不再有任何结构上的用处。**
- 因为不再使用Segment，初始化操作大大简化，修改为lazy-load形式，这样可以有效避免初始开销，解决了老版本很多人抱怨的这一点。
- 数据存储利用volatile来保证可见性。
- 使用CAS等操作，在特定场景进行无锁并发操作。
- 使用Unsafe、 LongAdder之类底层手段，进行极端情况的优化。

1.8 中，数据存储内部实现，我们可以发现**Key是final的，因为在生命周期中，一个条目的Key发生变化是不可能的；与此同时val，则声明为volatile，以保证可见性**。

```java
static class Node<K,V> implements Map.Entry<K,V> {
	final int hash;
	final K key;
	volatile V val;
	volatile Node<K,V> next;
	// …
}
```
**put：**

```java
final V putVal(K key, V value, boolean onlyIfAbsent) { 
	if (key == null || value == null) throw new NullPointerException();
	int hash = spread(key.hashCode());
	int binCount = 0;
	for (Node<K,V>[] tab = table;;) {
	Node<K,V> f; int n, i, fh; K fk; V fv;
	if (tab == null || (n = tab.length) == 0)
		tab = initTable(); //初始化
	else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
		// 利用CAS去进行无锁线程安全操作，如果bin是空的
		if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
		break;
	}
	else if ((fh = f.hash) == MOVED)
		tab = helpTransfer(tab, f);
	else if (onlyIfAbsent // 不加锁，进行检查
			&& fh == hash
			&& ((fk = f.key) == key || (fk != null && key.equals(fk)))
			&& (fv = f.val) != null)
		return fv;
	else {
		V oldVal = null;
			synchronized (f) {
			// 细粒度的同步修改操作...
			}
		}
		// Bin超过阈值，进行树化
		if (binCount != 0) {
			if (binCount >= TREEIFY_THRESHOLD)
				treeifyBin(tab, i);
			if (oldVal != null)
				return oldVal;
				break;
			}
		}
	}
	addCount(1L, binCount);
	return null;
}
```

**put** CAS 加锁
1.8中不依赖与segment加锁， segment数量与桶数量一致；

首先判断容器是否为空，为空则进行初始化利用volatile的sizeCtl作为互斥手段，如果发现竞争性的初始化，就暂停在那里，等待条件恢复，否则利用CAS设置排他标志（U.compareAndSwapInt(this, SIZECTL, sc, -1)） ;否则重试

对key hash计算得到该key存放的**桶位置（不再是segement)**，判断该桶是否为空，为空则利用CAS设置新节点

否则使用synchronize加锁，遍历桶中数据，替换或新增加点到桶中
最后判断是否需要转为红黑树，转换之前判断是否需要扩容

**size**
利用LongAdder累加计算（性能还要高于直接使用AtomicLong）

<br/>


### Java提供了哪些IO方式？ NIO如何实现多路复用？
**典型回答**
Java IO方式有很多种，基于不同的IO抽象模型和交互方式，可以进行简单区分。

首先，传统的java.io包，它基于流模型实现，提供了我们最熟知的一些IO功能，比如File抽象、输入输出流等。交互方式是同步、阻塞的方式，也就是说，在读取输入流或者写入输出流时，在读、写动作完成之前，线程会一直阻塞在那里，它们之间的调用是可靠的线性顺序。java.io包的好处是代码比较简单、直观，缺点则是IO效率和扩展性存在局限性，容易成为应用性能的瓶颈。

很多时候，人们也把java.net下面提供的部分网络API，比如Socket、 ServerSocket、 HttpURLConnection也归类到同步阻塞IO类库，因为网络通信同样是IO行为。

第二，在Java 1.4中引入了NIO框架（java.nio包），提供了Channel、 Selector、 Bufer等新的抽象，可以构建多路复用的、同步非阻塞IO程序，同时提供了更接近操作系统底层的高性能数据操作方式。

第三，在Java 7中， NIO有了进一步的改进，也就是NIO 2，引入了异步非阻塞IO方式，也有很多人叫它AIO（Asynchronous IO）。异步IO操作基于事件和回调机制，可以简单理解为，应用操作直接返回，而不会阻塞在那里，当后台处理完成，操作系统会通知相应线程进行后续工作。

**知识扩展**
首先，需要澄清一些基本概念：

区分同步或异步（synchronous/asynchronous）。简单来说，同步是一种可靠的有序运行机制，当我们进行同步操作时，后续的任务是等待当前调用返回，才会进行下一步；
而异步则相反，其他任务不需要等待当前调用返回，通常依靠事件、回调等机制来实现任务间次序关系。

区分阻塞与非阻塞（blocking/non-blocking）。在进行阻塞操作时，当前线程会处于阻塞状态，无法从事其他任务，只有当条件就绪才能继续，比如ServerSocket新连接建立完毕，或数据读取、写入操作完成；而非阻塞则是不管IO操作是否结束，直接返回，相应操作在后台继续处理。

**1.Java NIO概览**
首先，熟悉一下NIO的主要组成部分：
Buffer，高效的数据容器，除了布尔类型，所有原始数据类型都有相应的Buffer实现。
Channel，类似在Linux之类操作系统上看到的文件描述符，是NIO中被用来支持批量式IO操作的一种抽象。

File或者Socket，通常被认为是比较高层次的抽象，而Channel则是更加操作系统底层的一种抽象，这也使得NIO得以充分利用现代操作系统底层机制，获得特定场景的性能优化，例如， DMA（Direct Memory Access）等。不同层次的抽象是相互关联的，我们可以通过Socket获取Channel，反之亦然。

**Selector，是NIO实现多路复用的基础，它提供了一种高效的机制，可以检测到注册在Selector上的多个Channel中，是否有Channel处于就绪状态，进而实现了单线程对多Channel的高效管理**。Selector同样是基于底层操作系统机制，不同模式、不同版本都存在区别。

Chartset，提供Unicode字符串定义， NIO也提供了相应的编解码器等，例如，通过下面的方式进行字符串到ByteBufer的转换：

```java
Charset.defaultCharset().encode("Hello world!"));
```
BIO NIO 代码略。

在Java 7引入的NIO 2中，又增添了一种额外的异步IO模式，利用事件和回调，处理Accept、 Read等操作。 AIO实现看起来是类似这样子：

```java
AsynchronousServerSocketChannel serverSock =AsynchronousServerSocketChannel.open().bind(sockAddr);
serverSock.accept(serverSock, new CompletionHandler<>() { //为异步操作指定CompletionHandler回调函数
	@Override
	public void completed(AsynchronousSocketChannel sockChannel,AsynchronousServerSocketChannel serverSock) {
	serverSock.accept(serverSock, this);
	// 另外一个 write（sock， CompletionHandler{}）
	sayHelloWorld(sockChannel, Charset.defaultCharset().encode("Hello World!"));
	}
	// 省略其他路径处理方法...
});
```
**小结**:
- BIO 同步阻塞；
- NIO 同步非阻塞； 
- AIO 异步非阻塞.

<br/>

### Java有几种文件拷贝方式？哪一种最高效？
**典型回答**
Java有多种比较典型的文件拷贝实现方式，比如：
利用java.io类库，直接为源文件构建一个FileInputStream读取，然后再为目标文件构建一个FileOutputStream，完成写入工作。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191008153557794.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)
或者，利用java.nio类库提供的transferTo或transferFrom方法实现。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019100815362759.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)
当然， Java标准类库本身已经提供了几种Files.copy的实现。
对于Copy的效率，这个其实与操作系统和配置等情况相关，总体上来说， NIO transferTo/From的方式可能更快，因为它更能利用现代操作系统底层机制，避免不必要拷贝和上下
文切换。

<br/>

### 谈谈接口和抽象类有什么区别？
**典型回答**
接口和抽象类是Java面向对象设计的两个基础机制。

接口是对行为的抽象，它是抽象方法的集合，利用接口可以达到API定义和实现分离的目的。接口，不能实例化；不能包含任何非常量成员，任何feld都是隐含着public static
final的意义；同时，没有非静态方法实现，也就是说要么是抽象方法，要么是静态方法。 Java标准类库中，定义了非常多的接口，比如java.util.List。

抽象类是不能实例化的类，用abstract关键字修饰class，其目的主要是代码重用。除了不能实例化，形式上和一般的Java类并没有太大区别，可以有一个或者多个抽象方法，也可
以没有抽象方法。抽象类大多用于抽取相关Java类的共用方法实现或者是共同成员变量，然后通过继承的方式达到代码复用的目的。 Java标准库中，比如collection框架，很多通用
部分就被抽取成为抽象类，例如java.util.AbstractList。

设想，为接口添加任何抽象方法，相应的所有实现了这个接口的类，也必须实现新增方法，否则会出现编译错误。对于抽象类，如果我们添加非抽象方法，其子类只会享受到能力扩展，而不用担心编译出问题.

接口的职责也不仅仅限于抽象方法的集合，其实有各种不同的实践。有一类没有任何方法的接口，通常叫作Marker Interface，顾名思义，它的目的就是为了声明某些东西，比如我
们熟知的Cloneable、 Serializable等。这种用法，也存在于业界其他的Java产品代码中。


<br/>

### 谈谈你知道的设计模式？请手动实现单例模式， Spring等框架中使用了哪些模式？
**典型回答**
大致按照模式的应用目标分类，设计模式可以分为创建型模式、结构型模式和行为型模式。
- 创建型模式，是对对象创建过程的各种问题和解决方案的总结，包括各种工厂模式（Factory、 Abstract Factory）、单例模式（Singleton）、构建器模式（Builder）、原型模
式（ProtoType）。
- 结构型模式，是针对软件设计结构的总结，关注于类、对象继承、组合方式的实践经验。常见的结构型模式，包括桥接模式（Bridge）、适配器模式（Adapter）、装饰者模式
（Decorator）、代理模式（Proxy）、组合模式（Composite）、外观模式（Facade）、享元模式（Flyweight）等。
- 行为型模式，是从类或对象之间交互、职责划分等角度总结的模式。比较常见的行为型模式有策略模式（Strategy）、解释器模式（Interpreter）、命令模式（Command）、
观察者模式（Observer）、迭代器模式（Iterator）、模板方法模式（Template Method）、访问者模式（Visitor）。


一起来简要看看主流开源框架，如Spring等如何在API设计中使用设计模式。你至少要有个大体的印象，如：
- BeanFactory和ApplicationContext应用了工厂模式。
- 在Bean的创建中， Spring也为不同scope定义的对象，提供了单例和原型等模式实现。
- AOP领域则是使用了代理模式、装饰器模式、适配器模式等。
- 各种事件监听器SpringEvent，是观察者模式的典型应用。
- 类似JdbcTemplate等则是应用了模板模式。

<br/>

### synchronized和ReentrantLock有什么区别？有人说synchronized最慢，这话靠谱吗？
**典型回答**
synchronized是Java内建的同步机制，所以也有人称其为Intrinsic Locking，它提供了互斥的语义和可见性，当一个线程已经获取当前锁时，其他试图获取的线程只能等待或者阻
塞在那里。

在Java 5以前， synchronized是仅有的同步手段，在代码中， synchronized可以用来修饰方法，也可以使用在特定的代码块儿上，本质上synchronized方法等同于把方法全部语
句用synchronized块包起来。

ReentrantLock，通常翻译为再入锁，是Java 5提供的锁实现，它的语义和synchronized基本相同。再入锁通过代码直接调用lock()方法获取，代码书写也更加灵活。与此同
时， ReentrantLock提供了很多实用的方法，能够实现很多synchronized无法做到的细节控制，比如可以控制fairness，也就是公平性，或者利用定义条件等。但是，编码中也需
要注意，必须要明确调用unlock()方法释放，不然就会一直持有该锁。

synchronized和ReentrantLock的性能不能一概而论，早期版本synchronized在很多场景下性能相差较大，在后续版本进行了较多改进，在低竞争场景中表现可能优
于ReentrantLock。


线程安全需要保证几个基本特性：
- 原子性，简单说就是相关操作不会中途被其他线程干扰，一般通过同步机制实现。
- 可见性，是一个线程修改了某个共享变量，其状态能够立即被其他线程知晓，通常被解释为将线程本地状态反映到主内存上， volatile就是负责保证可见性的。
- 有序性，是保证线程内串行语义，避免指令重排等。

ReentrantLock。你可能好奇什么是再入？它是表示当一个线程试图获取一个它已经获取的锁时，这个获取动作就自动成功，这是对锁获取粒度的一个概念，也就是锁的持
有是以线程为单位而不是基于调用次数。 Java锁实现强调再入性是为了和thread的行为进行区分。

ReentrantLock相比synchronized，因为可以像普通对象一样使用，所以可以利用其提供的各种便利方法，进行精细的同步操作，甚至是实现synchronized难以表达的用例，如：
带超时的获取锁尝试。
可以判断是否有线程，或者某个特定线程，在排队等待获取锁。
可以响应中断请求。
...
这里我特别想强调条件变量（ java.util.concurrent.Condition），如果说ReentrantLock是synchronized的替代选择， Condition则是将wait、 notify、 notifyAll等操作转化为相
应的对象，将复杂而晦涩的同步操作转变为直观可控的对象行为。
条件变量最为典型的应用场景就是标准类库中的ArrayBlockingQueue等。

<br/>

### synchronized底层如何实现？什么是锁的升级、降级？
synchronized代码块是由一对儿monitorenter/monitorexit指令实现的， Monitor对象是同步的基本实现单元。

在Java 6之前， Monitor的实现完全是依靠操作系统内部的互斥锁，因为需要进行用户态到内核态的切换，所以同步操作是一个无差别的重量级操作。

现代的（ Oracle） JDK中， JVM对此进行了大刀阔斧地改进，提供了三种不同的Monitor实现，也就是常说的三种不同的锁：偏斜锁（ Biased Locking）、轻量级锁和重量级锁，大
大改进了其性能。

所谓锁的升级、降级，就是JVM优化synchronized运行的机制，当JVM检测到不同的竞争状况时，会自动切换到适合的锁实现，这种切换就是锁的升级、降级。
当没有竞争出现时，默认会使用偏斜锁。 JVM会利用CAS操作（ compare and swap），在对象头上的Mark Word部分设置线程ID，以表示这个对象偏向于当前线程，所以并不涉
及真正的互斥锁。这样做的假设是基于在很多应用场景中，大部分对象生命周期中最多会被一个线程锁定，使用偏斜锁可以降低无竞争开销。

如果有另外的线程试图锁定某个已经被偏斜过的对象， JVM就需要撤销（ revoke）偏斜锁，并切换到轻量级锁实现。轻量级锁依赖CAS操作Mark Word来试图获取锁，如果重试成
功，就使用普通的轻量级锁；否则，进一步升级为重量级锁。

我注意到有的观点认为Java不会进行锁降级。实际上据我所知，锁降级确实是会发生的，当JVM进入安全点（ SafePoint）的时候，会检查是否有闲置的Monitor，然后试图进行降
级。 


Java核心类库中还有其他一些特别的锁类型，具体请参考下面的图。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191010100647506.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

这些锁竟然不都是实现了Lock接口， ReadWriteLock是一个单独的接口，它通常是代表了一对儿锁，分别对应只读和写操作，标准类库中提供了再入版本的读写
锁实现（ ReentrantReadWriteLock），对应的语义和ReentrantLock比较相似。

StampedLock竟然也是个单独的类型，从类图结构可以看出它是不支持再入性的语义的，也就是它不是以持有锁的线程为单位。

为什么我们需要读写锁（ ReadWriteLock）等其他锁呢？

这是因为，虽然ReentrantLock和synchronized简单实用，但是行为上有一定局限性，通俗点说就是“太霸道”，要么不占，要么独占。实际应用场景中，有的时候不需要大量竞争
的写操作，而是以并发读取为主，如何进一步优化并发操作的粒度呢？

Java并发包提供的读写锁等扩展了锁的能力，它所基于的原理是多个读操作是不需要互斥的，因为读操作并不会更改数据，所以不存在互相干扰。而写操作则会导致并发一致性的问
题，所以写线程之间、读写线程之间，需要精心设计的互斥逻辑。

<br/>

### 一个线程两次调用start()方法会出现什么情况？谈谈线程的生命周期和状态转移。
**典型回答**
Java的线程是不允许启动两次的，第二次调用必然会抛出IllegalThreadStateException，这是一种运行时异常，多次调用start被认为是编程错误。

关于线程生命周期的不同状态，在Java 5以后，线程状态被明确定义在其公共内部枚举类型java.lang.Thread.State中，分别是：
- 新建（ NEW），表示线程被创建出来还没真正启动的状态，可以认为它是个Java内部状态。
- 就绪（ RUNNABLE），表示该线程已经在JVM中执行，当然由于执行需要计算资源，它可能是正在运行，也可能还在等待系统分配给它CPU片段，在就绪队列里面排队。
- 在其他一些分析中，会额外区分一种状态RUNNING，但是从Java API的角度，并不能表示出来。
- 阻塞（ BLOCKED），这个状态和我们前面两讲介绍的同步非常相关，阻塞表示线程在等待Monitor lock。比如，线程试图通过synchronized去获取某个锁，但是其他线程已经
独占了，那么当前线程就会处于阻塞状态。
- 等待（ WAITING），表示正在等待其他线程采取某些操作。一个常见的场景是类似生产者消费者模式，发现任务条件尚未满足，就让当前消费者线程等待（ wait），另外的生产
者线程去准备任务数据，然后通过类似notify等动作，通知消费线程可以继续工作了。 Thread.join()也会令线程进入等待状态。
- 计时等待（ TIMED_WAIT），其进入条件和等待状态类似，但是调用的是存在超时条件的方法，比如wait或join等方法的指定超时版本，如下面示例：
>public fnal native void wait(long timeout) throws InterruptedException;

- 终止（ TERMINATED），不管是意外退出还是正常执行结束，线程已经完成使命，终止运行，也有人把这个状态叫作死亡。
在第二次调用start()方法的时候，线程可能处于终止或者其他（非NEW）状态，但是不论如何，都是不可以再次启动的。

**知识扩展**

1.首先，我们来整体看一下线程是什么？

从操作系统的角度，可以简单认为，线程是系统调度的最小单元，一个进程可以包含多个线程，作为任务的真正运作者，有自己的栈（ Stack）、寄存器（ Register）、本地存储
（ Thread Local）等，但是会和进程内其他线程共享文件描述符、虚拟地址空间等。

2.从线程生命周期的状态开始展开，那么在Java编程中，有哪些因素可能影响线程的状态呢？主要有：

- 线程自身的方法，除了start，还有多个join方法，等待线程结束； yield是告诉调度器，主动让出CPU；另外，就是一些已经被标记为过时的resume、 stop、 suspend之类，据
我所知，在JDK最新版本中， destory/stop方法将被直接移除。
- 基类Object提供了一些基础的wait/notify/notifyAll方法。如果我们持有某个对象的Monitor锁，调用wait会让当前线程处于等待状态，直到其他线程notify或者notifyAll。所
以，本质上是提供了Monitor的获取和释放的能力，是基本的线程间通信方式。
- 并发类库中的工具，比如CountDownLatch.await()会让当前线程进入等待状态，直到latch被基数为0，这可以看作是线程间通信的Signal。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191011145236970.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

<br/>

### 什么情况下Java程序会产生死锁？如何定位、修复？

**典型回答**
死锁是一种特定的程序状态，在实体之间，由于循环依赖导致彼此一直处于等待之中，没有任何个体可以继续前进。死锁不仅仅是在线程之间会发生，存在资源独占的进程之间同样
也可能出现死锁。通常来说，我们大多是聚焦在多线程场景中的死锁，指两个或多个线程之间，由于互相持有对方需要的锁，而永久处于阻塞的状态。

定位死锁最常见的方式就是利用jstack等工具获取线程栈，然后定位互相之间的依赖关系，进而找到死锁。如果是比较明显的死锁，往往jstack等就能直接定位，类似JConsole甚至
可以在图形界面进行有限的死锁检测。


如何在编程中尽量预防死锁呢？

首先，我们来总结一下前面例子中死锁的产生包含哪些基本元素。基本上死锁的发生是因为：
- 互斥条件，类似Java中Monitor都是独占的，要么是我用，要么是你用。
- 互斥条件是长期持有的，在使用结束之前，自己不会释放，也不能被其他线程抢占。
- 循环依赖关系，两个或者多个个体之间出现了锁的链条环。

第一种方法
如果可能的话，尽量避免使用多个锁，并且只有需要时才持有锁。否则，即使是非常精通并发编程的工程师，也难免会掉进坑里，嵌套的synchronized或者lock非常容易出问题。

第二种方法
如果必须使用多个锁，尽量设计好锁的获取顺序，这个说起来简单，做起来可不容易，你可以参看著名的银行家算法.

第三种方法
使用带超时的方法，为程序带来更多可控性。
类似Object.wait(…)或者CountDownLatch.await(…)，都支持所谓的timed_wait，我们完全可以就不假定该锁一定会获得，指定超时时间，并为无法得到锁时准备退出逻辑。

<br/>

### Java并发包提供了哪些并发工具类？
**典型回答**
我们通常所说的并发包也就是java.util.concurrent及其子包，集中了Java并发的各种基础工具类，具体主要包括几个方面：
- 提供了比synchronized更加高级的各种同步结构，包括CountDownLatch、 CyclicBarrier、 Semaphore等，可以实现更加丰富的多线程操作，比如利用Semaphore作为资源
控制器，限制同时进行工作的线程数量。
- 各种线程安全的容器，比如最常见的ConcurrentHashMap、有序的ConcunrrentSkipListMap，或者通过类似快照机制，实现线程安全的动态数
组CopyOnWriteArrayList等。
- 各种并发队列实现，如各种BlockedQueue实现，比较典型的ArrayBlockingQueue、 SynchorousQueue或针对特定场景的PriorityBlockingQueue等。
- 强大的Executor框架，可以创建各种不同类型的线程池，调度任务运行等，绝大部分情况下，不再需要自己从头实现线程池和任务调度器.

**知识扩展 **
- CountDownLatch，允许一个或多个线程等待某些操作完成。
- CyclicBarrier，一种辅助性的同步结构，允许多个线程等待到达某个屏障。
- Semaphore， Java版本的信号量实现,Semaphore就是个计数器， 其基本逻辑基于acquire/release，并没有太复杂的同步逻辑。

**Semaphore**

1.工作原理

以一个停车场是运作为例。为了简单起见，假设停车场只有三个车位，一开始三个车位都是空的。这时如果同时来了五辆车，看门人允许其中三辆不受阻碍的进入，然后放下车拦，剩下的车则必须在入口等待，此后来的车也都不得不在入口处等待。这时，有一辆车离开停车场，看门人得知后，打开车拦，放入一辆，如果又离开两辆，则又可以放入两辆，如此往复。这个停车系统中，每辆车就好比一个线程，看门人就好比一个信号量，看门人限制了可以活动的线程。假如里面依然是三个车位，但是看门人改变了规则，要求每次只能停两辆车，那么一开始进入两辆车，后面得等到有车离开才能有车进入，但是得保证最多停两辆车。对于Semaphore类而言，就如同一个看门人，限制了可活动的线程数。

2.主要方法

- Semaphore(int permits):构造方法，创建具有给定许可数的计数信号量并设置为非公平信号量。
- Semaphore(int permits,boolean fair):构造方法，当fair等于true时，创建具有给定许可数的计数信号量并设置为公平信号量。
- void acquire():从此信号量获取一个许可前线程将一直阻塞。相当于一辆车占了一个车位。
- void acquire(int n):从此信号量获取给定数目许可，在提供这些许可前一直将线程阻塞。比如n=2，就相当于一辆车占了两个车位。
- void release():释放一个许可，将其返回给信号量。就如同车开走返回一个车位。
- void release(int n):释放n个许可。
- int availablePermits()：当前可用的许可数。

3. 更多

https://www.cnblogs.com/klbc/p/9500947.html



下面，来看看CountDownLatch和CyclicBarrier，它们的行为有一定的相似度，经常会被考察二者有什么区别，我来简单总结一下。

- CountDownLatch是不可以重置的，所以无法重用；而CyclicBarrier则没有这种限制，可以重用。
- CountDownLatch的基本操作组合是countDown/await。调用await的线程阻塞等待countDown足够的次数，不管你是在一个线程还是多个线程里countDown，只要次数足够
即可。所以就像Brain Goetz说过的， CountDownLatch操作的是事件。
- CyclicBarrier的基本操作组合，则就是await，当所有的伙伴（ parties）都调用了await，才会继续进行任务，并自动进行重置。 注意，正常情况下， CyclicBarrier的重置都是自
动发生的，如果我们调用reset方法，但还有线程在等待，就会导致等待线程被打扰，抛出BrokenBarrierException异常。 CyclicBarrier侧重点是线程，而不是调用事件，它的
典型应用场景是用来等待并发线程结束。

**CountDownLatch**
模拟五个线程同时启动:
```java
	public static void main(String[] args) {
		
		//所有线程阻塞，然后统一开始
		CountDownLatch begin = new CountDownLatch(1);
		
		//主线程阻塞，直到所有分线程执行完毕
		CountDownLatch end = new CountDownLatch(5);
		
		for(int i = 0; i < 5; i++){
			Thread thread = new Thread(new Runnable() {
				
				@Override
				public void run() {
					try {
						begin.await();
						System.out.println(Thread.currentThread().getName() + " 起跑");
						Thread.sleep(1000);
						System.out.println(Thread.currentThread().getName() + " 到达终点");
						end.countDown();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					
				}
			});
			
			thread.start();
		}
		
		try {
			System.out.println("1秒后统一开始");
			Thread.sleep(1000);
			begin.countDown();
 
			end.await();
			System.out.println("停止比赛");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
	}
```
结果:
```java
1秒后统一开始
Thread-1 起跑
Thread-4 起跑
Thread-3 起跑
Thread-0 起跑
Thread-2 起跑
Thread-3 到达终点
Thread-0 到达终点
Thread-4 到达终点
Thread-1 到达终点
Thread-2 到达终点
停止比赛

```



并发包里提供的线程安全Map、 List和Set:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191012143046896.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

如果我们的应用侧重于Map放入或者获取的速度，而不在乎顺序，大多推荐使用ConcurrentHashMap，反之则使
用ConcurrentSkipListMap；如果我们需要对大量数据进行非常频繁地修改， ConcurrentSkipListMap也可能表现出优势。

SkipList结构:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191012143349247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

关于两个CopyOnWrite容器，其实CopyOnWriteArraySet是通过包装了CopyOnWriteArrayList来实现的，所以在学习时，我们可以专注于理解一种。

首先， CopyOnWrite到底是什么意思呢？它的原理是，任何修改操作，如add、 set、 remove，都会拷贝原数组，修改后替换原来的数组，通过这种防御性的方式，实现另类的线程安全。

```java
public boolean add(E e) {
	synchronized (lock) {
		Object[] elements = getArray();
		int len = elements.length;
		// 拷贝
		Object[] newElements = Arrays.copyOf(elements, len + 1);
		newElements[len] = e;
		// 替换
		setArray(newElements);
		return true;
	}
}
final void setArray(Object[] a) {
	array = a;
}
```


<br/>

### 并发包中的ConcurrentLinkedQueue和LinkedBlockingQueue有什么区别？
**典型回答**

有时候我们把并发包下面的所有容器都习惯叫作并发容器，但是严格来讲，类似ConcurrentLinkedQueue这种“Concurrent*”容器，才是真正代表并发。

关于问题中它们的区别：
- Concurrent类型基于lock-free，在常见的多线程访问场景，一般可以提供较高吞吐量。
- 而LinkedBlockingQueue内部则是基于锁，并提供了BlockingQueue的等待性方法。

不知道你有没有注意到， java.util.concurrent包提供的容器（ Queue、 List、 Set）、 Map，从命名上可以大概区分为Concurrent、 CopyOnWrite和Blocking*等三类，同样是线
程安全容器，可以简单认为：

- Concurrent类型没有类似CopyOnWrite之类容器相对较重的修改开销。
- 但是，凡事都是有代价的， Concurrent往往提供了较低的遍历一致性。你可以这样理解所谓的弱一致性，例如，当利用迭代器遍历时，如果容器发生修改，迭代器仍然可以继续
进行遍历。
- 与弱一致性对应的，就是我介绍过的同步容器常见的行为“fast-fail”，也就是检测到容器在遍历过程中发生了修改，则抛出ConcurrentModifcationException，不再继续遍历。
- 弱一致性的另外一个体现是， size等操作准确性是有限的，未必是100%准确。
- 与此同时，读取的性能具有一定的不确定性。


**知识扩展**

**线程安全队列一览:**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191018163501881.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

ArrayBlockingQueue是最典型的的有界队列，其内部以final的数组保存数据，数组的大小就决定了队列的边界，所以我们在创建ArrayBlockingQueue时，都要指定容量，如
```
public ArrayBlockingQueue(int capacity, boolean fair)
```

LinkedBlockingQueue，容易被误解为无边界，但其实其行为和内部代码都是基于有界的逻辑实现的，只不过如果我们没有在创建队列时就指定容量，那么其容量限制就自动被
设置为Integer.MAX_VALUE ，成为了无界队列。

SynchronousQueue，这是一个非常奇葩的队列实现，每个删除操作都要等待插入操作，反之每个插入操作也都要等待删除动作。那么这个队列的容量是多少呢？是1吗？其实不
是的，其内部容量是0。

PriorityBlockingQueue是无边界的优先队列，虽然严格意义上来讲，其大小总归是要受系统资源影响。

DelayedQueue和LinkedTransferQueue同样是无边界的队列。对于无边界的队列，有一个自然的结果，就是put操作永远也不会发生其他BlockingQueue的那种等待情况。

使用Blocking实现的生产者消费者代码:
```
package com.ryze.chapter3;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class ConsumerProducer {
    public static final String EXIT_MSG = "Good bye!";

    public static void main(String[] args) {
        // 使用较小的队列，以更好地在输出中展示其影响
        BlockingQueue<String> queue = new ArrayBlockingQueue<>(3);
        Producer producer = new Producer(queue);
        Consumer consumer = new Consumer(queue);
        new Thread(producer).start();
        new Thread(consumer).start();
    }

    static class Producer implements Runnable {
        private BlockingQueue<String> queue;

        public Producer(BlockingQueue<String> q) {
            this.queue = q;
        }

        @Override
        public void run() {
            for (int i = 0; i < 20; i++) {
                try {
                    Thread.sleep(5L);
                    String msg = "Message" + i;
                    System.out.println("Produced new item: " + msg);
                    queue.put(msg);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            try {
                System.out.println("Time to say good bye!");
                queue.put(EXIT_MSG);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    static class Consumer implements Runnable {
        private BlockingQueue<String> queue;

        public Consumer(BlockingQueue<String> q) {
            this.queue = q;
        }

        @Override
        public void run() {
            try {
                String msg;
                while (!EXIT_MSG.equalsIgnoreCase((msg = queue.take()))) {
                    System.out.println("Consumed item: " + msg);
                    Thread.sleep(10L);
                }
                System.out.println("Got exit message, bye!");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```


####前面介绍了各种队列实现，在日常的应用开发中，如何进行选择呢？

以LinkedBlockingQueue、 ArrayBlockingQueue和SynchronousQueue为例，我们一起来分析一下，根据需求可以从很多方面考量：

考虑应用场景中对队列边界的要求。 ArrayBlockingQueue是有明确的容量限制的，而LinkedBlockingQueue则取决于我们是否在创建时指定， SynchronousQueue则干脆不
能缓存任何元素。

从空间利用角度，数组结构的ArrayBlockingQueue要比LinkedBlockingQueue紧凑，因为其不需要创建所谓节点，但是其初始分配阶段就需要一段连续的空间，所以初始内存
需求更大。

通用场景中， LinkedBlockingQueue的吞吐量一般优于ArrayBlockingQueue，因为它实现了更加细粒度的锁操作。

ArrayBlockingQueue实现比较简单，性能更好预测，属于表现稳定的“选手”。

如果我们需要实现的是两个线程之间接力性（ handof）的场景，按照专栏上一讲的例子，你可能会选择CountDownLatch，但是SynchronousQueue也是完美符合这种场景
的，而且线程间协调和数据传输统一起来，代码更加规范。

可能令人意外的是，很多时候SynchronousQueue的性能表现，往往大大超过其他实现，尤其是在队列元素较小的场景。


### Java并发类库提供的线程池有哪几种？ 分别有什么特点?

**典型回答**
通常开发者都是利用Executors提供的通用线程池创建方法，去创建不同配置的线程池，主要区别在于不同的ExecutorService类型或者不同的初始参数。
Executors目前提供了5种不同的线程池创建配置：

newCachedThreadPool()，它是一种用来处理大量短时间工作任务的线程池，具有几个鲜明特点：它会试图缓存线程并重用，当无缓存线程可用时，就会创建新的工作线程；如
果线程闲置的时间超过60秒，则被终止并移出缓存；长时间闲置时，这种线程池，不会消耗什么资源。其内部使用SynchronousQueue作为工作队列。

newFixedThreadPool(int nThreads)，重用指定数目（ nThreads）的线程，其背后使用的是无界的工作队列，任何时候最多有nThreads个工作线程是活动的。这意味着，如
果任务数量超过了活动队列数目，将在工作队列中等待空闲线程出现；如果有工作线程退出，将会有新的工作线程被创建，以补足指定的数目nThreads。

newSingleThreadExecutor()，它的特点在于工作线程数目被限制为1，操作一个无界的工作队列，所以它保证了所有任务的都是被顺序执行，最多会有一个任务处于活动状
态，并且不允许使用者改动线程池实例，因此可以避免其改变线程数目。

newSingleThreadScheduledExecutor()和newScheduledThreadPool(int corePoolSize)，创建的是个ScheduledExecutorService，可以进行定时或周期性的工作调度，
区别在于单一工作线程还是多个工作线程。

ewWorkStealingPool(int parallelism)，这是一个经常被人忽略的线程池， Java 8才加入这个创建方法，其内部会构建ForkJoinPool，利用Work-Stealing算法，并行地处
理任务，不保证处理顺序。


Executor框架可不仅仅是线程池，我觉得至少下面几点值得深入学习：
- 掌握Executor框架的主要内容，至少要了解组成与职责，掌握基本开发用例中的使用。
- 对线程池和相关并发工具类型的理解，甚至是源码层面的掌握。
- 实践中有哪些常见问题，基本的诊断思路是怎样的。
- 如何根据自身应用特点合理使用线程池。

**知识扩展**
首先，我们来看看Executor框架的基本组成，请参考下面的类图。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191119162743953.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

Executor是一个基础的接口，其初衷是将任务提交和任务执行细节解耦，这一点可以体会其定义的唯一方法。
```
void execute(Runnable command);
```
ExecutorService则更加完善，不仅提供service的管理功能，比如shutdown等方法，也提供了更加全面的提交任务机制，如返回Future而不是void的submit方法。
```
<T> Future<T> submit(Callable<T> task);
```

从源码角度，分析线程池的设计与实现，我将主要围绕最基础的ThreadPoolExecutor源码:

![](https://img-blog.csdnimg.cn/2019111916342758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)


简单理解一下：
- 工作队列负责存储用户提交的各个任务，这个工作队列，可以是容量为0的SynchronousQueue（使用newCachedThreadPool），也可以是像固定大小线程池
（ newFixedThreadPool）那样使用LinkedBlockingQueue。
	```
	private final BlockingQueue<Runnable> workQueue;
	```

- 内部的“线程池”，这是指保持工作线程的集合，线程池需要在运行过程中管理线程创建、销毁。例如，对于带缓存的线程池，当任务压力较大时，线程池会创建新的工作线程；当
业务压力退去，线程池会在闲置一段时间（默认60秒）后结束线程。
	```
	private final HashSet<Worker> workers = new HashSet<>();
	```
	
线程池的工作线程被抽象为静态内部类Worker，基于AQS实现。
- ThreadFactory提供上面所需要的创建线程逻辑。
- 如果任务提交时被拒绝，比如线程池已经处于SHUTDOWN状态，需要为其提供处理逻辑， Java标准库提供了类似ThreadPoolExecutor.AbortPolicy 等默认实现，也可以按照实
际需求自定义。

从上面的分析，就可以看出线程池的几个基本组成部分，一起都体现在线程池的构造函数中，从字面我们就可以大概猜测到其用意：

- corePoolSize，所谓的核心线程数，可以大致理解为长期驻留的线程数目（除非设置了allowCoreThreadTimeOut）。对于不同的线程池，这个值可能会有很大区别，比
如newFixedThreadPool会将其设置为nThreads，而对于newCachedThreadPool则是为0。

- maximumPoolSize，顾名思义，就是线程不够时能够创建的最大线程数。同样进行对比，对于newFixedThreadPool，当然就是nThreads，因为其要求是固定大小，
而newCachedThreadPool则是Integer.MAX_VALUE 。

- keepAliveTime和TimeUnit，这两个参数指定了额外的线程能够闲置多久，显然有些线程池不需要它。

- workQueue，工作队列，**必须是BlockingQueue。**

通过配置不同的参数，我们就可以创建出行为大相径庭的线程池，这就是线程池高度灵活性的基础

```
public ThreadPoolExecutor(int corePoolSize,
						int maximumPoolSize,
						long keepAliveTime,
						TimeUnit unit,
						BlockingQueue<Runnable> workQueue,
						ThreadFactory threadFactory,
						RejectedExecutionHandler handler)
```


###  AtomicInteger底层实现原理是什么？如何在自己的产品代码中应用CAS操作？

**典型回答**
AtomicIntger是对int类型的一个封装，提供原子性的访问和更新操作，其原子性操作的实现是基于CAS（ compare-and-swap）.

目前Java提供了两种公共API，可以实现这种CAS操作，比如使用java.util.concurrent.atomic.AtomicLongFieldUpdater，它是基于反射机制创建，我们需要保证类型和字段名称正确。

AQS内部数据和方法，可以简单拆分为：
- 一个volatile的整数成员表征状态，同时提供了setState和getState方法
```
private volatile int sate;
```
- 一个先入先出（ FIFO）的等待线程队列，以实现多线程间竞争和等待，这是AQS机制的核心之一。
- 各种基于CAS的基础操作方法，以及各种期望具体同步结构去实现的acquire/release方法。

利用AQS实现一个同步结构，至少要实现两个基本类型的方法，分别是acquire操作，获取资源的独占权；还有就是release操作，释放对某个资源的独占

排除掉一些细节，整体地分析acquire方法逻辑，其直接实现是在AQS内部，调用了tryAcquire和acquireQueued，这是两个需要搞清楚的基本部分。
```
public final void acquire(int arg) {
if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
	selfInterrupt();
}
```

以非公平的tryAcquire为例，其内部实现了如何配合状态与CAS获取锁，注意，对比公平版本的tryAcquire，它在锁无人占有时，并不检查是否有其他等待者，这里体现了非公平的
语义。
```
final boolean nonfairTryAcquire(int acquires) {
	final Thread current = Thread.currentThread();
	int c = getState();// 获取当前AQS内部状态量
	if (c == 0) { // 0表示无人占有，则直接用CAS修改状态位，
		if (compareAndSetState(0, acquires)) {// 不检查排队情况，直接争抢
			setExclusiveOwnerThread(current); //并设置当前线程独占锁
			return true;
		}
	} else if (current == getExclusiveOwnerThread()) { //即使状态不是0，也可能当前线程是锁持有者，因为这是再入锁
		int nextc = c + acquires;
		if (nextc < 0) // overflow
			throw new Error("Maximum lock count exceeded");
		setState(nextc);
		return true;
	}
	return false;
}
```

再来分析acquireQueued，如果前面的tryAcquire失败，代表着锁争抢失败，进入排队竞争阶段。这里就是我们所说的，利用FIFO队列，实现线程间对锁的竞争的部分，
算是是AQS的核心逻辑。

当前线程会被包装成为一个排他模式的节点（ EXCLUSIVE），通过addWaiter方法添加到队列中。 acquireQueued的逻辑，简要来说，就是如果当前节点的前面是头节点，则试图
获取锁，一切顺利则成为新的头节点；否则，有必要则等待，具体处理逻辑请参考我添加的注释。

```
final boolean acquireQueued(final Node node, int arg) {
	boolean interrupted = false;
	try {
		for (;;) {// 循环
			final Node p = node.predecessor();// 获取前一个节点
			if (p == head && tryAcquire(arg)) { // 如果前一个节点是头结点，表示当前节点合适去tryAcquire
				setHead(node); // acquire成功，则设置新的头节点
				p.next = null; // 将前面节点对当前节点的引用清空
				return interrupted;
			}
			if (shouldParkAfterFailedAcquire(p, node)) // 检查是否失败后需要park
			interrupted |= parkAndCheckInterrupt();
		}
	} catch (Throwable t) {
		cancelAcquire(node);// 出现异常，取消
		if (interrupted)
		selfInterrupt();
		throw t;
	}
}
```

到这里线程试图获取锁的过程基本展现出来了， tryAcquire是按照特定场景需要开发者去实现的部分，而线程间竞争则是AQS通过Waiter队列与acquireQueued提供的，
在release方法中，同样会对队列进行对应操作.




### 请介绍类加载过程，什么是双亲委派模型？

**典型回答**
一般来说，我们把Java的类加载过程分为三个主要步骤：加载、链接、初始化，具体行为在Java虚拟机规范里有非常详细的定义。

首先是加载阶段（ Loading），它是Java将字节码数据从不同的数据源读取到JVM中，并映射为JVM认可的数据结构（ Class对象），这里的数据源可能是各种各样的形态，如jar文
件、 class文件，甚至是网络数据源等；如果输入数据不是ClassFile的结构，则会抛出ClassFormatError。
加载阶段是用户参与的阶段，我们可以自定义类加载器，去实现自己的类加载过程。

第二阶段是链接（ Linking），这是核心的步骤，简单说是把原始的类定义信息平滑地转化入JVM运行的过程中。这里可进一步细分为三个步骤：

验证（ Verifcation），这是虚拟机安全的重要保障， JVM需要核验字节信息是符合Java虚拟机规范的，否则就被认为是VerifyError，这样就防止了恶意信息或者不合规的信息危
害JVM的运行，验证阶段有可能触发更多class的加载。

准备（ Preparation），创建类或接口中的静态变量，并初始化静态变量的初始值。但这里的“初始化”和下面的显式初始化阶段是有区别的，侧重点在于分配所需要的内存空间，
不会去执行更进一步的JVM指令。

解析（ Resolution），在这一步会将常量池中的符号引用（ symbolic reference）替换为直接引用。在Java虚拟机规范中，详细介绍了类、接口、方法和字段等各个方面的解
析。

最后是初始化阶段（ initialization），这一步真正去执行类初始化的代码逻辑，包括静态字段赋值的动作，以及执行类定义中的静态初始化块内的逻辑，编译器在编译阶段就会把这
部分逻辑整理好，父类型的初始化逻辑优先于当前类型的逻辑。

再来谈谈双亲委派模型，简单说就是当类加载器（ Class-Loader）试图加载某个类型的时候，除非父加载器找不到相应类型，否则尽量将这个任务代理给当前加载器的父加载器去
做。使用委派模型的目的是避免重复加载Java类型。


通常类加载机制有三个基本特征：

双亲委派模型。但不是所有类加载都遵守这个模型，有的时候，启动类加载器所加载的类型，是可能要加载用户代码的，比如JDK内部的ServiceProvider/ServiceLoader机制，
用户可以在标准API框架上，提供自己的实现， JDK也需要提供些默认的参考实现。 例如， Java 中JNDI、 JDBC、文件系统、 Cipher等很多方面，都是利用的这种机制，这种情
况就不会用双亲委派模型去加载，而是利用所谓的上下文加载器。

可见性，子类加载器可以访问父加载器加载的类型，但是反过来是不允许的，不然，因为缺少必要的隔离，我们就没有办法利用类加载器去实现容器的逻辑。

单一性，由于父加载器的类型对于子加载器是可见的，所以父加载器中加载过的类型，就不会在子加载器中重复加载。但是注意，类加载器“邻居”间，同一类型仍然可以被加载多
次，因为互相并不可见。



### 谈谈JVM内存区域的划分，哪些区域可能发生OutOfMemoryError？

**典型回答**

通常可以把JVM内存区域分为下面几个方面，其中，有的区域是以线程为单位，而有的区域则是整个JVM进程唯一的。

首先， 程序计数器（ PC， Program Counter Register）。在JVM规范中，每个线程都有它自己的程序计数器，并且任何时间一个线程都只有一个方法在执行，也就是所谓的当前方
法。程序计数器会存储当前线程正在执行的Java方法的JVM指令地址；或者，如果是在执行本地方法，则是未指定值（ undefned）。

第二， Java虚拟机栈（ Java Virtual Machine Stack），早期也叫Java栈。每个线程在创建时都会创建一个虚拟机栈，其内部保存一个个的栈帧（ Stack Frame），对应着一次次
的Java方法调用。

前面谈程序计数器时，提到了当前方法；同理，在一个时间点，对应的只会有一个活动的栈帧，通常叫作当前帧，方法所在的类叫作当前类。如果在该方法中调用了其他方法，对应
的新的栈帧会被创建出来，成为新的当前帧，一直到它返回结果或者执行结束。 JVM直接对Java栈的操作只有两个，就是对栈帧的压栈和出栈。
栈帧中存储着局部变量表、操作数（ operand）栈、动态链接、方法正常退出或者异常退出的定义等。

第三， 堆（ Heap），它是Java内存管理的核心区域，用来放置Java对象实例，几乎所有创建的Java对象实例都是被直接分配在堆上。堆被所有的线程共享，在虚拟机启动时，我们
指定的“Xmx”之类参数就是用来指定最大堆空间等指标。
理所当然，堆也是垃圾收集器重点照顾的区域，所以堆内空间还会被不同的垃圾收集器进行进一步的细分，最有名的就是新生代、老年代的划分。

第四， 方法区（ Method Area）。这也是所有线程共享的一块内存区域，用于存储所谓的元（ Meta）数据，例如类结构信息，以及对应的运行时常量池、字段、方法代码等。
由于早期的Hotspot JVM实现，很多人习惯于将方法区称为永久代（ Permanent Generation）。 Oracle JDK 8中将永久代移除，同时增加了元数据区（ Metaspace）。

第五， 运行时常量池（ Run-Time Constant Pool），这是方法区的一部分。如果仔细分析过反编译的类文件结构，你能看到版本号、字段、方法、超类、接口等各种信息，还有一
项信息就是常量池。 Java的常量池可以存放各种常量信息，不管是编译期生成的各种字面量，还是需要在运行时决定的符号引用，所以它比一般语言的符号表存储的信息更加宽泛。

第六， 本地方法栈（ Native Method Stack）。它和Java虚拟机栈是非常相似的，支持对本地方法的调用，也是每个线程都会创建一个。在Oracle Hotspot JVM中，本地方法栈
和Java虚拟机栈是在同一块儿区域，这完全取决于技术实现的决定，并未在规范中强制。


![在这里插入图片描述](https://img-blog.csdnimg.cn/20191120165915355.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)


**所有的对象实例都是创建在堆上。**


除了程序计数器，其他区域都有可能会因为可能的空间不足发生OutOfMemoryError，简单总结如下：

堆内存不足是最常见的OOM原因之一，抛出的错误信息是“java.lang.OutOfMemoryError:Java heap space”，原因可能千奇百怪，例如，可能存在内存泄漏问题；也很有可
能就是堆的大小不合理，比如我们要处理比较可观的数据量，但是没有显式指定JVM堆大小或者指定数值偏小；或者出现JVM处理引用不及时，导致堆积起来，内存无法释放等。

而对于Java虚拟机栈和本地方法栈，这里要稍微复杂一点。如果我们写一段程序不断的进行递归调用，而且没有退出条件，就会导致不断地进行压栈。类似这种情况， JVM实际会
抛出StackOverFlowError；当然，如果JVM试图去扩展栈空间的的时候失败，则会抛出OutOfMemoryError。

对于老版本的Oracle JDK，因为永久代的大小是有限的，并且JVM对永久代垃圾回收（如，常量池回收、卸载不再需要的类型）非常不积极，所以当我们不断添加新类型的时
候，永久代出现OutOfMemoryError也非常多见，尤其是在运行时存在大量动态类型生成的场合；类似Intern字符串缓存占用太多空间，也会导致OOM问题。对应的异常信息，
会标记出来和永久代相关： “java.lang.OutOfMemoryError: PermGen space”。

随着元数据区的引入，方法区内存已经不再那么窘迫，所以相应的OOM有所改观，出现OOM，异常信息则变成了： “java.lang.OutOfMemoryError: Metaspace”。

直接内存不足，也会导致OOM.


**思考**
我在试图分配一个100M bytes大数组的时候发生了OOME，但是GC日志显示，明明堆上还有远不止100M的空间，你觉得可能问题的原因是什么？想要弄清楚这个问题，还需要什么信息呢？

思路1:
如果仅从jvm的角度来看，要看下新生代和老年代的垃圾回收机制是什么。如果新生代是serial，会默认使用copying算法，利用两块eden和survivor来进行处理。但是默认当遇到超大对象
时，会直接将超大对象放置到老年代中，而不用走正常对象的存活次数记录。因为要放置的是一个byte数组，那么必然需要申请连续的空间，当空间不足时，会进行gc操作。这里又需要看老年
代的gc机制是哪一种。如果是serial old，那么会采用mark compat，会进行整理，从而整理出连续空间，如果还不够，说明是老年代的空间不够，所谓的堆内存大于100m是新+老共同的结
果。如果采用的是cms(concurrent mark sweep)，那么只会标记清理，并不会压缩，所以内存会碎片化，同时可能出现浮游垃圾。如果是cms的话，即使老年代的空间大于100m，也会出现
没有连续的空间供该对象使用。

思路2:
从不同的垃圾收集器角度来看：
首先，数组的分配是需要连续的内存空间的（据说，有个别非主流JVM支持大数组用不连续的内存空间分配��）。所以：
1）对于使用年轻代和老年代来管理内存的垃圾收集器，堆大于 100M，表示的是新生代和老年代加起来总和大于100M，而新生代和老年代各自并没有大于 100M 的连续内存空间。
进一步，又由于大数组一般直接进入老年代（会跳过对对象的年龄的判断），所以，是否可以认为老年代中没有连续大于 100M 的空间呢。
2）对于 G1 这种按 region 来管理内存的垃圾收集器，可能的情况是没有多个连续的 region，它们的内存总和大于 100M。
当然，不管是哪种垃圾收集器以及收集算法，当内存空间不足时，都会触发 GC，只不过，可能 GC 之后，还是没有连续大于 100M 的内存空间，于是 OOM了。



### 如何监控和诊断JVM堆内和堆外内存使用？

**典型回答**
了解JVM内存的方法有很多，具体能力范围也有区别，简单总结如下：

可以使用综合性的图形化工具，如JConsole、 VisualVM（注意，从Oracle JDK 9开始， VisualVM已经不再包含在JDK安装包中）等。这些工具具体使用起来相对比较直观，直
接连接到Java进程，然后就可以在图形化界面里掌握内存使用情况。

以JConsole为例，其内存页面可以显示常见的堆内存和各种堆外部分使用状态。

也可以使用命令行工具进行运行时查询，如jstat和jmap等工具都提供了一些选项，可以查看堆、方法区等使用数据。

或者，也可以使用jmap等提供的命令，生成堆转储（ Heap Dump）文件，然后利用jhat或Eclipse MAT等堆转储分析工具进行详细分析。

如果你使用的是Tomcat、 Weblogic等Java EE服务器，这些服务器同样提供了内存管理相关的功能。

另外，从某种程度上来说， GC日志等输出，同样包含着丰富的信息。


首先，堆内部是什么结构？
对于堆内存，我在上一讲介绍了最常见的新生代和老年代的划分，其内部结构随着JVM的发展和新GC方式的引入，可以有不同角度的理解，下图就是年代视角的堆结构示意图。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191120172351641.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)


你可以看到，按照通常的GC年代方式划分， Java堆内分为：
1.新生代

新生代是大部分对象创建和销毁的区域，在通常的Java应用中，绝大部分对象生命周期都是很短暂的。其内部又分为Eden区域，作为对象初始分配的区域；两个Survivor，有时候
也叫from、 to区域，被用来放置从Minor GC中保留下来的对象。

JVM会随意选取一个Survivor区域作为“to”，然后会在GC过程中进行区域间拷贝，也就是将Eden中存活下来的对象和from区域的对象，拷贝到这个“to”区域。这种设计主要是为
了防止内存的碎片化，并进一步清理无用对象。

从内存模型而不是垃圾收集的角度，对Eden区域继续进行划分， Hotspot JVM还有一个概念叫做Thread Local Allocation Bufer（ TLAB），据我所知所有OpenJDK衍生出来
的JVM都提供了TLAB的设计。这是JVM为每个线程分配的一个私有缓存区域，否则，多线程同时分配内存时，为避免操作同一地址，可能需要使用加锁等机制，进而影响分配速
度，你可以参考下面的示意图。从图中可以看出， TLAB仍然在堆上，它是分配在Eden区域内的。其内部结构比较直观易懂， start、 end就是起始地址， top（指针）则表示已经
分配到哪里了。所以我们分配新对象， JVM就会移动top，当top和end相遇时，即表示该缓存已满， JVM会试图再从Eden里分配一块儿。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191120172701892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

2.老年代
放置长生命周期的对象，通常都是从Survivor区域拷贝过来的对象。当然，也有特殊情况，我们知道普通的对象会被分配在TLAB上；如果对象较大， JVM会试图直接分配在Eden其
他位置上；如果对象太大，完全无法在新生代找到足够长的连续空闲空间， JVM就会直接分配到老年代。


3.永久代
这部分就是早期Hotspot JVM的方法区实现方式了，储存Java类元数据、常量池、 Intern字符串缓存，在JDK 8之后就不存在永久代这块儿了。

那么，我们如何利用JVM参数，直接影响堆和内部区域的大小呢？我来简单总结一下：
- 最大堆体积
```
-Xmx value

```

- 初始的最小堆体积
```
-Xms value
```

- 老年代和新生代的比例
```
-XX:NewRatio=value
```
默认情况下，这个数值是3，意味着老年代是新生代的3倍大；换句话说，新生代是堆大小的1/4。
当然，也可以不用比例的方式调整新生代的大小，直接指定下面的参数，设定具体的内存大小数值.
```
-XX:NewSize=value
```
Eden和Survivor的大小是按照比例设置的，如果SurvivorRatio是8，那么Survivor区域就是Eden的1/8大小，也就是新生代的1/10，因为YoungGen=Eden +2*Survivor
JVM参数格式是
```
-XX:SurvivorRatio=value
```
**思考:**
如果用程序的方式而不是工具，对Java内存使用进行监控，有哪些技术可以做到?

利用JMX MXbean公开出来的api:ManagementFactory;



### Java常见的垃圾收集器有哪些？

**典型回答:**
实际上，垃圾收集器（ GC， Garbage Collector）是和具体JVM实现紧密相关的，不同厂商（ IBM、 Oracle），不同版本的JVM，提供的选择也不同。接下来，我来谈谈最主流
的Oracle JDK。
Serial GC，它是最古老的垃圾收集器， “Serial”体现在其收集工作是单线程的，并且在进行垃圾收集过程中，会进入臭名昭著的“Stop-The-World”状态。当然，其单线程设计也
意味着精简的GC实现，无需维护复杂的数据结构，初始化也简单，所以一直是Client模式下JVM的默认选项。
从年代的角度，通常将其老年代实现单独称作Serial Old，它采用了标记-整理（ Mark-Compact）算法，区别于新生代的复制算法。
Serial GC的对应JVM参数是：
```
-XX:+UseSerialGC
```

ParNew GC，很明显是个新生代GC实现，它实际是Serial GC的多线程版本，最常见的应用场景是配合老年代的CMS GC工作，下面是对应参数
```
-XX:+UseConcMarkSweepGC -XX:+UseParNewGC

```

CMS（ Concurrent Mark Sweep） GC，基于标记-清除（ Mark-Sweep）算法，设计目标是尽量减少停顿时间，这一点对于Web等反应时间敏感的应用非常重要，一直到今
天，仍然有很多系统使用CMS GC。但是， CMS采用的标记-清除算法，存在着内存碎片化问题，所以难以避免在长时间运行等情况下发生full GC，导致恶劣的停顿。另外，既然
强调了并发（ Concurrent）， CMS会占用更多CPU资源，并和用户线程争抢。

Parrallel GC，在早期JDK 8等版本中，它是server模式JVM的默认GC选择，也被称作是吞吐量优先的GC。它的算法和Serial GC比较相似，尽管实现要复杂的多，其特点是新
生代和老年代GC都是并行进行的，在常见的服务器环境中更加高效。

开启选项是：
```
-XX:+UseParallelGC
```


另外， Parallel GC引入了开发者友好的配置项，我们可以直接设置暂停时间或吞吐量等目标， JVM会自动进行适应性调整，例如下面参数：
```
-XX:MaxGCPauseMillis=value
-XX:GCTimeRatio=N // GC时间和用户时间比例 = 1 / (N+1)
```

G1 GC这是一种兼顾吞吐量和停顿时间的GC实现，是Oracle JDK 9以后的默认GC选项。 G1可以直观的设定停顿时间的目标，相比于CMS GC， G1未必能做到CMS在最好情况
下的延时停顿，但是最差情况要好很多。

G1 GC仍然存在着年代的概念，但是其内存结构并不是简单的条带式划分，而是类似棋盘的一个个region。 Region之间是复制算法，但整体上实际可看作是标记-整理（ MarkCompact）算法，可以有效地避免内存碎片，尤其是当Java堆非常大的时候， G1的优势更加明显。

G1吞吐量和停顿表现都非常不错，并且仍然在不断地完善，与此同时CMS已经在JDK 9中被标记为废弃（ deprecated），所以G1 GC值得你深入掌握。


常见的垃圾收集算法，我认为总体上有个了解，理解相应的原理和优缺点，就已经足够了，其主要分为三类：

复制（ Copying）算法，我前面讲到的新生代GC，基本都是基于复制算法，过程就如专栏上一讲所介绍的，将活着的对象复制到to区域，拷贝过程中将对象顺序放置，就可以避
免内存碎片化。
这么做的代价是，既然要进行复制，既要提前预留内存空间，有一定的浪费；另外，对于G1这种分拆成为大量region的GC，复制而不是移动，意味着GC需要维护region之间对
象引用关系，这个开销也不小，不管是内存占用或者时间开销。

标记-清除（ Mark-Sweep）算法，首先进行标记工作，标识出所有要回收的对象，然后进行清除。这么做除了标记、清除过程效率有限，另外就是不可避免的出现碎片化问题，
这就导致其不适合特别大的堆；否则，一旦出现Full GC，暂停时间可能根本无法接受。

标记-整理（ Mark-Compact），类似于标记-清除，但为避免内存碎片化，它会在清理过程中将对象移动，以确保移动后的对象占用连续的内存空间。


**在垃圾收集的过程，对应到Eden、 Survivor、 Tenured等区域会发生什么变化呢？**

这实际上取决于具体的GC方式，先来熟悉一下通常的垃圾收集流程，我画了一系列示意图，希望能有助于你理解清楚这个过程。

第一， Java应用不断创建对象，通常都是分配在Eden区域，当其空间占用达到一定阈值时，触发minor GC。仍然被引用的对象（绿色方块）存活下来，被复制到JVM选择
的Survivor区域，而没有被引用的对象（黄色方块）则被回收。注意，我给存活对象标记了“数字1”，这是为了表明对象的存活时间。

第二， 经过一次Minor GC， Eden就会空闲下来，直到再次达到Minor GC触发条件，这时候，另外一个Survivor区域则会成为to区域， Eden区域的存活对象和From区域对象，都
会被复制到to区域，并且存活的年龄计数会被加1。

第三， 类似第二步的过程会发生很多次，直到有对象年龄计数达到阈值，这时候就会发生所谓的晋升（ Promotion）过程，如下图所示，超过阈值的对象会被晋升到老年代。这个阈
值是可以通过参数指定：
```
-XX:MaxTenuringThreshold=<N>
```

后面就是老年代GC，具体取决于选择的GC选项，对应不同的算法。下面是一个简单标记-整理算法过程示意图，老年代中的无用对象被清除后， GC会将对象进行整理，以防止内存
碎片化。

通常我们把老年代GC叫作Major GC，将对整个堆进行的清理叫作Full GC，但是这个也没有那么绝对，因为不同的老年代GC算法其实表现差异很大，例如CMS， “concurrent”就
体现在清理工作是与工作线程一起并发运行的.



**总结**:
- JVM提供的收集器较多，特征不一，适用于不同的业务场景：
- Serial收集器：串行运行；作用于新生代；复制算法；响应速度优先；适用于单CPU环境下的client模式。
- ParNew收集器：并行运行；作用于新生代；复制算法；响应速度优先；多CPU环境Server模式下与CMS配合使用。
- Parallel Scavenge收集器：并行运行；作用于新生代；复制算法；吞吐量优先；适用于后台运算而不需要太多交互的场景。
- Serial Old收集器：串行运行；作用于老年代；标记-整理算法；响应速度优先；单CPU环境下的Client模式。
- Parallel Old收集器：并行运行；作用于老年代；标记-整理算法；吞吐量优先；适用于后台运算而不需要太多交互的场景。
- CMS收集器：并发运行；作用于老年代；标记-清除算法；响应速度优先；适用于互联网或B/S业务。(jdk9中已经被标记废弃)
- G1收集器：并发运行；可作用于新生代或老年代；标记-整理算法+复制算法；响应速度优先；面向服务端应用。




### Java内存模型中的happen-before是什么？

**典型回答**
Happen-before关系，是Java内存模型中保证多线程操作可见性的机制，也是对早期语言规范中含糊的可见性概念的一个精确定义。

它的具体表现形式，包括但远不止是我们直觉中的synchronized、 volatile、 lock操作顺序等方面，例如：
- 线程内执行的每个操作，都保证happen-before后面的操作，这就保证了基本的程序顺序规则，这是开发者在书写程序时的基本约定。
- 对于volatile变量，对它的写操作，保证happen-before在随后对该变量的读取操作。
- 对于一个锁的解锁操作，保证happen-before加锁操作。
- 对象构建完成，保证happen-before于fnalizer的开始动作。
- 甚至是类似线程内部操作的完成，保证happen-before其他Thread.join()的线程等。
- 这些happen-before关系是存在着传递性的，如果满足a happen-before b和b happen-before c，那么a happen-before c也成立。
前面我一直用happen-before，而不是简单说前后，是因为它不仅仅是对执行时间的保证，也包括对内存读、写操作顺序的保证。仅仅是时钟顺序上的先后，并不能保证线程交互的
可见性。


JMM内部的实现通常是依赖于所谓的内存屏障，通过禁止某些重排序的方式，提供内存可见性保证，也就是实现了各种happen-before规则。与此同时，更多复杂度在于，需要尽量
确保各种编译器、各种体系结构的处理器，都能够提供一致的行为。

以volatile为例，看看如何利用内存屏障实现JMM定义的可见性？

对于一个volatile变量：
- 对该变量的写操作之后，编译器会插入一个写屏障。
- 对该变量的读操作之前，编译器会插入一个读屏障。极客时间

内存屏障能够在类似变量读、写操作之后，保证其他线程对volatile变量的修改对当前线程可见，或者本地修改对其他线程提供可见性。换句话说，线程写入，写屏障会通过类似强迫
刷出处理器缓存的方式，让其他线程能够拿到最新数值。


### Java程序运行在Docker等容器环境有哪些新问题？

**典型回答**
对于Java来说， Docker毕竟是一个较新的环境，例如，其内存、 CPU等资源限制是通过CGroup（ Control Group）实现的，早期的JDK版本（ 8u131之前）并不能识别这些限
制，进而会导致一些基础问题：

如果未配置合适的JVM堆和元数据区、直接内存等参数， Java就有可能试图使用超过容器限制的内存，最终被容器OOM kill，或者自身发生OOM。

错误判断了可获取的CPU资源，例如， Docker限制了CPU的核数， JVM就可能设置不合适的GC并行线程数等


**知识扩展**
首先，我们先来搞清楚Java在容器环境的局限性来源， Docker到底有什么特别？

虽然看起来Docker之类容器和虚拟机非常相似，例如，它也有自己的shell，能独立安装软件包，运行时与其他容器互不干扰。但是，如果深入分析你会发现， Docker并不是一种完
全的虚拟化技术，而更是一种轻量级的隔离技术。

对于Java平台来说，这些未隐藏的底层信息带来了很多意外的困难，主要体现在几个方面：

第一，容器环境对于计算资源的管理方式是全新的， CGroup作为相对比较新的技术，历史版本的Java显然并不能自然地理解相应的资源限制。

第二， namespace对于容器内的应用细节增加了一些微妙的差异，比如jcmd、 jstack等工具会依赖于“/proc//”下面提供的部分信息，但是Docker的设计改变了这部分信息的原有
结构，我们需要对原有工具进行修改以适应这种变化。

从JVM运行机制的角度，为什么这些“沟通障碍”会导致OOM等问题呢？

你可以思考一下，这个问题实际是反映了JVM如何根据系统资源（内存、 CPU等）情况，在启动时设置默认参数

这就是所谓的Ergonomics机制，例如：
- JVM会大概根据检测到的内存大小，设置最初启动时的堆大小为系统内存的1/64；并将堆最大值，设置为系统内存的1/4。
- 而JVM检测到系统的CPU核数，则直接影响到了Parallel GC的并行线程数目和JIT complier线程数目，甚至是我们应用中ForkJoinPool等机制的并行等级。

这些默认参数，是根据通用场景选择的初始值。但是由于容器环境的差异， Java的判断很可能是基于错误信息而做出的。
**这就类似，我以为我住的是整栋别墅，实际上却只有一个房间是给我住的。**

更加严重的是， JVM的一些原有诊断或备用机制也会受到影响。为保证服务的可用性，一种常见的选择是依赖“-XX:OnOutOfMemoryError”功能，通过调用处理脚本的形式来做一
些补救措施，比如自动重启服务等。但是，这种机制是基于fork实现的，当Java进程已经过度提交内存时， fork新的进程往往已经不可能正常运行了。

根据前面的总结，似乎问题非常棘手，那我们在实践中， 如何解决这些问题呢？
- 首先，如果你能够升级到最新的JDK版本，这个问题就迎刃而解了。
- 针对这种情况， JDK 9中引入了一些实验性的参数，以方便Docker和Java“沟通”，例如针对内存限制，可以使用下面的参数设置：

```
-XX:+UnlockExperimentalVMOptions
-XX:+UseCGroupMemoryLimitForHeap
```

如果你可以切换到JDK 10或者更新的版本，问题就更加简单了。 Java对容器（ Docker）的支持已经比较完善，默认就会自适应各种资源限制和实现差异。前面提到的实验性参
数“UseCGroupMemoryLimitForHeap”已经被标记为废弃。

与此同时，新增了参数用以明确指定CPU核心的数目。
```
-XX:ActiveProcessorCount=N
```


但是，如果我暂时只能使用老版本的JDK怎么办？

这里有几个建议：

明确设置堆、元数据区等内存区域大小，保证Java进程的总大小可控。

例如，我们可能在环境中，这样限制容器内存：
```
$ docker run -it --rm --name yourcontainer -p 8080:8080 -m 800M repo/your-java-container:openjdk
```

那么，就可以额外配置下面的环境变量，直接指定JVM堆大小。
```
-e JAVA_OPTIONS='-Xmx300m'
```

明确配置GC和JIT并行线程数目，以避免二者占用过多计算资源。
```
-XX:ParallelGCThreads
-XX:CICompilerCount
```

除了我前面介绍的OOM等问题，在很多场景中还发现Java在Docker环境中，似乎会意外使用Swap。具体原因待查，但很有可能也是因为Ergonomics机制失效导致的，我建议配
置下面参数，明确告知JVM系统内存限额。
```
-XX:MaxRAM=`cat /sys/fs/cgroup/memory/memory.limit_in_bytes`
```
也可以指定Docker运行参数，例如：
```
--memory-swappiness=0
```


### 你了解Java应用开发中的注入攻击吗？

下面是几种主要的注入式攻击途径，原则上提供动态执行能力的语言特性，都需要提防发生注入攻击的可能。

首先，就是最常见的SQL注入攻击。一个典型的场景就是Web系统的用户登录功能，根据用户输入的用户名和密码，我们需要去后端数据库核实信息.














































































































