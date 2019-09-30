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
原则上， **建议避免无意中的装箱、拆箱行为**，尤其是在性能敏感的场合，创建10万个Java对象和10万个整数的开销可不是一个数量级的，不管是内存使用还是处理速度，光是对象头
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

















