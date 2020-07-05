## Java基础复习
#### 目录
* [概述](#absrtact)
* [一、数据类型](#data_type)
    * [基本类型](#basic_type)
    * [包装类型](#wrap_type)
    * [缓存池](#cache_pool)
* [二、String相关](#string)
    * [String 的内部存储](#string_core)
    * [String 重要方法](#string_methods)
    * [String 的不可变性](#string_immutable)
    * [String 常量池](#string_pool)
    * [String、StringBuilder 和 StringBuffer 的比较](#string_compare)
* [三、关键字](#key_word)
	* [关键字简单介绍](#key_word_table)
	* [部分关键字详解](#key_word_detail)
	    * [break](#break)
	    * [final](#final)
	    * [finally](#finally)
	    * [static](#static)
	    * [synchronized](#synchronized)
	    * [volatile](#volatile)
* [四、三大特性](#characteristic)
	* [封装](#encapsulation)
		* [访问修饰符](#keyword_access)
		* [this 关键字](#keyword_this)
		* [内部类](#innerclass)
	* [继承](#inheritance)
		* [继承的特性](#inheritance_charactertistic)
		* [方法的重写](#inheritance_override)
		* [初始化](#inheritance_initial)
		* [继承相关关键字](#inheritance_key_words)
	* [多态](#polymorphism)
		* [引用多态](#polymorphism_reference)
		* [方法多态](#polymorphism_method)
		* [引用类型转换](#polymorphism_reference_transfer)
		* [抽象类](#polymorphism_absrtact)
		* [接口](#polymorphism_interface)
		* [接口和抽象类的区别](#polymorphism_difference)
* [五、异常](#throwable)
	* [异常机制的概述](#throwable_abstract)
	* [异常的结构](#throwable_structure)
		* [Throwable](#throwable_throwable)
		* [Error](#throwable_error)
		* [Exception](#throwable_exception)
* [六、反射](#reflection)
	* [反射的主要用途](#relflection_usage)
	* [反射的基本使用](#relflection_how)
		* [获得 Class 对象](#relflection_how_class)
		* [判断一个对象是不是某个类的实例](#relflection_how_isinstance)
		* [实例的创建](#relflection_how_newinstance)
		* [获取成员和使用](#relflection_how_fields&methods)
	* [反射的一些注意事项](#relflection_tips)
* [七、泛型](#generics)
	* [泛型详解](#generics_detail)
		* [概述](#generics_detail_abstract)
		* [一个栗子](#generics_detail_example)
		* [特性](#generics_detail_characteristic)
		* [泛型的使用](#generics_detail_usage)
			* [泛型类](#generics_detail_usage_class)
			* [泛型接口](#generics_detail_usage_interface)
			* [泛型通配符](#generics_detail_usage_wildcards)
			* [泛型方法](#generics_detail_usage_methods)
				* [泛型方法的基本用法](#generics_detail_usage_methods_usage)
				* [类中的泛型方法](#generics_detail_usage_class_methods)
				* [泛型方法与可变参数](#generics_detail_usage_methods&params)
				* [静态方法与泛型](#generics_detail_usage_methods&params)
				* [泛型方法总结](#generics_detail_usage_method_conclusion)
		* [泛型上下边界](#generics_detail_bound)
		* [关于泛型数组要提一下](#generics_detail_array)
		* [最后](#generics_detail_conclusion)
	* [泛型相关问题](#generics_questions)
* [八、注解](#annotation)
	* [注解基础](#annotation_basic)
		* [什么是注解](#annotation_basic_what)
		* [注解原理](#annotation_basic_principle)


### <a id="data_type">一、数据类型</a>
本章主要介绍了基本类型、包装类型以及相关的缓存池的内容，涉及到基本的存储空间、自动装箱拆箱、编译反编译操作等。



#### <a id="basic_type">基本类型</a>

Java有八种基本类型：

类型名称|字节空间|使用场景
---|:--:|---:
byte|1字节（8 bit）|存储字节数据（较常用）
short|2字节（16 bit）|兼容性考虑（很少使用）
int|4字节（32 bit）|存储普通整数（常用）
long|8字节（64 bit）|存储长整数（常用）
float|4字节（32 bit）|存储浮点数（不常用）
double|8字节（64 bit）|存储双精度浮点数（常用）
char|2字节（16 bit）|存储一个字符（常用）
boolean|1或4字节（8或32 bit）|存储逻辑变量（true、false）（常用）

* **byte：** byte 又称字节，为最小的基本类型，取值范围-2^7~2^7-1，赋值方式也有两种，一种直接整数，一种以2进制形式。
* **short：** 为2字节16位，但是用的比较少，一般直接用 int 类型。一般只有在使用大数组的时候，为了节约空间使用 short 代替 int。
* **int：** int 是最常用的整型类型，一个 int 变量占4个字节32位，最大表示范围：-2^31~2^31-1，赋值方式可以是一般的十进制整数或者是16进制形式（0x或者0X开头），也可以是8进制形式（0开头）。
* **long：**如果 int 类型表示范围不够，可以使用 long 类型，占8字节64位，最大表示范围：-2^63~2^63-1，但是使用时需要以小写l或者大写L做结尾，否则会有编译错误。常用于存储时间及日期，jdk中的方法 System.currentTimeMills() 。
* **float：**又称为浮点类型，即小数类型，4字节32位运用方式和 double 类型相似，但是使用时需要在最后加上 f 或者大写 F ，一般情况用 double 即可。
* **double：**也叫作双精度浮点类型，8字节64位精度是 float 的两倍，大多数场合使用 double 来表示浮点数即小数，浮点数具有两种写法，一般写法和科学计数法。
* **char：**char类型实际上叫做字符类型，是一个16位无符号整数，这个值是对应字符的编码，采用 unicode 字符集编码。char类型变量赋值可以采用三种形式：字符直接量，整型直接量以及 unicode 编码赋值。同时 char 类型中有不方便输出的字符，需要进行转义。
* **boolean：** boolean 类型的变量一般用于关系运算中，只有两个值 true 和 false ，表示条件成立或者不成立。



#### <a id="wrap_type">包装类型</a>
[Java自动装箱拆箱详解](https://www.cnblogs.com/lewisyoung/p/12769084.html)

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



#### <a id="cache_pool">缓存池</a>

缓存池的内容在上面包装类型的博客里面有提及，这里再来讲一下，作为数据类型的收尾。
这边有一段代码：

```java
    Integer x1 = 100;
    Integer y1 = 100;
    Integer x2 = 130;
    Integer y2 = 130;
    System.out.println(x1 == y1);    // true
    System.out.println(x2 == y2);    // flase
```

上面的比较都是基于 Integer 类型的比较，因此不会自动拆包使用 int 值而是比较两个示例是否是同一个。出现这个输出结果原因，就在于缓存池。
上面代码前四行会触发自动装箱，调用 Integer.valueOf() 方法，我们再来看看这个方法：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

可以看到，返回的内容跟缓存池有关。当参数 i 在缓存池范围内，则返回缓存池内的实例。参数 i 不在缓存池范围内，则返回新建对象。
查看静态内部类 IntegerCache 的静态变量和方法：

```java
static final int low = -128;
static final int high;
static final Integer cache[];
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
```

可以看到下限是 -128定死了，上限由虚拟机定，否则为127。值得一提的是，**在所有包装类中，只有 Integer 的缓存池上限可以由虚拟机指定。**在启动 jvm 的时候，通过 `-XX:AutoBoxCacheMax=<size> `来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界。
基本类型对应的缓冲池如下：

* boolean values true and false
* all byte values
* short values between -128 and 127
* int values between -128 and 127
* char in the range \u0000 to \u007F

在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以直接使用缓冲池中的对象。



-----------
### <a id="string">二、String相关</a>

本章主要介绍了 String 的主要内容，包括 String 的实现和重要方法源码解读、String 的特性以及用处、常量池、老生常谈的 String、StringBuilder、StringBuffer 的异同等。



#### <a id="string_core">String 的内部存储</a>

在 Java 8中，String 的部分代码如下：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

可以看到，String 被声明为 final，因此不能被继承。并且内部使用 final 修饰 char 数组来存储数据，说明 String 是不可变的。
在 Java 9中，String 内部使用 final 修饰的 byte 数组保存，同时使用 coder 来标识使用了哪种编码。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```



#### <a id="string_methods">String 重要方法</a>

在 String 类型中，我们常用的方法有很多，这边只讲几个值得注意的方法。

**equals方法**

equals 方法是 Object 类的方法，String 重写了 equals 方法，代码如下：

```java
public boolean equals(Object anObject) {
	    // 先比较比较两个 String 对象内存地址，如果内存地址相同那么必定相等
        if (this == anObject) {
            return true;
        }
        // 判断参数对象类型，如果不是 String 那么肯定不相等，否则进行 char 数组元素比较
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

**compareTo**

compareTo 方法比较了两个字符串的“大小”，这个“大小”比较的代码如下：

```java
    public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        while (k < lim) {
        	// 逐个比较 char 的大小，char 的大小由 ascii 码决定
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        // 如果还没一个字符串比完了还没比完，就返回长度的比较
        return len1 - len2;
    }
```

由这段代码就可以知道为什么下面代码的输出是这样子了：

```java
	String s0 = "2";
	String s1 = "10";
	String s2 = "abcd";
	String s3 = "abcde";
	System.out.println(s0.compareTo(s1));    // 1
	System.out.println(s2.compareTo(s3));    // -1
```

**hashCode**

我们知道 String 常常用来作为 HashMap 的键，因为 String 不可变因此 hash 值也不变。这里我们看一下 Java8 中 hash 值的计算方法：

```java
	public int hashCode() {
        int h = hash;
        // 如果 hash 值存在或者 String 长度为0，则直接返回计算好的 hash 值，避免多次计算
        // 否则，hash 值的计算公式为 s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

**intern**

这是个非常厉害的方法，而且我觉得面试经常会问到。
首先是代码：

```java
	public native String intern();
```

这是一个 native 方法，虚拟机会去调用 C++。因此具体实现我们就不看了，我们就看看这个方法干了什么。其实这个方法干的事情很简单，就是将 String 对象放到常量池，并返回一个对它的引用。（[常量池内容介绍](#string_pool)）
接下来是几段常问的代码：

```java
	String s1 = new String("123");
	String s2 = new String("123");
	System.out.println(s1 == s2);    // false
	String s3 = s1.intern();
	System.out.println(s3 == "123");    // true
	String s4 = s2.intern();
	System.out.println(s3 == s4);    // true
    String s5 = new String("123") + new String("123");
	String s6 = new String("123123");
	String s7 = "123123";
	String s8 = s3 + s4;
	System.out.println(s5 == s6);    // false
	System.out.println(s6 == s7);    // false
	System.out.println(s7 == s8);    // false
	final String s9 = "123";
	final String s10 = "123";
	String s11 = s9 + s10;
	System.out.println(s7 == s11);    //true
```

s1 != s2，因为两个分别指向堆中的不同对象；
s3 == "123"，因为 s3 是 s1 intern()的结果，指向常量池中 "123" 对象的地址。故相等
s3 == s4，两者都指向常量池中 "123"的地址
s5 != s6，这里我们反编译一下：

```java
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=5, locals=3, args_size=1
         0: new           #16                 // class java/lang/StringBuilder
         3: dup
         4: new           #18                 // class java/lang/String
         7: dup
         8: ldc           #20                 // String 123
        10: invokespecial #22                 // Method java/lang/String."<init>":(Ljava/lang/String;)V
        13: invokestatic  #25                 // Method java/lang/String.valueOf:(Ljava/lang/Object;)Ljava/lang/String;
        16: invokespecial #29                 // Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
        19: new           #18                 // class java/lang/String
        22: dup
        23: ldc           #20                 // String 123
        25: invokespecial #22                 // Method java/lang/String."<init>":(Ljava/lang/String;)V
        28: invokevirtual #30                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        31: invokevirtual #34                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        34: astore_1
        35: ldc           #38                 // String 123123
        37: astore_2
        38: getstatic     #40                 // Field java/lang/System.out:Ljava/io/PrintStream;
		...
}

```

可以发现，String操作所谓的 + 号，其实是新建了一个 StringBuilder 并调用 append 方法和 toString 方法。因此返回的是新的 String 对象，故显然s5 != s6
s6 != s7，诶？这个我为什么要再做一遍？算了，s6指向堆中新建的对象，s7指向常量池中的对象
s7 != s8，这个反编译后跟 s5 与 s6 的原理相同
s7 == s11，这个厉害了。反编译后代码如下：

```java
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=5, args_size=1
         0: ldc           #16                 // String 123123
         2: astore_1
         3: ldc           #18                 // String 123
         5: astore_2
         6: ldc           #18                 // String 123
         8: astore_3
         9: ldc           #16                 // String 123123
        11: astore        4
        13: getstatic     #20                 // Field java/lang/System.out:Ljava/io/PrintStream;
        16: aload_1
        17: aload         4
        19: if_acmpne     26
        22: iconst_1
        23: goto          27
        26: iconst_0
        27: invokevirtual #26                 // Method java/io/PrintStream.println:(Z)V
        30: return
        LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      31     0  args   [Ljava/lang/String;
            3      28     1    s7   Ljava/lang/String;
            6      25     2    s9   Ljava/lang/String;
            9      22     3   s10   Ljava/lang/String;
           13      18     4   s11   Ljava/lang/String;
```

可以看到，用 final 修饰的 String 相加在编译后并不是调用 StringBuilder 的 append 方法，而是直接用了常量池中的值！神奇的编译器。



#### <a id="string_immutable">String 的不可变性</a>

**不可变性的原因**

String 的不可变性相信大家都知道，一问，很多人会说，“因为 String 保存的时候 char 数组是用 final 修饰的”。其实这么说并不准确。String 的不可变性是 Java 工程师数据类型构造能力的体现，而不仅仅是一个 final。比如说，虽然 char 数组是用 final 修饰的，但这仅仅保证了 value[] 不会再指向其他数据的地址而已，如果我们强行 value[0] = 'a'，这样还是可以做到对 String 的改变的。

String 不可变性的原因包括：
* 类型使用 final 修饰，保证了类不会被继承，避免了别人修改 String 类型的能力；
* value 数组使用 final 修饰，保证数组不会指向别的数组；
* value 数组使用 private 修饰，保证别人无法直接使用数组进行赋值；
* String 类的方法中避免了对 value 内容的直接暴露，避免了别人趁机修改。

看到了这里，真的是服了。。。

**不可变性的好处**

String 类型不可变性的好处主要有以下两点：
* 安全：可以用作多线程中、可以放心用作如 HashMap、HashSet 的键值
* 效率：不用反复创建相同的对象
* 节约空间：同上



#### <a id="string_pool">String 常量池</a>

先在这里声明，网上一些博客说，字符串常量池是在运行时常量池中的，而运行时常量池是在方法区中的，因此字符串常量池也是在方法区中的，这个说法没有问题，因为虚拟机是在进化的！！在 Java 7 以前，字符串常量池与运行时常量池还没有分家，在 7 以后，字符串常量池就被放到了堆当中（我们知道，堆是放对象和数组的，这部分空间比方法区大多了）。

解决了在哪里的问题，我们来说说是什么。在写程序时，我们写了一大堆代码，其中有部分内容是不会变的，比如说类型、方法名、修饰符、常量等。在一个类还没有被加载时，这个类的相关的不会变的内容都放在静态常量池中（即.class中），加载之后，相关的内容会被加载在运行时常量池（在方法区）中，而有关 String 的常量，就被放到了堆中（上面说了 Java 7 以后字符串常量池被放到了堆中）。
解决了是什么，我们再来说说为什么。建立字符串常量池的原因，还是因为代码中 String 用的太多了，建立常量池的好处就在于节省了空间和避免反复创建。

下面来一个问题：

```java
    // 下面这段代码发生了啥？
    String s1 = new String("123");
```

首先在堆上建立了一个对象，然后查找常量池中是否存在 String "123"，有的话就把这个对象作为参数，调用构造函数 String()。
空口无凭，反编译为证：

```java
Compiled from "Test.java"
public class Test
 ...
Constant pool:
   ...
  #16 = Class              #17            // java/lang/String
  #17 = Utf8               java/lang/String
  #18 = String             #19            // 123
  #19 = Utf8               123
  #20 = Methodref          #16.#21        // java/lang/String."<init>":(Ljava/lang/String;)V
  #21 = NameAndType        #5:#22         // "<init>":(Ljava/lang/String;)V
  ...
{
  ...

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: new           #16                 // class java/lang/String
         3: dup
         4: ldc           #18                 // String 123
         6: invokespecial #20                 // Method java/lang/String."<init>":(Ljava/lang/String;)V
         9: astore_1
        10: return
      LineNumberTable:
        line 6: 0
        line 7: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  args   [Ljava/lang/String;
           10       1     1    s1   Ljava/lang/String;
}

```

在 Constant Pool 中，#19 存储这字符串字面量 "123"，#18 是 String Pool 的字符串对象，它指向 #19 这个字符串字面量。在 main 方法中，0: 行使用 new #16 在堆中创建一个字符串对象 #17，并且使用 ldc #18 将 String Pool 中的字符串对象作为 String 构造函数的参数。

以下是 String 构造函数的源码，可以看到，在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 数组内容，而是都会指向同一个 value 数组。

```java
	public String(String original) {
	    this.value = original.value;
	    this.hash = original.hash;
	}
```



#### <a id="string_compare">String、StringBuilder 和 StringBuffer 的比较</a>

1、可变性
* String 不可变
* StringBuffer 和 StringBuilder可变

2、线程安全
* String 不可变，因此是线程安全的
* StringBuilder 不是线程安全的
* StringBuffer 是线程安全的，内部使用 synchronized 进行同步

**对于三者使用的总结**
如果要操作少量的数据用 = String
单线程操作字符串缓冲区 下操作大量数据 = StringBuilder
多线程操作字符串缓冲区 下操作大量数据 = StringBuffer



###  <a id="#key_word">三、关键字</a>

#### <a id="#key_word_table">关键字简单介绍</a>

Java关键字一共53个，其中包含了两个保留字 `goto, const`。

关键字|含义
---|:--:
abstract|表明类或者成员方法具有抽象属性
assert|断言，用来进行程序调试
boolean|基本数据类型之一，布尔类型
[break](#break)|提前跳出一个块
byte|基本数据类型之一。字节类型
case|用来 switch 语句中，表示其中一个分支
catch|用在异常处理中，用来捕获异常
char|基本数据类型之一，字符类型
class|申明一个类
const|保留关键字，没有具体含义
continue|回到一个块的开始处
default|默认。例如用在 swtich 语句中表明一个默认的分支
do|用在 do-while 循环结构中
double|基本数据类型之一，双精度浮点数类型
else|用在条件语句中，表明当条件不成立时的分支
enum|枚举
extends|表明一个类型是另一个类型的子类型，这里常见的类型有类和接口
[final](#final)|用来说明最终属性，表明一个类不能派生出子类，或者成员方法不能被覆盖，或者成员变量的值不能被改变，用来定义常量
[finally](#finally)|用于处理异常情况，用来生命一个基本肯定会执行到的语句
float|基本数据类型之一，单精度浮点数类型
for|一种循环结构的引导词
goto|保留关键字，没有具体含义
if|条件语句的引导词
implements|表明一个类实现了给定的接口
import|表明要访问指定的类和包
instanceof|用来测试一个对象是否是指定类型的实例对象
int|基本数据类型之一，整数类型
interface|接口
long|基本数据类型之一，长整数类型
native|用来声明一个方法是由其他语言（C、C++等）实现的
new|用来创建新的实例对象
package|包
private|一种访问控制方式：私有模式
protect|一种访问控制方式：保护模式
public|一种访问控制方式：公有模式
return|从成员方法中返回数据
short|基本数据类型之一，短整数类型
[static](#static)|表明具有静态属性
strictfp|用来声明FP_strict（单精度或双精度浮点数）表达式遵循 IEEE 754 算数规范
super|表明当前对象的父类型的引用或者父类型的构造方法
switch|分支语句结构的引导词
[synchronized](#synchronized)|表明一段代码需要同步执行
this|指向当前实例对象的引用
throw|抛出一个异常
throws|声明在当前定义的成员方法中有需要抛出的异常
transient|申明不用序列化的成员
try|尝试一个可能抛出异常的程序块
void|声明当前成员方法没有返回值
[volatile](#volatile)|表明两个或者多个变量必须同步地发生变化
while|用在循环结构中



#### <a id="key_word_detail">部分关键字详解</a>

由于关键字太多，且部分关键字没有什么好说的，本文就挑几个原理比较复杂的或者面试经常问到的关键字进行深入了解。

**<a id="break">break</a>**

我们都知道，break 常用语 while、for 等循环以及 switch 分支中，但是奇怪的是很少有人知道 break 和 java 标签（label）的配合使用。。。

比如下面这段代码：

```java
	for(...){
		for(...){
			...
			break;
		}
	}
```

break 只能跳出当前代码块，怎么跳出外层循环呢？我们可以用标签。

```java
	label1:for(...){
		label2:for(...){
			...
			break label1;
		}
	}
```

continue 也一样，也可以配合标签使用。

**<a id="final">final</a>**

final 可以修饰的对象有：1、数据；2、方法；3、类

1、数据
声明数据为常量，可以使编译时常量，也可以是运行时被初始化后不能被改变的常量。
* 对于基本类型，final 使数值不变；
* 对于引用类型，final 使引用不变，也就是不能引用其他对象，但是被引用的对象本身是可以修改的。

```java
	final int x = 1;
	// x = 2;  // cannot assign value to final variable 'x'
	final A y = new A();
	y.a = 1;
```

2、方法
声明方法不能被子类重写。
private 方法隐式地被指定为 final，如果子类中定义的方法和父类中的一个 private 方法签名相同，此时子类的方法不是重写父类方法，而是在子类中定义了一个新的方法。

3、类
声明类不允许被继承（如 String）。

**<a id="finally">finally</a>**
finally 关键字经常给人一种“是在 try...catch 语句后肯定会执行的部分”的感觉，但其实 finally 修饰的代码块并不一定会被执行，比如下面三种情况：

* case1：try-catch 异常退出

```java
	try{
	  ...
	  System.exit(1);
	}finally{
	  打印("finally");
	}
```

* case2：无限循环

```java
	try{
	  while(true);
	}
	finally{
		...
	}

```

* case1：try-catch 异常退出
    执行try-catch的线程被杀死

**<a id="static">static</a>**

1、静态变量
* 静态变量，又称类变量，也就是说这个变量属于类的，类的所有实例都共享静态变量，可以通过类名来访问它。静态变量在内存中只存在一份。（java 8 以后，静态成员也从方法区转到堆中了）
* 实例变量，每创建一个实例就会产生一个实例变量，它与该实例同生共死

```java
public class A {

    private int x;         // 实例变量
    private static int y;  // 静态变量

    public static void main(String[] args) {
        // int x = A.x;  // Non-static field 'x' cannot be referenced from a static context
        A a = new A();
        int x = a.x;
        int y = A.y;
    }
}
```

2、静态方法

静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。

```java
public abstract class A {
    public static void func1(){
    }
    // public abstract static void func2();  // Illegal combination of modifiers: 'abstract' and 'static'
}
```

只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字，因此这两个关键字与具体对象关联。

```java
public class A {

    private static int x;
    private int y;

    public static void func1(){
        int a = x;
        // int b = y;  // Non-static field 'y' cannot be referenced from a static context
        // int b = this.y;     // 'A.this' cannot be referenced from a static context
    }
}
```

3、静态语句块

静态语句块在类初始化时运行一次。
如下面代码

```java
public class A {
    static {
        System.out.println("123");
    }

    public static void main(String[] args) {
        A a1 = new A();
        A a2 = new A();
    }
}
```

输出只有一个`123`

4、静态内部类

非静态内部类依赖于外部类的实例化，也就是说需要先创建外部类的实例，才能用这个实例去创建非静态内部类。而静态内部类不需要。

```java
public class OuterClass {

    class InnerClass {
    }

    static class StaticInnerClass {
    }

    public static void main(String[] args) {
        // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        StaticInnerClass staticInnerClass = new StaticInnerClass();
    }
}
```

静态内部类不能访问外部类的非静态的变量和方法。

5、静态导包

在使用静态变量和方法时不用再指明 ClassName，从而简化代码，但可读性大大降低。

```java
    import static com.xxx.ClassName.*
```

6、初始化顺序

静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于他们在代码中的顺序。（这边篇幅已经很长了，就不用实验证明了。大家可以自己去做试试看）

```java
public static String staticField = "静态变量";
```

```java
static {
    System.out.println("静态语句块");
}
```

```java
public String field = "实例变量";
```

```java
{
    System.out.println("普通语句块");
}
```

最后才是构造函数的初始化

```java
public InitialOrderTest() {
    System.out.println("构造函数");
}
```

存在继承的情况下，初始化顺序为：
* 父类（静态变量、静态语句块）
* 子类（静态变量、静态语句块）
* 父类（实例变量、普通语句块）
* 父类（构造函数）
* 子类（实例变量、普通语句块）
* 子类（构造函数）

**<a id="synchronized">synchronized</a>**

（本节需要并发编程的基础知识）
synchronized 关键字最主要有以下三种应用方式：
* 修饰实例方法，将当前实例变成锁，进入同步代码前要获得当前实例(this)的锁；
* 修饰静态方法，将当前类对象变成锁，进入同步代码块前要获得当前类对象（.class）的锁；
* 修饰代码块，指定加锁对象，对给定对象加锁，进入同步代码块前要获得给定对象的锁

**synchronized 作用于实例方法**

所谓的实例对象锁就是用 synchronized 

```java
public class Test implements Runnable {
	
	//共享资源（临界资源）
	public Integer i = 0;
	
	private synchronized void increase() {
		i++;
	}
	
	@Override
	public void run() {
		// TODO Auto-generated method stub
		for(int i=0;i<10000;i++) {
			increase();
		}
		
	}

	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		Test test =  new Test();
		Thread t1 = new Thread(test);
		Thread t2 = new Thread(test);
		t1.start();
		t2.start();
		t1.join();
		t2.join();
		System.out.println(test.i);
	}

}
```

输出`20000`。

在上面代码中，我们创建了一个实现 Runnable 接口的类 Test，其中实现了 synchronized 修饰的方法 increase，负责对共享资源 i 进行 i++ 操作。我们都知道，由于 i++ 操作不具有原子性，因此在多线程进行 increase 方法调用时，会出现脏数据的现象，导致的结果就是最后输出内容与我们逻辑上设置的不同，如上述代码结果基本上都会小于 20000 。但是这里我们结果无论运行多少遍，都是2000，这是因为我们使用了 synchronized 关键字对 increase 方法进行修饰。上面我们讲了，synchronized 修饰普通成员方法时，锁是类的实例，当一个线程拿到这个锁的时候，该线程可以执行 synchronized 修饰的方法，其他线程没有锁于是他们不能执行 i++。这就保证了在一个时间点有且仅有一个线程可以进行完整的 i++ 操作。不过其他没有被 synchronized 修饰的成员方法，其他线程还是可以正常调用的。

**synchronized 作用于静态方法**

所谓的类对象锁就是用 synchronized 修饰类中的静态方法（和上面正好相反），此时该类（class）就变成了锁。拥有该类对象锁的线程才能进行被调用成员方法的运行。为什么会需要类对象锁呢？看下面代码：

```java
public class Test implements Runnable {
	
	public static Integer i = 0;
	
	private synchronized void increase() {
		i++;
	}
	
	@Override
	public void run() {
		// TODO Auto-generated method stub
		for(int i=0;i<10000;i++) {
			increase();
		}
		
	}

	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		Thread t1 = new Thread(new Test());
		Thread t2 = new Thread(new Test());
		t1.start();
		t2.start();
		t1.join();
		t2.join();
		System.out.println(i);
	}

}
```

改代码运行的结果是多少呢？答案是`15027`（每次运行不太一样）。因为上述代码反了一个错误，synchronized 修饰的方法是普通的成员方法，因此锁是实例对象锁，但是这边 new 了两个实例，因此他们不会产生竞争锁的关系，因此 static 修饰的共享资源 i 会小于20000。对于这种情况，我们要将 synchronized 修饰静态方法。代码如下：

```java
	...
	private static synchronized void increase() {
		i++;
	}
	...
```

此时无论多少次，输出都是 20000。因为这时的锁变成了类对象锁，而类对象不属于任何一个实例。
要注意的是，当一个类中同时有 synchronized 修饰的普通成员方法和静态方法时，这两者之间不会产生竞争关系（因为锁不同）。

**synchronized 作用于代码块**

如果光是上面两种作用范围，相比我们编程的时候会发生诸多不便，比如说如果方法体特别大，然而涉及到同步的操作只有一小部分，又懒得去额外写一个方法，这时候用上面两种方式就不太合适。Java 给我们提供了一种更加灵活地方式，同步代码块。同步代码块需要我们自己指定锁，锁可以是一个普通成员对象，也可以是静态成员对象，也可以是this，甚至是class!代码如下：

```java
// 普通成员对象作为锁
public class Test implements Runnable {
	
	final String LOCK = "LOCK";
	
	public Integer i = 0;
	
	private void increase() {
		i++;
	}
	
	@Override
	public void run() {
		// TODO Auto-generated method stub
		for(int i=0;i<10000;i++) {
			synchronized (LOCK) {
				increase();
			}
		}
		
	}

	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		Test test = new Test();
		Thread t1 = new Thread(test);
		Thread t2 = new Thread(test);
		t1.start();
		t2.start();
		t1.join();
		t2.join();
		System.out.println(test.i);
	}

}
```

更神奇的是，你完全可以根据需要将锁对象进行修改！比如说下面：

```java
// 静态对象作为锁，这就跟同步静态方法差不多了
public class Test implements Runnable {
	
	static final String LOCK = "LOCK";
	
	public static Integer i = 0;
	
	private void increase() {
		i++;
	}
	
	@Override
	public void run() {
		// TODO Auto-generated method stub
		for(int i=0;i<10000;i++) {
			synchronized (LOCK) {
				increase();
			}
		}
		
	}

	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
//		Test test = new Test();
		Thread t1 = new Thread(new Test());
		Thread t2 = new Thread(new Test());
		t1.start();
		t2.start();
		t1.join();
		t2.join();
		System.out.println(i);
	}
}

```

```java
// 实例作为锁，就是实例对象锁，只是这边显示表达了，跟 synchronized 修饰成员方法原理一样
public class Test implements Runnable {
	
//	static final String LOCK = "LOCK";
	
	public Integer i = 0;
	
	private void increase() {
		i++;
	}
	
	@Override
	public void run() {
		// TODO Auto-generated method stub
		for(int i=0;i<10000;i++) {
			synchronized (this) {
				increase();
			}
		}
		
	}

	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		Test test = new Test();
		Thread t1 = new Thread(test);
		Thread t2 = new Thread(test);
		t1.start();
		t2.start();
		t1.join();
		t2.join();
		System.out.println(test.i);
	}
}

```

```java
// class作为锁，其实就是类对象锁，跟 synchronized 修饰静态方法意思差不大
public class Test implements Runnable {
	
//	static final String LOCK = "LOCK";
	
	public static Integer i = 0;
	
	private void increase() {
		i++;
	}
	
	@Override
	public void run() {
		// TODO Auto-generated method stub
		for(int i=0;i<10000;i++) {
			synchronized (Test.class) {
				increase();
			}
		}
		
	}

	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
//		Test test = new Test();
		Thread t1 = new Thread(new Test());
		Thread t2 = new Thread(new Test());
		t1.start();
		t2.start();
		t1.join();
		t2.join();
		System.out.println(i);
	}
}

```

**synchronized 原理**

还没整理，可以看[大佬博客](https://blog.csdn.net/javazejian/article/details/72828483#synchronized%E7%9A%84%E4%B8%89%E7%A7%8D%E5%BA%94%E7%94%A8%E6%96%B9%E5%BC%8F)

**<a id="volatile">volatile</a>**
并发编程的三个概念：原子性、可见性、有序性。
volatile的特性：
1、保证可见性，但不保证原子性
volatile 修饰变量时，JMM 会把该线程本地内存中的变量强制刷新到主内存中，并使其他线程中的缓存无效。但不能保证多线程下结果的正确。
2、禁止指令重排
重排序的规则是不会对有数据依赖的操作进行重排，运行结果不会发生变化。volatile 关键字起到了内存屏障的作用，保证了 volatile 前后操作隔离，重排序不会串。



### <a id="characteristic">四、三大特性</a>

#### <a id="encapsulation">封装</a>

说到封装，基本上是个程序员都用过。我们在写一个方法的时候，不会将一个方法体写的特别长，
而是选择将方法拆解成好几个方法，将不同逻辑的方法进行封装。Java 的封装比一般的方法封装更加系统化一些。

概念：将类的成员变量隐藏在类的内部不允许外部程序直接访问，仅可通过该类提供的方法来实现对隐藏信息的操作和访问。
封装可以被认为是一个保护屏障，防止该类的代码和数据被外部类定义的代码随机访问。要访问该类的代码和数据，必须通过严格的接口控制。封装最主要的功能在于我们能修改自己的实现代码，而不用修改那些调用我们代码的程序片段。适当的封装可以让代码更容易理解和维护，也加强了代码的安全性。

优点：
* 良好的封装能够减少耦合；
* 类内部的结构可以自由修改；
* 可以对成员变量进行更精确的控制；
* 隐藏信息，实现细节

措施：
**<a id="keyword_access">1、访问修饰符</a>**

访问修饰符|本类|本包|子类|其他
:--:|:--:|:--:|:--:|
private|Y|N|N|N
默认|Y|Y|N|N
protected|Y|Y|Y|N
public|Y|Y|Y|Y

从 private 到 public ，封装性越来越差

**<a id="keyword_this">2、this 关键字</a>**

使用 this 关键字是为了区别类提供的公共方法中对于函数参数和成员参数名的区别，如
```
	public void setName(String name){
		this.name = name;
	}
```

**<a id="innerclass">3、Java 中的内部类</a>**

Java 内部类提供了更好的封装，可以将内部类隐藏在外部类之内，不允许同一个包中的其他类直接访问内部类。

根据内部类的实现方式，我们将内部类分为：成员内部类、静态内部类、方法内部类以及匿名内部类。内部类和接口的配合，很好地实现了多继承的效果。

**成员内部类**

成员内部类是最常见的内部类，代码如下：

```java
public class OutterClass {
	public String outterName = "OutterClass";
	
	private String outterSecret = "outterSecret";
	
	public static String method = "test";
	
	public Integer age = 2;
	
	public void test() {
//		外部类无法直接调用内部类的属性和方法
//		System.out.println(innerName);
	}
	
	public class InnerClass{
		
		public String innerName = "InnerClass";
		
		private String innerSecret = "innersecret";
		
		public Integer age = 1;
		
		public void test() {
			// 可以访问外部类的成员属性，public、private 修饰的都可以
			System.out.println(outterName);
			System.out.println(outterSecret);
			
			// 内部类和外部类存在相同名称属性的情况
			System.out.println(age);
			System.out.println(this.age);
			System.out.println(OutterClass.this.age);
		}
	}
	
	public static void main(String[] args) {
		OutterClass outterClass = new OutterClass();
//		通过实例化内部类的方式调用内部类方法，成员内部类没有 static 属性和方法
		InnerClass innerClass = outterClass.new InnerClass();
		innerClass.test();
	}
}
```

从上面代码可以看到成员内部类的使用方法和注意事项：
* 成员内部类如同外部类的成员一样定义
* 成员内部类可以调用外部类的属性和方法，即便是 private 修饰的变量
* 外部类无法直接调用成员内部类的属性和方法，只能通过实例化成员内部类的方式调用，实例化的方式为`内部类 对象名 = 外部类实例.new 内部类()`
* 成员内部类和外部类有相同属性的情况下，内部类方法默认调用内部类属性，使用`外部类名.this.属性`的方式调用外部类属性
* 在编译外部类时，我们会发现产生了两个.class文件

**静态内部类**

静态内部类是 static 修饰的内部类，代码如下：

```java
public class OutterClass {
	public String outterName = "OutterClass";
	
	private String outterSecret = "outterSecret";
	
	public static String method = "test";
	
	public Integer age = 2;
	
	public void test() {
//		外部类无法直接调用内部类的属性和方法
//		System.out.println(innerName);
	}
	
	public static class InnerClass{
		
		public String innerName = "InnerClass";
		
		private String innerSecret = "innersecret";
		
		public static String method = "test2";
		
		public Integer age = 1;
		
		public void test() {
			// 无法直接访问外部类非静态的成员的方法，但可以通过实例化外部类的方式访问
//			System.out.println(outterName);
//			System.out.println(outterSecret);
			System.out.println((new OutterClass()).outterName);
			
			// 内部类和外部类存在相同名称属性的情况
			System.out.println(method);
			System.out.println(this.method);
			System.out.println(OutterClass.method);
		}
	}
	
	public static void main(String[] args) {
//		OutterClass outterClass = new OutterClass();
//		访问静态内部类时，不需要实例化外部类，也不用使用成员内部类的方式调用，直接 new 即可
		InnerClass innerClass = new InnerClass();
		innerClass.test();
	}
}
```

从上面我们可以总结出静态内部类的使用方法和注意事项：
* 静态内部类无法直接访问外部类的非静态成员属性和方法，但是可以通过实例化外部类的方式访问
* 内部类和外部类存在相同名称属性的情况下，默认调用内部类的属性
* 使用静态内部类时不需要通过实例化外部类的方式，直接 new 即可

**方法内部类**

方法内部类只存在于外部类的方法中，仅在该方法内部可以使用，代码如下：

```java
public class OutterClass {
	public String outterName = "OutterClass";
	
	private String outterSecret = "outterSecret";
	
	public static String method = "test";
	
	public Integer age = 2;
	
	public void test() {
//		外部类无法直接调用内部类的属性和方法
//		System.out.println(innerName);
		
		class InnerClass{
			
			public String innerName = "InnerClass";
			
			private String innerSecret = "innersecret";
					
			public Integer age = 1;
			
			public void test() {
				// 可以直接访问外部类的属性
				System.out.println(outterName);
				System.out.println(outterSecret);
				
				// 内部类和外部类存在相同名称属性的情况
				System.out.println(age);
				System.out.println(this.age);
				System.out.println(OutterClass.this.age);
			}
		}
		
//		实例化和调用内部类
		InnerClass innerClass = new InnerClass();
		innerClass.test();
	}
}
```

方法内部类不能使用访问修饰符进行修饰，因为他只能在方法内部进行使用

**匿名内部类**

匿名内部类是我们开发的时候使用的最多的一种，也是可以讲的最多的一种。他适合那种只创建一次的使用场景。
首先，我们来看两段实现的代码：

```java
// 基于抽象类的匿名内部类实现方式
abstract class Animal{
	public abstract void bark();
}

public class Test{

	public static void main(String[] args) throws InterruptedException {
		// 这个不叫匿名内部类，因为有 animal 这个名字
		Animal animal = new Animal() {
			
			@Override
			public void bark() {
				// TODO Auto-generated method stub
				System.out.println("bark");
			}
		};
		animal.bark();
		
		// 这才是匿名内部类，符合 new + Class + body 的格式  
		// 我们常用 Thread -> start() 就是这种写法
		new Animal() {
			
			@Override
			public void bark() {
				// TODO Auto-generated method stub
				System.out.println("bark bark");
			}
		}.bark();;
	}
}

```

```java
// 基于接口的匿名内部类实现方式
interface Animal{
	public void bark();
}

public class Test{

	public static void main(String[] args){
		makeBarks(new Animal() {
			
			@Override
			public void bark() {
				// TODO Auto-generated method stub
				System.out.println("bark");
			}
		});
	}
	
	public static void makeBarks(Animal animal) {
		animal.bark();
	}
}

```

从上面代码我们总结出下面几点：
* 使用匿名内部类时，我们需要遵循两种写法：
	* new + Class + {} + method
	* invoke( new + Class + {})
* 匿名内部类中不能定义构造函数(初始化工作使用构造代码块完成)；
* 匿名内部类中不能存在任何静态成员变量和静态方法；
* 匿名内部类是方法内部类的延伸，所以方法内部类的所有限制同样生效；
* 匿名内部类中必须实现继承的类或者实现的接口的所有抽象方法；
* 内部类中使用的形参必须为 final 。

原因（[参考博客](https://blog.51cto.com/android/384844)）

>同时在这个例子，留意外部类的方法的形参，当所在的方法的形参需要被内部类里面使用时，该形参必须为final。这里可以看到形参name已经定义为final了，而形参city 没有被使用则不用定义为final。为什么要定义为final呢？在网上找到本人比较如同的解释：
 “这是一个编译器设计的问题，如果你了解java的编译原理的话很容易理解。  
首先，内部类被编译的时候会生成一个单独的内部类的.class文件，这个文件并不与外部类在同一class文件中。  
当外部类传的参数被内部类调用时，从java程序的角度来看是直接的调用例如：  
public void dosome(final String a,final int b){  
  class Dosome{public void dosome(){System.out.println(a+b)}};  
  Dosome some=new Dosome();  
  some.dosome();  
}  
从代码来看好像是那个内部类直接调用的a参数和b参数，但是实际上不是，在java编译器编译以后实际的操作代码是  
class Outer$Dosome{  
  public Dosome(final String a,final int b){  
  this.Dosome$a=a;  
  this.Dosome$b=b;  
}  
  public void dosome(){  
  System.out.println(this.Dosome$a+this.Dosome$b);  
}  
}}  
从以上代码看来，内部类并不是直接调用方法传进来的参数，而是内部类将传进来的参数通过自己的构造器备份到了自己的内部，自己内部的方法调用的实际是自己的属性而不是外部类方法的参数。  
这样理解就很容易得出为什么要用final了，因为两者从外表看起来是同一个东西，实际上却不是这样，如果内部类改掉了这些参数的值也不可能影响到原参数，然而这样却失去了参数的一致性，因为从编程人员的角度来看他们是同一个东西，如果编程人员在程序设计的时候在内部类中改掉参数的值，但是外部调用的时候又发现值其实没有被改掉，这就让人非常的难以理解和接受，为了避免这种尴尬的问题存在，所以编译器设计人员把内部类能够使用的参数设定为必须是final来规避这种莫名其妙错误的存在。”
 (简单理解就是，拷贝引用，为了避免引用值发生改变，例如被外部类的方法修改等，而导致内部类得到的值不一致，于是用final来让该引用不可改变)



#### <a id="inheritance">继承</a>

继承是java面向对象编程技术的一块基石，因为它允许创建分等级层次的类。

继承就是子类继承父类的特征和行为，使得子类对象（实例）具有父类的实例域和方法，或子类从父类继承方法，使得子类具有父类相同的行为。因此继承需要符合的关系是：is-a，父类更加通用，子类更具体。

那么为什么需要继承呢？主要目的就是解决代码的冗余性，实现代码复用。

语法规则：`class 子类 extends 父类`

**<a id="inheritance_charactertistic">继承的特性</a>**

1、我们知道 Java 不支持多继承，但是支持多重继承。
![继承类型]()
2、子类拥有父类非 private 的属性、方法；
3、子类可以拥有自己的属性和方法，即子类可以对父类进行扩展；
4、子类可以用自己的方式实现父类的方法（override，重写不是重载，和overload不同），调用时优先调用子类中的该方法

**<a id="inheritance_override">方法的重写</a>**

在继承中，子类会自动继承父类的属性和方法。但是如果子类需要对父类的方法修改，以实现不同的业务逻辑的时候，这时候就需要方法的重写（override）。**当调用方法时，会优先调用子类的方法。**如下面的代码中：

```java
abstract class Animal{
	abstract void bark();
}

class Person extends Animal{

	@Override
	void bark() {
		// TODO Auto-generated method stub
		System.out.println("Say hello !");
	}
	
}

class Man extends Animal{

	@Override
	void bark() {
		// TODO Auto-generated method stub
		System.out.println("Say hello man!");
	}
	
}

class Woman extends Animal{

	@Override
	void bark() {
		// TODO Auto-generated method stub
		System.out.println("Say hello woman!");
	}
	
}

class Dog extends Animal{

	@Override
	void bark() {
		// TODO Auto-generated method stub
		System.out.println("WAH WAH !");
	}
	
}
```

重写时需要注意，方法的返回值、方法名、参数类型及个数必须和父类相同，才叫方法的重写。By the way，有些人说必须是方法签名相同，这边普及一下方法签名是什么。Effective Java 书中说的是方法签名由两部分组成：**方法的名称**和**参数类型**。Java 中不允许出现方法签名一样的方法，因此，下面这段代码是无法通过编译的：

```java
public int A(){}

public long A(){}
```

有些人在说 override 的时候喜欢说是方法的重载，在中文里面感觉差不多，但是翻译过来还是不一样的。重载是(overload)，重写是(override)。重写时在同一个类中实现方法名相同，参数不同的方法，两个方法的签名是不同的。重写是实现子类中对父类存在的方法进行修改的方法，子类和父类的方法签名相同。

**<a id="inheritance_initial">初始化</a>**

这个我们在关键字 static 中讲过。
存在继承的情况下，初始化顺序为：
* 父类（静态变量、静态语句块）
* 子类（静态变量、静态语句块）
* 父类（实例变量、普通语句块）
* 父类（构造函数）
* 子类（实例变量、普通语句块）
* 子类（构造函数）

**<a id="inheritance_key_words">继承相关关键字</a>**

1、extends
这个关键字用于声明类的继承

2、fianl
使用 final 进行修饰，有一种不可变的意思。
* 修饰类，则表示这个类**不能被继承**
* 修饰方法，则表示这个方法**不能被重写**
* 修饰属性，则表示该属性只能够被赋值一次，一旦被赋值就不可以再被改变（对基本类型来说，是值不能被改变，对引用类型来说是引用对象不能改变）。属性的值初始化可以在两个地方：一是在属性定义时直接赋值，二是在构造函数中

3、super 和 this
super 用于在子类中使用父类的方法和属性：
* super.property 访问父类的属性
* super.method 访问父类的方法
* super() 使用父类的构造方法（PS：在子类的构造方法中，我们需要调用父类的构造方法。如果我们没有显式调用父类的构造方法，那么系统默认调用父类无参构造方法。如果父类没有无参构造方法，那么编译会出错）
this 用于类中表示这个实例的意思。

**<a id="inheritance_object">Object 类</a>**

Object 类是所有类的父类，如果一个类没有明确表示集成另一个类，那么这个类默认继承 Object 类。因此，可以将 Object 类视为所有类的父类，Object 类中的方法在所有类中都能使用。

下面来看看 Object 类中的方法：
1、native 方法 registerNatives()
作用是在加载的动态文件中定位并连接 native 方法。该方法在代码块中，在类加载时就已经被执行。
2、Object()
无参构造方法
3、native 方法 getClass()
获取类信息
4、native 方法 clone()
返回当前实例的一个副本
5、equals()
返回两个对象地址的比较。重写 equals 方法以进行属性的比较。
6、finalize()
销毁对象，由GC调用
7、hashCode()
返回 hash 值
8、notify()
唤醒一个以当前实例为锁的线程
9、唤醒所有线程
唤醒所有以当前实例为锁的线程
10、toString()
返回类名 + hash 码。对其进行重写以返回实例的属性值
11、wait()、wait(long,int)、wait(long)
当前线程进入休眠直到被其他线程唤醒或者直到一定时间之后



#### <a id="polymorphism">多态</a>

多态是同一个行为具有多种不同表现形式或者形态的能力，或者对象的多种形态的能力。
Java 中的多态主要表现在两个方面：引用的多态、方法的多态

**<a id="polymorphism_reference">引用多态</a>**

引用多态其实就是一句话：父类的引用可以指向父类的对象，父类的引用可以指向子类的对象。
比如下面一段代码：

```java
class Person{

	void bark() {
		// TODO Auto-generated method stub
		System.out.println("Say hello !");
	}
	
}

class Man extends Person{

	void bark() {
		// TODO Auto-generated method stub
		System.out.println("Say hello man!");
	}
	
}

class Woman extends Person{

	void bark() {
		// TODO Auto-generated method stub
		System.out.println("Say hello woman!");
	}
	
}


public class Test{
	public static void main(String[] args) {
		//父类引用可以指向父类对象，父类引用也可以指向子类对象
		Person person = new Person();
		Person man = new Person();
		Person woman = new Person();
		invoke(person);
		invoke(man);
		invoke(woman);
		// 子类对象可以放到父类形参的方法中
		Man newMan = new Man();
		Woman newWoman = new Woman();
		invoke(newMan);
		invoke(newWoman);
	}
	
	public static void invoke(Person person) {
		person.bark();
	}
}
```

**<a id="polymorphism_method">方法多态</a>**

创建父类对象时，调用的方法是父类的方法，调用子类对象时，调用的方法是子类的方法。
就如上面的代码中:

```java
	//父类引用可以指向父类对象，父类引用也可以指向子类对象
	Person person = new Person();
	Person man = new Person();
	Person woman = new Person();
	invoke(person);
	invoke(man);
	invoke(woman);
	// 子类对象可以放到父类形参的方法中
	Man newMan = new Man();
	Woman newWoman = new Woman();
	invoke(newMan);
	invoke(newWoman);
```

输出结果为：

```java
Say hello !
Say hello !
Say hello !
Say hello man!
Say hello woman!
```

**<a id="polymorphism_reference_transfer">引用类型转换</a>**

引用类型转换分为：向上转换和向下转换。

**向上转换**

向上转换是子类到父类的转换，如之前的代码
```
	Person person = new Person();
	Person man = new Person();
	Person woman = new Person();
	invoke(person);
	invoke(man);
	invoke(woman);
```
这种转化没有风险。

**向下转换**

向下转换是从父类到子类的转换，如下面的代码：

```java
	Person man = new Person();
	Person woman = new Person();		
	Man newMan = (Man)man;
	Woman newWoman = (Woman)woman;
```

这样的转换需要在对象面前加上 (目标Class) 进行强制类型转换，因此有数据溢出的风险。
但是，下面的代码虽然编译能通过，但是运行时会报错:

```java
Man man = (Man)new Person();
Woman woman = (Woman)new Person();
```

**instanceof 和 getClass 的使用**

为了避免类型转换的安全问题，这里可以使用 instanceof 和 getClass 的方法进行类型判断。但是 instanceof 是指某个对象是一个xx类的实例，比如说`man instanceof Person`是可以的，但是 getClass 不考虑继承关系，`man.getClass == Person.class`就不行了。

**<a id="polymorphism_absrtact">抽象类</a>**

抽象类是被 abstract 关键字修饰的类，它有以下特点：

1、抽象类不能被实例化
抽象类不能被直接创建，但是可以定义引用变量来指向子类对象。代码如下：

```java
abstract class Person{
	abstract void bark();
}

class Man extends Person{

	void bark() {
		// TODO Auto-generated method stub
		System.out.println("Say hello man!");
	}
	
}

class Woman extends Person{

	void bark() {
		// TODO Auto-generated method stub
		System.out.println("Say hello woman!");
	}
	
}


public class Test{
	public static void main(String[] args) {
		// Person person = new Person();    // 不能被实例化
		Person man = new Man();
		Person woman = new Woman();		
	}
}
```

2、抽象类只约定了子类必须有什么方法，但不关心子类如何实现这些方法。抽象类中可以包含普通的方法，也可以有抽象方法。
3、抽象类中如果有抽象方法，只有声明，没有实现。子类中必须实现对抽象方法的重写，否则会报错。

**<a id="polymorphism_interface">接口</a>**

1、概念
接口可以理解为一种特殊的类，由**全局常量**和公共的**抽象方法**所组成。也可理解为一个特殊的抽象类，因为它含有抽象方法。
如果说类是一种具体实现体，而接口定义了某一批类所需要遵守的规范，接口不关心这些类的内部数据，也不关心这些类里方法的实现细节，它只规定这些类里必须提供的某些方法。（这里与抽象类相似）

2、基本语法
```java
[修饰符] [abstract] interface 接口名 [extends父接口1,2....]（多继承）{
	0…n常量 (public static final)                                          
	0…n 抽象方法(public abstract)   
}
```
其中[]里的内容表示可选项，可以写也可以不写;接口中的属性都是常量，即使定义时不添加 public static final 修饰符，系统也会自动加上；接口中的方法都是抽象方法，即使定义时不添加 public abstract 修饰符，系统也会自动加上。

3、使用接口
一个类可以实现一个或多个接口，实现接口使用 implements 关键字。java 中一个类只能继承一个父类，是不够灵活的，通过实现多个接口可以补充。语法为：
```
[修饰符] class 类名 extends 父类 implements 接口1，接口2...{
	类体部分    //如果继承了抽象类，需要实现继承的抽象方法；要实现接口中的抽象方法
}
```
*注意：如果要继承父类，继承父类必须在实现接口之前,即 extends 关键字必须在 implements 关键字前*

**<a id="polymorphism_difference">接口和抽象类的区别</a>**

首先，这两者的语法区别如下：

参数|抽象类|接口
---|---|---
声明|抽象类使用 abstract 关键字声明|接口使用 interface 关键字声明
实现|子类使用 extends 关键字来继承抽象类。如果子类不是抽象类的话，他需要提供抽象类中所有声明的方法的实现|子类使用 implements关键字来实现接口。它需要提供接口中所有声明的方法的实现
构造器|抽象类可以有构造器|接口没有构造器
访问修饰符|抽象类中的方法可以是任意访问修饰符|接口方法默认修饰符是 public，并且不允许定义为 private 或者 protected
多继承（实现）|一个类最多只能继承一个抽象类|一个类可以实现多个接口
字段声明|抽象类的字段声明可以是任意的|接口的字段默认都是 static 和 final
设计|抽象类是模板类设计|接口是契约式设计
作用|抽象类被继承时体现的是 is-a 关系|接口被实现时体现的是 can-do 关系

接口和抽象类各有优缺点，在接口和抽象类的选择上，必须遵守这样的原则：
* 行为模型应该总是通过接口而不是抽象类定义，所以通常是优先选用接口，尽量少用抽象类；
* 选择抽象类的时候通常是如下情况：需要定义子类的行为，又要为子类提供通用的功能

备注：Java 8 中接口引入默认方法和静态方法，以此来减少抽象类和接口之间的差异。现在，我们可以为接口提供默认实现的方法了，并且不用强制子类来实现它。



### <a id="throwable">五、异常</a>

#### <a id="throwable_abstract">异常机制的概述</a>

异常机制是指当程序出现错误后，程序如何处理。具体来说，异常机制提供了程序退出的安全通道。当出现错误后，程序执行的流程发生改变，程序的控制权转移到异常处理器。

程序错误分为三种：1、编译错误；2、运行时错误；3、逻辑错误。
(1)编译错误是因为程序没有遵循语法规则，编译程序能够自己发现并且提示我们错误的原因和位置；
(2)运行时错误是因为程序在执行时，运行环境发现了不能执行的操作；
(3)逻辑错误是因为程序没有按照预期的逻辑顺序执行，异常也就是指程序运行时发生错误，而异常处理就是对这些错误进行处理和控制。



#### <a id="throwable_structure">异常的结构</a>

Java 中所有的异常都有一个共同的祖先 Throwable。Throwable 允许代码可以通过异常传播机制将代码中的异常进行抛出。
下面是 JDK 中异常的结构（图来自网络，侵删）：

![异常结构](https://uploader.shimo.im/f/3Dl5C1lDOHqXPLYm.png!thumbnail)

从上图的结构中可以看出来，Throwable 有两个重要的子类：Exception（异常）和 Error（错误），二者都是 Java 异常处理的重要的类。Throwable 是 Exception 和 Error 的父类，而 Exception 和 Error 的区别就在于 Exception 能被程序本身处理（try...catch...）,而 Error 是无法处理的（比如 StackOverflowError 和 OutOfMemoryError）。

**<a id="throwable_throwable">Throwable</a>**

Throwable 是所有异常的父类，实现了 Serializable 接口。他有以下重要的属性和方法：

1、detailMessage（String）
描述当前 Throwable 的具体信息。在构造函数中赋值，在 toString()、getMessage()方法中都有用到

2、stackTrace（StackTraceElement[]）
用于记录栈帧的数组，底层是用 native 方法 fillInStackTrace(int) 实现。

3、cause（Throwable）
可以看做原因，记录导致当前 Throwable 产生的那个 Throwable。没有的话就是本身

4、toString()
返回异常发生时的简要概述

5、getMessage()
返回 detailMessage

6、getLocalizedMessage()
返回异常对象的本地化信息。Throwable 实现的就是 getMessage() 方法。子类可以重写他。

7、printStackTrace()
控制台打印 stackTrace 中的栈帧信息。其中有控制台异常打印的锁的竞争。

8、readObject 和 writeObject
序列化和反序列化实现的方法

**<a id="throwable_error">Error</a>**

Error 是程序中出现无法处理的情况导致的，表示运行程序中出现了严重的错误问题。Error 或者 Error 的子类，都代表这种异常不应该被捕获，如果遇到应该被捕获的异常，那么应该改用 Exception。网上说 Error 都是因为虚拟机引起的，倒也不全然是这样。Error 的子类众多，引起的情况也五花八门，但是相同点都是认为 Error 及其子类都应该是意料之外的异常产生的，是**非受查异常**，因此 Error 及其子类有且仅有构造方法。下面列举几个常见的Error：

1、AnnotationFormatError
从类文件中读取注解时报错产生的，通常是因为反射读取注解文件产生的

2、IOError
和 IOException 有区别，当有 IO 方面的错误发生时生成

3、ReflectionError
是 sun.nio.ch.Reflect 的内部类，不开源

4、VirtualMachineError
比较有名的是它的两个子类，OutOfMemoryError 和 StackOverFlowError

5、OutOfMemoryError
当堆空间无法分配内存的情况下出现该错误，原因是创建对象过多

6、StackOverFlowError
栈深达到上限时抛出错误，原因是记录的方法栈帧太多，超出栈的最大深度（常出现在递归中）

**<a id="throwable_exception">Exception</a>**

Exception 是程序本身可以处理的异常。Exception 可以简单分为 RuntimeException 和其他 Exception。前者是非受查异常，后者是受查异常。列举几个常见的Exception：

1、XMLParseException
非受查异常，XML 解析时产生

2、IndexOutOfBoundsException
非受查异常，下标越界时产生

3、ReflectionException
受查异常，MBeans 中使用反射时可能产生

4、SQLException
受查异常，和数据库交互时可能产生



#### <a id="throwable_questions">异常常见的问题</a>

Q1、什么是受查异常和非受查异常？
A：受查异常就是在代码中必须需要显式try...catch...的异常，非受查异常就是不需要在代码中显示捕获也能运行的异常。除了 Error 及其子类、RuntimeException 及其子类是非受查异常外，其他的都是受查异常。

Q2：在开发过程中，是应该抛出异常还是捕获异常？
A：遇到知道如何处理的异常时进行捕获并处理，遇到不知道如何处理时则抛出。



### <a id="relflection">六、反射</a>

Java 的反射机制是在运行过程中，对于任何一个类，都能够知道这个类的所有属性和方法；对于任何一个对象，都能够调用他的任何一个方法和属性（这种说法不正确，我之前调用 private 方法时就报错。必须要在前面设置一下权限才能使用）。这种运行时动态获取信息以及动态调用对象的功能称为 java 语言的反射机制。



#### <a id="relflection_usage">反射的主要用途</a>

**反射最重要的用途就是开发框架。**
在我们日常开发工作中，难免会用到很多框架，比如说Spring、Mybatis，这些框架除了用注解之外，还需要我们使用 XML 文件对 Bean 等进行配置。这实际上就是对反射的使用，在 XML 文件中描述好一个 Bean 之后，项目启动时会有拦截器拦截并动态的去创建这些 Bean，这个过程中就需要 XML 中描述好的类信息，然后通过反射区创建甚至是调用一些方法。

其次，**我们日常开发中使用IDE，打出类名会自动跳出他的属性和方法**，这个过程其实也是对反射的一种使用。

其他时候，我们虽然不怎么用反射，但是反射作为一种思想，用在了很多框架的 IOC 模块，因此，想要深入理解这些框架，反射是必须要了解的。同时，了解反射的知识也可以丰富自己的编程思想，很有必要。



#### <a id="relflection_how">反射的基本使用</a>

反射可以通过获取一个类的 Class 来实现这个类的实例化、属性调用、方法调用，与反射有关的类基本上都在 java.lang.relfect 的包里。这边我们来讲一下反射的基本使用：

**<a id="relflection_how_class">1、获得 Class 对象</a>**

我们使用反射时，通常都需要先获得一个类的 Class 对象，然后根据 Class 对象来操作，可以进行实例化，可以调用静态方法和属性，也可以实例化后调用普通成员方法和属性。获得 Class 对象的方法有三种：

(1) 使用 Class 类的`forName`方法：
使用方法如下代码:

```java
// 被加载的类
public class MyClass {

	public static String str1 = "str1";
	static {
		System.out.println("static code block...");
		System.out.println(str1);
	}
	
	public MyClass() {
		System.out.println("construct method...");
	}
}
// 获取 Class 对象
public class Test{
	public static void main(String[] args) throws ClassNotFoundException {
		Class myClass = Class.forName("MyClass");
	}
}
```

运行后，控制台输出为：

```java
static code block...
str1
```

由此可以看出，Class.forName() 方法成功的将 .class 文件加载到了虚拟机中（即类加载过程），但是没有创建实例，所以只会进行静态成员的赋值和静态代码块的运行。
我们来看 forName 方法的源码：

```java
public static Class<?> forName(String className)
                throws ClassNotFoundException {
        Class<?> caller = Reflection.getCallerClass();
        return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
    }

private static native Class<?> forName0(String name, boolean initialize,
                                            ClassLoader loader,
                                            Class<?> caller)
        throws ClassNotFoundException;
```

所以可以看到最后还是使用的 native 方法。需要注意的是这边有 ClassLoader 类的参与，这又涉及到类加载器和双亲委派这些知识点了。

(2)直接获取某个实例的 class：

```java
public class Test{
	public static void main(String[] args) throws ClassNotFoundException {
		Class myClass = MyClass.class;
	}
}
```

此时，控制台输出为空！！！可以看到，这种方法是不会将类进行加载的。。。这就有点难理解了，我们反编译一下：

```java
...
 #16 = Utf8               Exceptions
  #17 = Class              #18            // java/lang/ClassNotFoundException
  #18 = Utf8               java/lang/ClassNotFoundException
  #19 = Class              #20            // java/lang/InstantiationException
  #20 = Utf8               java/lang/InstantiationException
  #21 = Class              #22            // java/lang/IllegalAccessException
  #22 = Utf8               java/lang/IllegalAccessException
  #23 = Class              #24            // MyClass
  #24 = Utf8               MyClass
  #25 = Utf8               args
  #26 = Utf8               [Ljava/lang/String;
  #27 = Utf8               myClass
...
...
public static void main(java.lang.String[]) throws java.lang.ClassNotFoundException, java.lang.InstantiationException, java.lang.IllegalAccessException;
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Exceptions:
      throws java.lang.ClassNotFoundException, java.lang.InstantiationException, java.lang.IllegalAccessException
    Code:
      stack=1, locals=2, args_size=1
         0: ldc           #23                 // class MyClass
         2: astore_1
         3: return
      LineNumberTable:
        line 4: 0
        line 5: 3
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       4     0  args   [Ljava/lang/String;
            3       1     1 myClass   Ljava/lang/Class;
```

可以看到，其实 .class 是在编译完成的时候就完成了值的替换。我们知道，类加载是在类首次真正使用的时候完成的，在`MyClass myClass = null`这种情况是不会进行类加载的。为什么呢？我个人的理解是，类加载实际上是为了在类进行使用前，将类的信息加载 JVM 所进行的操作，这时候需要开辟内存、静态变量的赋值和静态代码块的运行，这样才能确定在常量池中的空间占用。上面这种写法，在编译后发现实际上并不需要用到 MyClass 这个类，因此 JVM 就不会把它加载进来，因此不会执行静态代码块。
这么说来，反倒是 Class.forName() 把类直接加载了倒有点奇怪。不过看到参数里面有 ClassLoader 也应该想到了会直接加载把。

(3)对某个实例调用其 getClass() 方法：

```java
public static void main(String[] args){
		MyClass myClass = new MyClass();
		Class my_class = myClass.getClass();
}
```

这段代码都 new 了一个实例了，因此输出肯定是静态代码块的运行结果和构造函数的结果，不提了。


**<a id="relflection_how_isinstance">2、判断一个对象是不是某个类的实例</a>**

通常我们为了判断一个对象是不是某个类的实例，用到了以下三种方法：
(1) obj.getClass() == Class？
(2) instanceof 关键字
(3) Class.isInstance(obj)

如果仅仅是判断一个实例和他本身所属类的比较，那也没什么意思。这边我们来看和其父类的比较。

为了测试，我们创建了一下几个类：

```java
public class MyClass {
	public MyClass(){
		
	}
	
	public MyClass(String name) {
		
	}
}

public class MySubClass extends MyClass{
	public MySubClass() {
		
	}
	
	public MySubClass(String name) {
		
	}
}

public class Test{
	public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
		MySubClass mySubClass = new MySubClass();
		Class myClass = MyClass.class;
		System.out.println(mySubClass.getClass() == myClass);
		System.out.println(mySubClass instanceof MyClass);
		// isInstance 源码是 native 方法
		System.out.println(myClass.isInstance(mySubClass));
	}
}
```

输出结果是

```java
false
true
true
```

可以看出来，后面两种方法是可以判断继承的情况，而第一个不行。

**<a id="relflection_how_newinstance">3、实例的创建</a>**

通过反射来创建实例有两种方法：

(1)使用 Class 对象的 newInstance() 方法来创建：

```java
	Class myClass = MyClass.class;
	MyClass my_Class = (MyClass) myClass.newInstance();
```

(2)使用 Class 对象获取指定的Constructor 对象，再调用 Constructor 对象的 newInstance() 方法来创建实例（这种方法可以自己选择构造函数）。

```java
public class MyClass {
	public String name;
	
	public MyClass(){
		
	}
	
	public MyClass(String name) {
		this.name = name;
	}
}

public class Test{
	public static void main(String[] args) throws NoSuchMethodException, SecurityException, InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException {
		// 获取类对象
		Class myclass = MyClass.class;
		// 获取对应构造函数的构造器
		Constructor constructor1 = myclass.getConstructor();
		Constructor constructor2 = myclass.getConstructor(String.class);
		// 使用构造器进行实例化
		MyClass obj1 = (MyClass) constructor1.newInstance();
		MyClass obj2 = (MyClass) constructor2.newInstance("name");
		// 结果查看
		System.out.println(obj1.name);    // null
		System.out.println(obj2.name);    // name
	}
}
```
**<a id="relflection_how_fields&methods">4、获取成员和使用</a>**

创建如下类文件：

```java
public class MyClass {
	public String name = "lewis";
	protected String age = "24";
	String height = "175cm";
	private String weight = "55kg";
	
	public String getName() {
		return this.name;
	}
	
	protected String getWeight() {
		return this.weight;
	}
	
	String getHeight() {
		return this.height;
	}
	
	private String getAge() {
		return this.age;
	}
}
```

获取某个 Class 对象的成员变量，有以下几个方法：

(1)`Field[] declaredFields = myClass.getDeclaredFields();`
该方法返回类或者接口声明的所有成员变量，包括 public, protected, 默认, private 方法。

(2)`Field[] fields = myClass.getFields();`
该方法返回某个类的所有 public 成员变量。

(3)`Field field = myClass.getField(name);`
该方法返回一个特定名称的成员变量。

获取到特定的 Field 之后，如果想要获取某个具体对象 obj 中该属性的值，看如下代码：

```java
public class Test{
	public static void main(String[] args) throws NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException{
		Class myClass = MyClass.class;
		MyClass obj = new MyClass();
		Field[] declaredFields = myClass.getDeclaredFields();
		for(Field field:declaredFields) {
			System.out.println(field.get(obj));
		}
	}
}
```

此时运行会报错，因为在这些 Field 中存在访问权限，比如说 private 的 age。因此，需要在使用 `field.get(obj)` 之前加上 `field.setAccessible(true);` 破坏访问权限即可。

相对的获取某个 Class 对象的方法（Method），主要有以下几个方法：

(1)`Method[] declaredMethods = myClass.getDeclaredMethods();`
该方法返回类或者接口声明的所有方法，包括 public, protected, 默认, private 方法，但不包括继承的方法。

(2)`Method[] methods = myClass.getMethods();`
该方法返回某个类的所有 public 方法，包括其继承类的公用方法。

(3)`Method method = myClass.getMethod(name, parameterTypes);`
该方法返回一个特定的方法，其中第一个参数为方法名称，后面的参数为方法的参数对应 Class 的对象。

获取到特定的 Method 之后，如果想要调用某个具体的方法，如下代码所示：

```java
public class Test{
	public static void main(String[] args) throws NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException, InvocationTargetException{
		Class myClass = MyClass.class;
		MyClass obj = new MyClass();
		Method[] declaredMethods = myClass.getDeclaredMethods();
		for(Method method:declaredMethods) {
//			method.setAccessible(true);    // 权限破坏
			System.out.println(method.invoke(obj));
		}
	}
}
```

权限方面和 Field 相似




#### <a id="relflection_tips">反射的一些注意事项</a>

首先，利用反射访问成员属性和方法时，也会因为有访问修饰符而存在权限的限制。利用 `setAccessible()` 方法会破坏封装性，并且在访问完方法后不会回复。因此要注意使用安全问题。
其次，利用反射会额外消耗系统资源，如果不需要动态创建或者使用成员变量和方法，尽量少用反射。



### <a id="generics">七、泛型</a>

关于泛型的内容，我在找参考资料的时候，发现了两篇博客，内容已经写得很全了，我这边就当个搬运工。另外，请大家尊重版权，我标明了原地址了！这不是原创！
搬运地址：
1、java 泛型详解-绝对是对泛型方法讲解最详细的，没有之一(https://blog.csdn.net/s10461/article/details/53941091)
2、10 道 Java 泛型面试题(https://cloud.tencent.com/developer/article/1033693)

#### <a id="generics_detail">泛型详解</a>

搬运自[泛型详解](https://blog.csdn.net/s10461/article/details/53941091)

**<a id="generics_detail_abstract">1、概述</a>**

泛型在 Java 中有很重要的地位，在面向对象编程及各种设计模式中有非常广泛的应用。

什么是泛型？为什么要使用泛型？
>泛型，即“参数化类型”。一提到参数，最熟悉的就是定义方法时有形参，然后调用此方法时传递实参。那么参数化类型怎么理解呢？顾名思义，就是将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。
>泛型的本质是为了参数化类型（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。也就是说在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。

**<a id="generics_detail_example">2、一个栗子</a>**

一个被举了无数次的例子：

```java
List arrayList = new ArrayList();
arrayList.add("aaaa");
arrayList.add(100);

for(int i = 0; i< arrayList.size();i++){
    String item = (String)arrayList.get(i);
    Log.d("泛型测试","item = " + item);
}
```

毫无疑问，程序的运行结果会以崩溃结束：

```java
java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
```

ArrayList 可以存放任何类型（当然，基础数据类型不行，得转换成包装类型才行），栗子中添加了一个 String 类型，添加了一个 Integer 类型，在使用时都以 String 的方式使用，因此程序崩溃了。为了解决类似这样的问题（在编译阶段就可以解决），泛型应运而生。

我们将第一行声明初始化 list 的代码更改一下，编译器会在编一阶段就能帮我们发现类似这样的问题。

```java
List<String> arrayList = new ArrayList<String>();
...
//arrayList.add(100); 在编译阶段，编译器就会报错
```

**<a id="generics_detail_characteristic">3、特性</a>**

泛型只在编一阶段有效。看下面代码：

```java
List<String> stringArrayList = new ArrayList<String>();
List<Integer> integerArrayList = new ArrayList<Integer>();

Class classStringArrayList = stringArrayList.getClass();
Class classIntegerArrayList = integerArrayList.getClass();

if(classStringArrayList.equals(classIntegerArrayList)){
    Log.d("泛型测试","类型相同");
}
```

输出结果：`D/泛型测试: 类型相同`。

通过上面的例子可以证明，在编译之后程序会采取去泛型化的措施。也就是说 Java 中的泛型，只在编译阶段有效。在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦除，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法。也就是说，泛型信息不会进入到运行时阶段。

**对此总结成一句话：泛型类型在逻辑上可以看成是多个不同的类型，实际上都是相同的基本类型。**

**<a id="generics_detail_usage">4、泛型的使用</a>**

泛型有三种使用方式，分别为：泛型类、泛型接口、泛型方法

**<a id="generics_detail_usage_class">4.1、泛型类</a>**

泛型类型用于类的定义中，被称为泛型类。通过泛型可以完成对、一组类的操作、对外开放相同的接口。（顿号用来句读）最典型的就是各种容器类，如：List、Set、Map。

泛型类的最基本写法（这么看可能会有点晕，会在下面的例子中详解）：

```java
class 类名称 <泛型标识：可以随便写任意标识号，标识指定的泛型的类型>{
  private 泛型标识 /*（成员变量类型）*/ var; 
  .....

}
```

一个最普通的泛型类：

```java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{ 
    //key这个成员变量的类型为T,T的类型由外部指定  
    private T key;

    public Generic(T key) { //泛型构造方法形参key的类型也为T，T的类型由外部指定
        this.key = key;
    }

    public T getKey(){ //泛型方法getKey的返回值类型为T，T的类型由外部指定
        return key;
    }
}
```

```java
//泛型的类型参数只能是类类型（包括自定义类），不能是简单类型
//传入的实参类型需与泛型的类型参数类型相同，即为Integer.
Generic<Integer> genericInteger = new Generic<Integer>(123456);

//传入的实参类型需与泛型的类型参数类型相同，即为String.
Generic<String> genericString = new Generic<String>("key_vlaue");
Log.d("泛型测试","key is " + genericInteger.getKey());
Log.d("泛型测试","key is " + genericString.getKey());
```

```java
12-27 09:20:04.432 13063-13063/? D/泛型测试: key is 123456
12-27 09:20:04.432 13063-13063/? D/泛型测试: key is key_vlaue
```

定义的泛型类，就一定要传入泛型类型实参么？并不是这样，在使用泛型的时候，如果传入泛型实参，则会根据传入的泛型实参做相应的限制，此时泛型才会起到本应起到的限制作用。如果不传泛型类型实参的话，在泛型类中使用泛型的方法或成员变量定义的类型可以为任何的类型。

```java
// 这里并没有写成 Generic<String> generic = new Generic("111111") , 没有前面 <> 中的类型限制，那么后面传入的实参也没有类型限制 
Generic generic = new Generic("111111");
Generic generic1 = new Generic(4444);
Generic generic2 = new Generic(55.55);
Generic generic3 = new Generic(false);

Log.d("泛型测试","key is " + generic.getKey());
Log.d("泛型测试","key is " + generic1.getKey());
Log.d("泛型测试","key is " + generic2.getKey());
Log.d("泛型测试","key is " + generic3.getKey());
```

```java
D/泛型测试: key is 111111
D/泛型测试: key is 4444
D/泛型测试: key is 55.55
D/泛型测试: key is false
```

注意：
* 泛型的类型参数只能是类类型，不能是简单类型；
* 不能对确切的泛型类型使用 instanceof 操作，如下面的操作是违法的，编译时会出错

```java
if(ex_num instanceof Generic<Number>){   
}
```

**<a id="generics_detail_usage_interface">4.2、泛型接口</a>**

泛型接口与泛型类的定义及使用基本相同。泛型接口常被用在各种类的生产器中，可以看一个例子：

```java
//定义一个泛型接口
public interface Generator<T> {
    public T next();
}
```

当实现泛型接口的类，未传入泛型实参时：

```java
/**
 * 未传入泛型实参时，与泛型类的定义相同，在声明类的时候，需将泛型的声明也一起加到类中
 * 即：class FruitGenerator<T> implements Generator<T>{
 * 如果不声明泛型，如：class FruitGenerator implements Generator<T>，编译器会报错："Unknown class"
 */
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {
        return null;
    }
}
```

当实现泛型接口的类，传入泛型实参时：

```java
/**
 * 传入泛型实参时：
 * 定义一个生产器实现这个接口,虽然我们只创建了一个泛型接口Generator<T>
 * 但是我们可以为T传入无数个实参，形成无数种类型的Generator接口。
 * 在实现类实现泛型接口时，如已将泛型类型传入实参类型，则所有使用泛型的地方都要替换成传入的实参类型
 * 即：Generator<T>，public T next();中的的T都要替换成传入的String类型。
 */
public class FruitGenerator implements Generator<String> {

    private String[] fruits = new String[]{"Apple", "Banana", "Pear"};

    @Override
    public String next() {
        Random rand = new Random();
        return fruits[rand.nextInt(3)];
    }
}
```

**<a id="generics_detail_usage_wildcards">4.3、泛型通配符</a>**

我们知道 `Integer` 是 `Number` 的一个子类，同时在特性章节中我们也验证过 `Generic<Integer>` 与 `generic<Number>` 实际上是相同的一种基本类型。那么问题来了，在使用 `Generic<Number>` 作为形参的方法中，能否使用 `Generic<Integer>` 的实例传入呢？在逻辑上类似于 `Generic<Number>` 和 `Generic<Integer>` 是否可以看成具有父子关系的泛型类型呢？

为了弄清楚这个问题，我们使用 `Generic<T>` 这个泛型类继续看下面的例子：

```java
public void showKeyValue1(Generic<Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}
```

```java
Generic<Integer> gInteger = new Generic<Integer>(123);
Generic<Number> gNumber = new Generic<Number>(456);

showKeyValue(gNumber);

// showKeyValue这个方法编译器会为我们报错：Generic<java.lang.Integer> 
// cannot be applied to Generic<java.lang.Number>
// showKeyValue(gInteger);
```

通过提示信息我们可以看到 `Generic<Integer>` 不能被看作为 `Generic<Number>` 的子类。由此可以看出：**同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。**

回到上面的例子，如何解决上面的问题？总不能为了定义一个新的方法来处理 `Generic<Integer>` 类型的类，这显然与 Java 中的多态理念相违背。因此我们需要在一个逻辑上可以表示**同时**是 `Generic<Integer>` 和 `Generic<Number>` 父类的引用类型。由此，类型通配符应运而生。

我们可以将上面的方法改一下：

```java
public void showKeyValue1(Generic<?> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}
```

类型通配符一般是使用 ? 代替具体的类型实参，注意了，**此处'?'是类型实参，而不是类型形参**！重要的话说三遍！**此处'?'是类型实参，而不是类型形参**！**此处'?'是类型实参，而不是类型形参**！**此处'?'是类型实参，而不是类型形参**！再直白的意思就是，此处的 ? 和 Number、String、Integer 一样都是一种实际的类型，可以把 ? 看成所有类型的父类。是一种真实的类型。
可以解决当具体类型不确定的时候，这个通配符就是 ? 。当操作类型时，不需要使用类型的具体功能时，只使用 `Object` 类中的功能。那么用 ? 通配符来表示未知类型。

**<a id="generics_detail_usage_methods">4.4、泛型方法</a>**

在 Java 中，泛型类的定义非常简单，但是泛型方法就比较复杂了。
>尤其是我们见到的大多数泛型类中的成员方法也都使用了泛型，有的甚至泛型类中也包含着泛型的方法，这样初学者非常容易将泛型方法理解错了。

**泛型类，是在实例化类的时候指明泛型的具体类型；泛型方法，是在调用方法的时候指明泛型具体类型。**

```java
/**
 * 泛型方法的基本介绍
 * @param tClass 传入的泛型实参
 * @return T 返回值为T类型
 * 说明：
 *     1）public 与 返回值中间<T>非常重要，可以理解为声明此方法为泛型方法。
 *     2）只有声明了<T>的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法。
 *     3）<T>表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T。
 *     4）与泛型类的定义一样，此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型。
 */
public <T> T genericMethod(Class<T> tClass)throws InstantiationException ,
  IllegalAccessException{
        T instance = tClass.newInstance();
        return instance;
}
```
```
Object obj = genericMethod(Class.forName("com.test.test"));
```
**<a id="generics_detail_usage_methods_usage">4.4.1、泛型方法的基本用法</a>**

光看上面的例子有的同学可能依然会非常迷糊，我们再通过一个例子，把泛型方法再总结一下。

```java
public class GenericTest {
   //这个类是个泛型类，在上面已经介绍过
   public class Generic<T>{     
        private T key;

        public Generic(T key) {
            this.key = key;
        }

        //我想说的其实是这个，虽然在方法中使用了泛型，但是这并不是一个泛型方法。
        //这只是类中一个普通的成员方法，只不过他的返回值是在声明泛型类已经声明过的泛型。
        //所以在这个方法中才可以继续使用 T 这个泛型。
        public T getKey(){
            return key;
        }

        /**
         * 这个方法显然是有问题的，在编译器会给我们提示这样的错误信息"cannot reslove symbol E"
         * 因为在类的声明中并未声明泛型E，所以在使用E做形参和返回值类型时，编译器会无法识别。
        public E setKey(E key){
             this.key = keu
        }
        */
    }

    /** 
     * 这才是一个真正的泛型方法。
     * 首先在public与返回值之间的<T>必不可少，这表明这是一个泛型方法，并且声明了一个泛型T
     * 这个T可以出现在这个泛型方法的任意位置.
     * 泛型的数量也可以为任意多个 
     *    如：public <T,K> K showKeyName(Generic<T> container){
     *        ...
     *        }
     */
    public <T> T showKeyName(Generic<T> container){
        System.out.println("container key :" + container.getKey());
        //当然这个例子举的不太合适，只是为了说明泛型方法的特性。
        T test = container.getKey();
        return test;
    }

    //这也不是一个泛型方法，这就是一个普通的方法，只是使用了Generic<Number>这个泛型类做形参而已。
    public void showKeyValue1(Generic<Number> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
    }

    //这也不是一个泛型方法，这也是一个普通的方法，只不过使用了泛型通配符?
    //同时这也印证了泛型通配符章节所描述的，?是一种类型实参，可以看做为Number等所有类的父类
    public void showKeyValue2(Generic<?> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
    }

     /**
     * 这个方法是有问题的，编译器会为我们提示错误信息："UnKnown class 'E' "
     * 虽然我们声明了<T>,也表明了这是一个可以处理泛型的类型的泛型方法。
     * 但是只声明了泛型类型T，并未声明泛型类型E，因此编译器并不知道该如何处理E这个类型。
    public <T> T showKeyName(Generic<E> container){
        ...
    }  
    */

    /**
     * 这个方法也是有问题的，编译器会为我们提示错误信息："UnKnown class 'T' "
     * 对于编译器来说T这个类型并未项目中声明过，因此编译也不知道该如何编译这个类。
     * 所以这也不是一个正确的泛型方法声明。
    public void showkey(T genericObj){

    }
    */

    public static void main(String[] args) {


    }
}
```

**<a id="generics_detail_usage_class_methods">4.4.2、类中的泛型方法</a>**

当然这不是泛型方法的全部，泛型方法可以出现在任何地方和任何场景中。但是有一种情况是非常特殊的，当泛型方法出现在泛型类中，我们再通过一个例子看一下：

```java
public class GenericFruit {
    class Fruit{
        @Override
        public String toString() {
            return "fruit";
        }
    }

    class Apple extends Fruit{
        @Override
        public String toString() {
            return "apple";
        }
    }

    class Person{
        @Override
        public String toString() {
            return "Person";
        }
    }

    class GenerateTest<T>{
        public void show_1(T t){
            System.out.println(t.toString());
        }

        //在泛型类中声明了一个泛型方法，使用泛型E，这种泛型E可以为任意类型。可以类型与T相同，也可以不同。
        //由于泛型方法在声明的时候会声明泛型<E>，因此即使在泛型类中并未声明泛型，编译器也能够正确识别泛型方法中识别的泛型。
        public <E> void show_3(E t){
            System.out.println(t.toString());
        }

        //在泛型类中声明了一个泛型方法，使用泛型T，注意这个T是一种全新的类型，可以与泛型类中声明的T不是同一种类型。
        public <T> void show_2(T t){
            System.out.println(t.toString());
        }
    }

    public static void main(String[] args) {
        Apple apple = new Apple();
        Person person = new Person();

        GenerateTest<Fruit> generateTest = new GenerateTest<Fruit>();
        //apple是Fruit的子类，所以这里可以
        generateTest.show_1(apple);
        //编译器会报错，因为泛型类型实参指定的是Fruit，而传入的实参类是Person
        //generateTest.show_1(person);

        //使用这两个方法都可以成功
        generateTest.show_2(apple);
        generateTest.show_2(person);

        //使用这两个方法也都可以成功
        generateTest.show_3(apple);
        generateTest.show_3(person);
    }
}
```

**<a id="generics_detail_usage_methods&params">4.4.3、泛型方法与可变参数</a>**

再看一个泛型方法和可变参数的例子：

```java
public <T> void printMsg( T... args){
    for(T t : args){
        Log.d("泛型测试","t is " + t);
    }
}
```

```java
printMsg("111",222,"aaaa","2323.4",55.55);
```

**<a id="generics_detail_usage_methods&params">4.4.4、静态方法与泛型</a>**

静态方法有一种情况需要注意一下，那就是在类中的静态方法使用泛型：**静态方法无法访问类上定义的泛型；如果静态方法操作的引用数据类型不确定的时候，必须要将泛型定义在方法上。**
即：**如果静态方法要使用泛型的话，必须将静态方法也定义成泛型方法**

```java
public class StaticGenerator<T> {
    ....
    ....
    /**
     * 如果在类中定义使用泛型的静态方法，需要添加额外的泛型声明（将这个方法定义成泛型方法）
     * 即使静态方法要使用泛型类中已经声明过的泛型也不可以。
     * 如：public static void show(T t){..},此时编译器会提示错误信息：
          "StaticGenerator cannot be refrenced from static context"
     */
    public static <T> void show(T t){

    }
}
```

**<a id="generics_detail_usage_method_conclusion">4.4.5、泛型方法总结</a>**

泛型方法能使方法独立于类而产生变化，以下是一个基本的指导原则：
>无论何时，如果你能做到，你就该尽量使用泛型方法。也就是说，如果使用泛型方法将整个类泛型化，那么就应该使用泛型方法。另外对于一个static的方法而已，无法访问泛型类型的参数。所以如果static方法要使用泛型能力，就必须使其成为泛型方法。

**<a id="generics_detail_bound">5、泛型上下边界</a>**

在使用泛型的时候，我们还可以为传入的泛型类型实参进行上下边界的限制，如：类型实参只准传入某种类型的父类或某种类型的子类。

```java
public void showKeyValue1(Generic<? extends Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}
```

```java
Generic<String> generic1 = new Generic<String>("11111");
Generic<Integer> generic2 = new Generic<Integer>(2222);
Generic<Float> generic3 = new Generic<Float>(2.4f);
Generic<Double> generic4 = new Generic<Double>(2.56);

//这一行代码编译器会提示错误，因为String类型并不是Number类型的子类
//showKeyValue1(generic1);

showKeyValue1(generic2);
showKeyValue1(generic3);
showKeyValue1(generic4);
```

如果我们把泛型类的定义也改一下:

```java
public class Generic<T extends Number>{
    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}
```

```java
//这一行代码也会报错，因为String不是Number的子类
Generic<String> generic1 = new Generic<String>("11111");
```

再来一个泛型方法的例子：

```java
//在泛型方法中添加上下边界限制的时候，必须在权限声明与返回值之间的<T>上添加上下边界，即在泛型声明的时候添加
//public <T> T showKeyName(Generic<T extends Number> container)，编译器会报错："Unexpected bound"
public <T extends Number> T showKeyName(Generic<T> container){
    System.out.println("container key :" + container.getKey());
    T test = container.getKey();
    return test;
}
```

通过上面的两个例子可以看出：**泛型的上下边界添加，必须与泛型的声明在一起**。

如果是下边界的时候，关键字使用 `super`，其他格式不变

**<a id="generics_detail_array">6、关于泛型数组要提一下</a>**

看到了很多文章中都会提起泛型数组，经过查看sun的说明文档，在java中是”不能创建一个确切的泛型类型的数组”的。

也就是说下面的这个例子是不可以的：

```java
List<String>[] ls = new ArrayList<String>[10];  
```

而使用通配符创建泛型数组是可以的，如下面这个例子：

```java
List<String>[] ls = new ArrayList[10];
```

下面使用[Sun的一篇文档](https://docs.oracle.com/javase/tutorial/extra/generics/fineprint.html)的一个例子来说明这个问题：

```java
List<String>[] lsa = new List<String>[10]; // Not really allowed.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Unsound, but passes run time store check    
String s = lsa[1].get(0); // Run-time error: ClassCastException.
```

>这种情况下，由于JVM泛型的擦除机制，在运行时JVM是不知道泛型信息的，所以可以给oa[1]赋上一个ArrayList而不会出现异常，但是在取出数据的时候却要做一次类型转换，所以就会出现ClassCastException，如果可以进行泛型数组的声明，上面说的这种情况在编译期将不会出现任何的警告和错误，只有在运行时才会出错。
而对泛型数组的声明进行限制，对于这样的情况，可以在编译期提示代码有类型安全问题，比没有任何提示要强很多。

下面采用通配符的方式是被允许的:**数组的类型不可以是类型变量，除非是采用通配符的方式**，因为对于通配符的方式，最后取出数据是要做显式的类型转换的。

```java
List<?>[] lsa = new List<?>[10]; // OK, array of unbounded wildcard type.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Correct.    
Integer i = (Integer) lsa[1].get(0); // OK 
```

**<a id="generics_detail_conclusion">7、最后</a>**

本文中的例子主要是为了阐述泛型中的一些思想而简单举出的，并不一定有着实际的可用性。另外，一提到泛型，相信大家用到最多的就是在集合中，其实，在实际的编程过程中，自己可以使用泛型去简化开发，且能很好的保证代码质量。
(搬运自[泛型详解](https://blog.csdn.net/s10461/article/details/53941091)，加了点自己的看法以及对原博客错误的修正)



#### <a id="generics_questions">泛型相关问题</a>

搬运自[泛型相关问题](https://cloud.tencent.com/developer/article/1033693)

**1、Java中的泛型是什么 ? 使用泛型的好处是什么?**

A: 这是在各种Java泛型面试中，一开场你就会被问到的问题中的一个，主要集中在初级和中级面试中。那些拥有Java1.4或更早版本的开发背景的人都知道，在集合中存储对象并在使用前进行类型转换是多么的不方便。泛型防止了那种情况的发生。它提供了编译期的类型安全，确保你只能把正确类型的对象放入集合中，避免了在运行时出现`ClassCastException`。
（简单来说，一是避免频繁的强制类型转换，二是使得类和方法具有通用性）

**2、Java的泛型是如何工作的 ? 什么是类型擦除 ?**

A: 这是一道更好的泛型面试题。泛型是通过类型擦除来实现的，编译器在编译时擦除了所有类型相关的信息，所以在运行时不存在任何类型相关的信息。例如`List<String>`在运行时仅用一个`List`来表示。这样做的目的，是确保能和Java 5之前的版本开发二进制类库进行兼容。你无法在运行时访问到类型参数，因为编译器已经把泛型类型转换成了原始类型。根据你对这个泛型问题的回答情况，你会得到一些后续提问，比如为什么泛型是由类型擦除来实现的或者给你展示一些会导致编译器出错的错误泛型代码。请阅读我的Java中泛型是如何工作的来了解更多信息。

**3、什么是泛型中的限定通配符和非限定通配符 ?**

A: 这是另一个非常流行的Java泛型面试题。限定通配符对类型进行了限制。有两种限定通配符，一种是`<? extends T>`它通过确保类型必须是T的子类来设定类型的上界，另一种是`<? super T>`它通过确保类型必须是T的父类来设定类型的下界。泛型类型必须用限定内的类型来进行初始化，否则会导致编译错误。另一方面`<?>`表示了非限定通配符，因为`<?>`可以用任意类型来替代。更多信息请参阅我的文章泛型中限定通配符和非限定通配符之间的区别。
（非限定通配符 `?`，限定通配符是对 `?` 进行了上下界限定，使用 `extends` 和 `super` 关键字进行限定）

**4、`List<? extends T>`和`List <? super T>`之间有什么区别 ?**

A: 这和上一个面试题有联系，有时面试官会用这个问题来评估你对泛型的理解，而不是直接问你什么是限定通配符和非限定通配符。这两个 Lis t的声明都是限定通配符的例子，`List<? extends T>`可以接受任何继承自T的类型的`List`，而`List<? super T>`可以接受任何`T`的父类构成的`List`。例如`List<? extends Number>`可以接受`List<Integer>`或`List<Float>`。在本段出现的连接中可以找到更多信息。

**5、如何编写一个泛型方法，让它能接受泛型参数并返回泛型类型?**

A: 编写泛型方法并不困难，你需要用泛型类型来替代原始类型，比如使用`T, E or K,V`等被广泛认可的类型占位符。泛型方法的例子请参阅 Java 集合类框架。最简单的情况下，一个泛型方法可能会像这样:

```java
public <K,T,V> V put(K key, T value){
	return cache.put(key, value);
}
```
(原博的方法和上面关于泛型方法的说明相违背，这边列举了其他的例子)

**6. Java中如何使用泛型编写带有参数的类?**

A: 这是上一道面试题的延伸。面试官可能会要求你用泛型编写一个类型安全的类，而不是编写一个泛型方法。关键仍然是使用泛型类型来代替原始类型，而且要使用 JDK 中采用的标准占位符。

**7. 编写一段泛型程序来实现LRU缓存?**

A: 对于喜欢 Java 编程的人来说这相当于是一次练习。给你个提示， LinkedHashMap 可以用来实现固定大小的LRU缓存，当LRU缓存已经满了的时候，它会把最老的键值对移出缓存。LinkedHashMap 提供了一个称为`removeEldestEntry()`的方法，该方法会被`put()`和`putAll()`调用来删除最老的键值对。当然，如果你已经编写了一个可运行的 JUnit 测试，你也可以随意编写你自己的实现代码。

**8、你可以把`List<String>`传递给一个接受`List<Object>`参数的方法吗？**

A: 对任何一个不太熟悉泛型的人来说，这个Java泛型题目看起来令人疑惑，因为乍看起来 String 是一种 Object ，所以`List<String>`应当可以用在需要`List<Object>`的地方，但是事实并非如此。真这样做的话会导致编译错误。如果你再深一步考虑，你会发现 Java 这样做是有意义的，因为`List<Object>`可以存储任何类型的对象包括 String,  Integer 等等，而`List<String>`却只能用来存储 Strings 。

**9、Array中可以用泛型吗?**

A: 这可能是 Java 泛型面试题中最简单的一个了，当然前提是你要知道Array事实上并不支持泛型，这也是为什么 Joshua Bloch 在 Effective Java 一书中建议使用 List 来代替 Array，因为 List 可以提供编译期的类型安全保证，而 Array 却不能。

**10、如何阻止Java中的类型未检查的警告?**

A: 如果你把泛型和原始类型混合起来使用，例如下列代码，Java 5 的 javac 编译器会产生类型未检查的警告，例如`List<String> rawList = new ArrayList()`

(搬运自[泛型相关问题](https://cloud.tencent.com/developer/article/1033693))

### <a id="annotation">八、注解（标注）</a>

#### <a id="annotation_basic">注解基础</a>

**<a id="annotation_basic_what">什么是注解</a>**

注解（Annotation）是 JDK 5 以后引入的一种机制，又称标注。它的使用方法和注释基本相同，但与注释不同的是，JVM 可以通过**反射**的方法获取注解的内容，从而起到对修饰元素（包、类、方法、成员变量、参数以及本地变量等）起到说明和配置的功能。

**用处**

* 最常用的功能 - 生成文档。比如说 JDK 中自带的 `@param`、`@return`
* 格式检查。比如说 JDK 中自带的 `@override`、`@SuppressWarnings`，前者用来检查方法是不是覆盖了父类方法，后者用来忽略编译器的 warnning 提示。
* 跟踪代码依赖性，实现替代配置文件的功能。比如 Spring-boot 中的 `@Bean`、`@Controller`等等

**<a id="annotation_basic_principle">注解的原理</a>**

>注解本质是一个继承了`Annotation`的特殊接口，其具体实现类是 Java 运行时生成的动态代理类。而我们通过反射获取注解时，返回的是 Java 运行时生成的动态代理对象`$Proxy1`。通过代理对象调用自定义注解（接口）的方法，会最终调用`AnnotationInvocationHandler`的 `invoke`方法。该方法会从`memberValues`这个 Map 中索引出对应的值。而`memberValues`的来源是 Java 常量池。

一个注解其实就是一种特殊的注释，通过解析从而赋予它特殊的能力。而解析注解的方式有两种：1、编译期直接的扫描；2、运行时反射。
编译期的扫描指的是在对 Java 代码编译字节码的过程中会检测到某个类或者方法被一些注释修饰，这时它就会对于这些注解进行某些处理。
运行时反射指的是利用 JVM 通过动态代理机制生成注释的代理类，并触发方法。







参考文献
[Oracle Java Tutorials](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)
[百度经验](https://jingyan.baidu.com)
[菜鸟教程](https://www.runoob.com/java/java-tutorial.html)
[cs-notes:Java 基础]（https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20%E5%9F%BA%E7%A1%80.md）
深入理解Java虚拟机
[深入浅出java常量池](https://www.cnblogs.com/syp172654682/p/8082625.html)
[Java 中容易混淆的概念：Java 8 中的常量池、字符串池、包装类对象池](https://blog.csdn.net/Xu_JL1997/article/details/89150026)
[深入理解Java并发之synchronized实现原理](https://blog.csdn.net/javazejian/article/details/72828483#synchronized%E7%9A%84%E4%B8%89%E7%A7%8D%E5%BA%94%E7%94%A8%E6%96%B9%E5%BC%8F)
[详解Java中的异常(Error与Exception)](https://blog.csdn.net/qq_29229567/article/details/80773970?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2)
[泛型详解](https://blog.csdn.net/s10461/article/details/53941091)
[泛型相关问题](https://cloud.tencent.com/developer/article/1033693)