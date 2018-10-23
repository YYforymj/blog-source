---
title: jdk源码阅读01
date: 2018-10-23 20:55:17
categories: jdk
---

> jdk 源码阅读简单记录。

<!-- more -->

### java.lang

#### Object

Object 是所有类的父类，所以从它看起吧（idea 支持查看当前窗口类所有的方法，alt + 7 即可唤出）。

首先看静态方法：

```
private static native void registerNatives();
static {
registerNatives();
}
```

这里的代码涉及到 c++ 部分的代码，不是重点，简单看了下 c++ 实现，大概就是将 Object 类的方法在 c++ 中注册一下（暂且这么理解，其实底层应该是函数指针的传递）；

```
public native int hashCode();
```

获取 hashcode，注释表明了该函数的一些特性：

- 在同一个 jvm 中，对于同一对象调用 hashCode() 方法总会返回相同的哈希值
- 两个不同的对象有可能返回相同的哈希值，但是为了提高 hash table 的性能最好控制使之产生不同哈希值

简单看了下 cpp 文件，对于同一个对象，首次计算哈希值后即存储起来再次获取哈希值时会直接返回已经生成的哈希值；而 cpp 中在计算哈希值时有一项用于计算哈希值的 _hashStateX 属性，每次计算哈希值会调用 random 函数。

```
 public boolean equals(Object obj) {
        return (this == obj);
    }
```

对于 equals() 方法，注释写的很明白，如果需要覆盖 equals() 方法，则需要同时重写 hashCode() 方法，保证相同对象拥有相同的哈希值。

```
protected native Object clone() throws CloneNotSupportedException;
```

clone 方法是 native 方法，所以执行效率与 java 方法比较起来会高一些，但是最大的问题是需要实现类继承 Cloneable 接口，否则会报 CloneNotSupportedException 异常，这对于单继承的 java 来说不是特别友好，同时又有浅拷贝的问题（可以参见[这里](https://blog.csdn.net/zhangjg_blog/article/details/18369201)），所以平时用的不多。

```
public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

对于没有重写 toString() 的类，Object 提供了基本的 toString() 方法，不过个人更习惯使用 lombok，方便简洁。

```
public final native void notify();
```

从阻塞在该对象的线程中唤醒一个线程，唤醒的线程视具体不同的实现会有随机的唤醒；唤醒后线程处于等待状态，同其他等待线程具有相同的获取锁的优先级。notifyAll() 则是唤醒所有阻塞在该对象上的线程。

此外，Object 类还有 wait() 的三个方法。wait() 方法的作用是让当前持有锁的线程放弃锁进入等待队列，并进行等待，直至其他方法调用 notify() 或 notifyAll() 唤醒该线程，则该线程继续执行。对于有参数的两个 wait() 方法，参数用于指定超时时间。需要注意的是，由于 wait() 方法会放弃 monitor，所以对于 wait() 的调用线程必须持有 monitor。

#### String

String 类是被 final 修饰的，所以 String 具有**不可变性**；

```
/** The value is used for character storage. */
private final char value[];

/** Cache the hash code for the string */
private int hash; // Default to 0
```

两个主要类成员，其中字符数组 value[] 用于存储字符串信息；hash 用于缓存哈希值。

```
public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
    }
```

String 的第一个含参构造方法，注释上表示，该构造方法会拷贝一份传入参数，生成一个新的对象。也就是说，如下语句会产生两个对象：

```
String str = new String("abc");
```

```
public String(byte bytes[], String charsetName)
            throws UnsupportedEncodingException {
        this(bytes, 0, bytes.length, charsetName);
    }
```

用于获取指定编码格式的 String 对象。

```
public String(StringBuffer buffer) {
    synchronized(buffer) {
    this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
    }
}
```

此构造方法用于将 StringBuffer 转换为，可以看出其实就是对字符数组的操作。

```
public int hashCode() {
    int h = hash;
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

String 类的 hash 函数，计算公式为 s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]。

```
public String concat(String str) {
        int otherLen = str.length();
        if (otherLen == 0) {
            return this;
        }
        int len = value.length;
        char buf[] = Arrays.copyOf(value, len + otherLen);
        str.getChars(buf, len);
        return new String(buf, true);
    }
```

concat() 方法用于在字符串尾部添加字符串，源码中可以看出，依旧是对数组的操作。

```
public String trim() {
        int len = value.length;
        int st = 0;
        char[] val = value;    /* avoid getfield opcode */

        while ((st < len) && (val[st] <= ' ')) {
            st++;
        }
        while ((st < len) && (val[len - 1] <= ' ')) {
            len--;
        }
        return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
    }
```

trim() 方法通常是用来去除空格的，但是这里通过源码可以得知其实去除的不仅仅是空格，控制字符也一并去掉了（ascii 表中小于空格值的字符都是控制字符，例如换行符制表符等，详细可以查询 ascii 表）。对此进行如下实验：

