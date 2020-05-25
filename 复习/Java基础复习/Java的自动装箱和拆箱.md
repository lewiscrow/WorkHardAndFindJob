## Java的自动装箱和拆箱(Autoboxing and unboxing)
#### 目录
* [一、什么是自动装箱自动拆箱](#what)
* [二、自动装箱自动拆箱的实质](#how)
* [三、需要注意的点](#wpoints)
* [四、其他一些问题](#ralated)
* [五、总结](#conclusion)

#### <a id="what">一、什么是自动装箱自动拆箱</a>
自动装箱自动拆箱是在JDK5以后引入的一个特性。在学习Java的过程中，我们认识到有八种基础类型，以及他们对应的包装类型。

基本类型|包装类型
---|:--:
byte|Byte
short|Short
int|Integer
long|Long
float|Float
double|Double
char|Character
boolean|Boolean

简单的来说，自动装箱就是讲基础类型转换成包装类型，自动拆箱就是讲包装类型转换成基础类型。说是自动的，是因为我们没有在代码中显示的调用相关方法，转换过程由编译器自动帮我们完成。
比如下面代码：
```
	int i0 = 0;    //创建基础类型
    Integer i1 = i0;    //自动装箱
    int i2 = i1;    //自动拆箱
```

#### <a id="how">二、自动装箱自动拆箱的实质</a>
想要知道自动拆箱自动装箱的实质，我们就得跟着代码走一走。这里我们用 int 和 Integer 类型为例（用的比较多），其他的基本类型和包装类型大家自己可以试一下。
比如下面这个文件，SolutionTest.java
```
public class SolutionTest {
	 public static void main(String[] args) {
		 int i0 = 0;    //创建基础类型
		 Integer i1 = i0;    //自动装箱
		 int i2 = i1;    //自动拆箱
	 }
}
```
我们对其进行编译和反编译后，得到的结果
```
Compiled from "SolutionTest.java"
public class SolutionTest {
  public SolutionTest();
    Code:
       0: aload_0
       1: invokespecial #1          // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: iconst_0
       1: istore_1
       2: iload_1
       3: invokestatic  #2          // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       6: astore_2
       7: aload_2
       8: invokevirtual #3          // Method java/lang/Integer.intValue:()I
      11: istore_3
      12: return
}
```
我们可以看到在`Integer i1 = i0;`时，系统执行的是 Integer.valueOf 的方法，在`int i2 = i1;`时，系统执行了 Integer.intValue 方法。
哪路红豆，原来自动装箱和自动拆箱其实就是调用了 Integer 类的两个方法啊！那我们再来看看 Integer 的源码：
```
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
这个方法里面实现了两个步骤：
1、首先判断常量池中有没有值为 i 的对象，有的话就返回该对象。（ps：IntegerCache是Integer的静态内部类，cache是一个Integer类型的数组）
2、如果常量池中不存在值值为 i 的对象，那就建一个并返回呗。

然后是intValue方法：
```
public int intValue() {
    return value;
}
```
这个就直接返回了 Integer 的内部成员 value。值得一提的是，Integer 里面保存值的其实还是一个 int 类型的 value ，而 value 是用 final 修饰的，就是说你无法改变 Integer 实例的值，你看起来改变值的其实都是返回了另一个实例（这跟 IntegerCache 的常量池有关，思考一下）。
好了，这下就明白为啥了。

#### <a id="points">三、需要注意的点</a>
注意的点主要分为两块：1、实际开发过程中遇到的问题；2、面试可能会问到的问题。
实际开发中的问题主要涉及到的就是 Object 类的 equals 方法和 == 运算符的区别。比如下面代码：
```
	int i0 = 0;    
    Integer i1 = i0;
    int i2 = i1; 
    System.out.println(i0 == i1);    // true
    System.out.println(i1 == i2);    // true
    System.out.println(i0 == i2);    // true
    System.out.println(i0.equals(i1));    // 编译错误
    System.out.println(i1.equals(i2));    // true
    System.out.println(i2.equals(i0));    // 编译错误
```
除了不能编译的，其他的都为 true。在反编译之前，我们先自己结合上面的知识思考一下为什么呢？
没错，因为在使用 == 运算符的时候自动拆箱装箱了，而 equals 方法代码如下:
```
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```
看的出来本质上还是 int 类型的比较。
把编译错误的代码注释掉，反编译看一下猜想是否正确。
```
Compiled from "SolutionTest.java"
public class SolutionTest {
  public SolutionTest();
    Code:
       0: aload_0
       1: invokespecial #1          // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: iconst_0
       1: istore_1
       2: iload_1
       3: invokestatic  #2          // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       6: astore_2
       7: aload_2
       8: invokevirtual #3          // Method java/lang/Integer.intValue:()I
      11: istore_3
      12: getstatic     #4          // Field java/lang/System.out:Ljava/io/PrintStream;
      15: iload_1
      16: aload_2
      17: invokevirtual #3          // Method java/lang/Integer.intValue:()I
      20: if_icmpne     27
      23: iconst_1
      24: goto          28
      27: iconst_0
      28: invokevirtual #5          // Method java/io/PrintStream.println:(Z)V
      31: getstatic     #4          // Field java/lang/System.out:Ljava/io/PrintStream;
      34: aload_2
      35: invokevirtual #3          // Method java/lang/Integer.intValue:()I
      38: iload_3
      39: if_icmpne     46
      42: iconst_1
      43: goto          47
      46: iconst_0
      47: invokevirtual #5          // Method java/io/PrintStream.println:(Z)V
      50: getstatic     #4          // Field java/lang/System.out:Ljava/io/PrintStream;
      53: iload_1
      54: iload_3
      55: if_icmpne     62
      58: iconst_1
      59: goto          63
      62: iconst_0
      63: invokevirtual #5          // Method java/io/PrintStream.println:(Z)V
      66: getstatic     #4          // Field java/lang/System.out:Ljava/io/PrintStream;
      69: aload_2
      70: iload_3
      71: invokestatic  #2          // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      74: invokevirtual #6          // Method java/lang/Integer.equals:(Ljava/lang/Object;)Z
      77: invokevirtual #5          // Method java/io/PrintStream.println:(Z)V
      80: return
}

```
从结果看的出来，在 equals 的时候，int 参数先被自动装箱为 Integer 类型，然后进行比较。在 == 比较时，Integer 类型自动拆箱。

面试中可能碰到的问题，比如说：`Integer i1 = 0;`这句代码经过了什么流程，常量池什么时候建立、初始化的大小、什么时候扩充、扩充多少等问题。这些看源码就好啦。
1、`Integer i1 = 0;`这句代码经过了什么流程？
调用valueOf方法 -> 判断是否在常量池中 ->有？返回常量池中对象：返回新建对象
2、常量池什么时候建立？
常量池是 Integer 的静态内部类 IntegerCache 静态方法中初始化的，那么也就是说第一次 Integer 类加载的时候就全部加载了。
3、初始化的大小？
默认是 -127 ~ 128。但看代码可以知道，这个虚拟机中的属性有关：
```
static {
    // high value may be configured by property
    int h = 127;
    String integerCacheHighPropValue =
        VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    if (integerCacheHighPropValue != null) {
        try {
            int i = parseInt(integerCacheHighPropValue);
            i = Math.max(i, 127);
            // Maximum array size is Integer.MAX_VALUE
            h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        } catch( NumberFormatException nfe) {
            // If the property cannot be parsed into an int, ignore it.
        }
    }
    high = h;

    cache = new Integer[(high - low) + 1];
    int j = low;
    for(int k = 0; k < cache.length; k++)
        cache[k] = new Integer(j++);

    // range [-128, 127] must be interned (JLS7 5.1.7)
    assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
}
```
4、什么时候扩充以及扩充多少？
啊。。。源码中没看到，不太清楚。

#### <a id="related">四、其他一些问题</a>
1、在使用集合类时，都用 Integer 而不能用 int。
2、下面代码能编译通过但是运行时会报空指针异常：
```
	Integer i1 = null;
	int i2 = i1;
```
原因是自动拆箱时对一个 null 对象进行 intValue ，自然会报错。

#### <a id="conclusion">五、总结</a>
1、自动装箱就是讲基础类型转换成包装类型，自动拆箱就是讲包装类型转换成基础类型。
2、自动拆箱装箱实际上是调用了包装类的方法。
3、常量池的存在以及使用。
4、== 比较时: 基础类型之间不用说，基础类型和包装类型会进行自动拆箱，包装类型和包装类型之间是正常类型的比较。
5、equals 方法，参数为基础类型时会进行自动装箱。装箱后变成取 intValue 后基础类型值的比较。