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
