```
String str = "\n test \t";
System.out.println(str.length());
System.out.println(str.trim().length());
```

输出结果为 8 ，4。证明去除了换行符与制表符。

```
public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin) {
        if (srcBegin < 0) {
            throw new StringIndexOutOfBoundsException(srcBegin);
        }
        if (srcEnd > value.length) {
            throw new StringIndexOutOfBoundsException(srcEnd);
        }
        if (srcBegin > srcEnd) {
            throw new StringIndexOutOfBoundsException(srcEnd - srcBegin);
        }
        System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin);
    }
```

这个函数用于拷贝字符数组，后面 StringBuilder 会用到。

```
public native String intern();
```

intern() 方法是 String 类的最后一个方法，也是最有趣的一个。注释表明，String 类在 jvm 启动后维护了一个字符串常量池，调用 intern() 方法后，如果常量池中有该字符串，则返回常量池中的对象，否则添加该对象到常量池中。此外，**所有的 Literal String （就是以引号包裹的字符串，例如 String s = "abc" 中，"abc" 就是一个 Literal String）与字符串常量都存储在常量池中**。

#### AbstractStringBuilder

AbstractStringBuilder 是 StringBuilder 与 StringBuffer 的父类本身被定义为抽象类，有一些 StringBuilder 与 StringBuffer 公用的方法。同 String 一样，本身使用字符串数组作为内容存储的容器，有如下两个成员：

```
/**
* The value is used for character storage.
*/
char[] value;

/**
* The count is the number of characters used.
*/
int count;
```

```
private void ensureCapacityInternal(int minimumCapacity) {
        // overflow-conscious code
        if (minimumCapacity - value.length > 0) {
            value = Arrays.copyOf(value,
                    newCapacity(minimumCapacity));
        }
    }
```

ensureCapacityInternal() 用于为当前对象扩容。

```
public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
```

append(String str) 大概是其中最常用的方法，通过源码可以看出，首先判断追加字符串是否为空，如果为空的话在最后添加 "null" 四个字符（这里比较奇怪，类的注释上说明了该类的任何方法接受到 null 会抛出 NullPointerException）；否则，首先对字符数组进行扩容，然后复制字符数据并为长度数据赋值完成追加。（另：在 CharSequence 接口中发现 jdk 1.8 的特性：interface 可以包含有方法体的方法，由此可见 java 最初的单继承设计是比较糟糕的）

#### StringBuffer、StringBuilder

注释标明了这两个类的功能几乎是一样的，除了 StringBuffer 是线程安全的，而 StringBuilder 是非线程安全的。而这两个类的最常用的方法就是 append() 与 insert() 了，这里不赘述。

#### Boolean 

没什么好说的，boolean 的包装类，可以从 String 中解析布尔值。

#### Byte

```
/**
* A constant holding the minimum value a {@code byte} can
* have, -2<sup>7</sup>.
*/
public static final byte   MIN_VALUE = -128;

/**
* A constant holding the maximum value a {@code byte} can
* have, 2<sup>7</sup>-1.
*/
public static final byte   MAX_VALUE = 127;
```

同 byte 一样有上下限。

```
private static class ByteCache {
        private ByteCache(){}

        static final Byte cache[] = new Byte[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Byte((byte)(i - 128));
        }
    }

    public static Byte valueOf(byte b) {
        final int offset = 128;
        return ByteCache.cache[(int)b + offset];
    }

```

这两个方法表明了，Byte 类维护了一个对象池，该对象池中持有该类所有可能的取值，在调用 valueOf() 方法时，返回池中对应对象。

此外同基本类型一样，Byte 的一些方法也是先转换为 int 或其包装类再进行处理。

#### Double 、Float

这两个类比较类似，没有什么值得说的。

#### Integer

toString(int i, int radix) 方法按照 radix 指定的基数进行转换，用于将 Integer 对象转换为各个进制格式的字符串。当然也可以使用 toHexString(int i)、toOctalString(int i)、toBinaryString(int i) 等进行转换。

public static Integer decode(String nm) 用于解析数字，可以接受十六进制（0x、0X、# 开头）或者八进制（0 开头）数字。

同 Byte 一样，有缓存的对象池，代码如下

```
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
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

其中对象池上限可以通过 jvm 的设置  -XX:AutoBoxCacheMax=<size> 来设定。

#### Long

同 Byte 一样有对象池，范围也是 -128 ~ 127，不可以设定上下限。

#### ThreadLocal

每个Thread维护着一个ThreadLocalMap的引用，ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储。调用ThreadLocal的set()方法时，实际上就是往ThreadLocalMap设置值，key是ThreadLocal对象，值是传递进来的对象。调用ThreadLocal的get()方法时，实际上就是往ThreadLocalMap获取值，key是ThreadLocal对象，ThreadLocal本身并不存储值，它只是作为一个key来让线程从ThreadLocalMap获取value。

#### Class

注释表明，枚举是一种类，注解是一种接口；而接口也可以通过 Class 类进行判断。