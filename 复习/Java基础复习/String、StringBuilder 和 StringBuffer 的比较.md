## String、StringBuilder 和 StringBuffer 的比较
首先看一下这三者的源码
String:
```
	public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    	private final char value[];
    	...
    }
```
StringBuilder:
```
public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
{
	...
}
```
StringBuffer:
```
public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
{
	...
}
```
下图表示三者的继承情况：
