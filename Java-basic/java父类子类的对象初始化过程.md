## java父类子类的对象初始化过程

1. 基本初始化过程：

对于一个简单类的初始化过程是：

    static 修饰的模块（static变量和static 块）  ---> 按照代码顺序依次执行。

        |

    实例变量  及非static模块---> 按照代码顺序依次执行。

        |

    构造函数 ---> 执行对应的构造函数。

子类的初始化过程。

    父类static修饰的模块

        |

    子类static修饰模块

        |

    父类实例变量和非static块

        |

    父类对应构造函数。当子类对应构造函数中没有显示调用时调用的是父类默认的构造函数。

        |

    子类实例变量和非static块

        |

    子类构造函数
	
	
Class的static模块是唯一的，所以只初始化一次。所有类的实例公用Class的static模块。

static模块的初始化条件： ( From： 引用1 )

    （1）遇到new、getstatic、putstatic 或 invokestatic这4条字节码指令时，如果类还没初始化，则触发初始化。对应场景是：new实例化对象时、读或设置一个静态字段时（被final修饰，已在编译器把结果放入常量池的静态字段除外），以及调用一个类的静态方法时

    （2）对类进行反射调用时

    （3）初始化子类。但父类还没有初始化时，先触发父类初始化

    （4）当虚拟机启动时，需指定一个需要执行的主类（包含main方法），虚拟机会先初始化该类
	
	
2. 关于覆盖和隐藏

    覆盖：被覆盖的方法相当于被擦除了，不可恢复（即使类型强制转换也无法调用）。

    隐藏：一直保存的某个位置，等需要的时候可以继续使用（类型强制转换后可以调用）。

关于父类子类之间的覆盖和隐藏关系： （From ：引用2）

    1. 父类的实例变量和类变量能被子类的同名变量隐藏。 

    2. 父类的静态方法被子类的同名静态方法隐藏，父类的实例方法被子类的同名实例方法覆盖。 

    3. 不能用子类的静态方法隐藏父类的实例方法，也不能用子类的实例方法覆盖父类的静态方法，否则编译器会异常。 

    4. 用final关键字修饰的最终方法不能被覆盖。 

    5. 变量只能被隐藏不会被覆盖，子类的实例变量可以隐藏父类的类变量，子类的类变量也可以隐藏父类的实例变量。
	

涉及的典型情况：

前提：

```

class B extends A{……}

```

（1） B instanceB = new B();

instanceB的初始化过程如第一部分中：子类的初始化过程。

此时instanceB中实际上有一个super的全部信息（除了父类中被覆盖的实例方法），但是当前的引用是子类的信息（如果子类中没有的变量和方法则是从父类继承来）。



（2）A instanceA = new B();

此时父类A的变量和静态方法会将子类的变量和静态方法隐藏。instanceA此时唯一可能调用的子类B的地方就是子类B中覆盖了父类A中的实例方法。

执行 B instanceB = (B) instanceA; 后 此时instanceB相当于 B instanceB = new B();


注意：

    (1) 这里唯一的覆盖的情况：父类的实例方法被子类的同名实例方法覆盖。

    这里方法调用变量时，实际上是带有一个默认的this的。也就是此实例方法中调用的是当前Class的值。涉及到继承的情况时，要十分注意super，this的情况。

    (2) 在初始化的时候，有两个阶段。第一步是申请空间，第二步是赋值。

    具体见：

        static的值和final static值在初始化时的区别。（引用1）

        子类覆盖父类实例方法时，调用子类的实例方法，而此时子类的实例方法调用了子类中的变量（此时static变量已经初始化，但是实例变量并没有初始化）。（引用2）


这段代码来自引用2，如何精炼，如此有趣。值得一看。
```java
class Parent{
         int x=10;
         public Parent(){
              add(2);
         }
         void add(int y){
              x+=y;
         }
    }
     class Child extends Parent{
         int x=9;
         void add(int y){
              x+=y;
         }
         public static void main(String[] args){
              Parent p=new Child();
              System.out.println(p.x);
         } 
    }
```
引用 1. 类加载机制：

http://blog.csdn.net/kai_wei_zhang/article/details/8234146 

引用 2. 隐藏和覆盖，以及一个好例子：

http://www.cnblogs.com/binjoo/articles/1585342.html

引用 3. 隐藏和覆盖

http://renyanwei.iteye.com/blog/258304 

引用 4. 基本的初始化过程

http://www.cnblogs.com/miniwiki/archive/2011/03/25/1995615.html 

来源:https://my.oschina.net/beabetterman/blog/228324