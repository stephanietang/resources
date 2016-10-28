# String

标签： Java

---
## String Literal v.s. String Object

```java
String strObject = new String("Java");
```
会在heap中新建一个object

```java
String strLiteral = "Java";
```

会查找字符串常量池(String Pool：Java 7之前属于PermGen的一部分，Java 7之后移入heap）中是否已经存在相同的字符串，如果不存在，则在字符串常量池中创建一个，如果已经存在，则返回池中的字符串引用。

> `string pool`又可以叫`string literal pool`/`string constant pool`

[JDK 7文档](http://docs.oracle.com/javase/7/docs/technotes/guides/vm/enhancements-7.html)中提到：

> In JDK 7, interned strings are no longer allocated in the permanent generation of the Java heap, but are instead allocated in the main part of the Java heap (known as the young and old generations), along with the other objects created by the application. This change will result in more data residing in the main Java heap, and less data in the permanent generation, and thus may require heap sizes to be adjusted. Most applications will see only relatively small differences in heap usage due to this change, but larger applications that load many classes or make heavy use of the String.intern() method will see more significant differences.

## String是不可变类的好处

- 节约内存：只有String是不可变的，字符串池(String Pool)才有可能实现，不同的字符串变量可以指向池中的同一个字符串，这样节约很多内存空间
- 线程安全问题：多线程情况下，操作不可变对象时不需要担心线程安全的问题，因此不会存在线程阻塞
- 安全问题：譬如数据库的用户名、密码、文件名通常都是以String的形式传入，如果String是可变的，传入之后如果可以被hacker线程修改，那么必然会有安全隐患
- 适合作为Map的Key:因为不可变性，hashcode是确定的，那么在Map中的bucket位置也会确定而不改变，因此很适合做Map的Key

[参考1](http://javarevisited.blogspot.hk/2010/10/why-string-is-immutable-or-final-in-java.html)|[参考2](http://www.artima.com/intv/gosling313.html)

## 如何创建不可变类

要创建不可变类，要实现下面几个步骤：

- 将类声明为final，所以它不能被继承
- 将所有的成员声明为私有的，这样就不允许直接访问这些成员
- 对变量不要提供setter方法
- 将所有可变的成员声明为final，这样只能对它们赋值一次
- 通过构造器初始化所有成员，进行深拷贝(deep copy)
- 在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝

下面就是一个不可变类：
```java
package com.journaldev.java;

import java.util.HashMap;
import java.util.Iterator;

public final class FinalClassExample {

	private final int id;
	
	private final String name;
	
	private final HashMap<String,String> testMap;
	
	public int getId() {
		return id;
	}


	public String getName() {
		return name;
	}

	/**
	 * Accessor function for mutable objects
	 */
	public HashMap<String, String> getTestMap() {
		//return testMap;
		return (HashMap<String, String>) testMap.clone();
	}

	/**
	 * Constructor performing Deep Copy
	 * @param i
	 * @param n
	 * @param hm
	 */
	public FinalClassExample(int i, String n, HashMap<String,String> hm){
		System.out.println("Performing Deep Copy for Object initialization");
		this.id = i;
		this.name = n;
		HashMap<String,String> tempMap = new HashMap<String,String>();
		String key;
		Iterator<String> it = hm.keySet().iterator();
		while(it.hasNext()){
			key = it.next();
			tempMap.put(key, hm.get(key));
		}
		this.testMap=tempMap;
	}
	
	
	/**
	 * Constructor performing Shallow Copy
	 * @param i
	 * @param n
	 * @param hm
	 */
	/**
	public FinalClassExample(int i, String n, HashMap<String,String> hm){
		System.out.println("Performing Shallow Copy for Object initialization");
		this.id=i;
		this.name=n;
		this.testMap=hm;
	}
	*/
	
	/**
	 * To test the consequences of Shallow Copy and how to avoid it with Deep Copy for creating immutable classes
	 * @param args
	 */
	public static void main(String[] args) {
		HashMap<String, String> h1 = new HashMap<String,String>();
		h1.put("1", "first");
		h1.put("2", "second");
		
		String s = "original";
		
		int i=10;
		
		FinalClassExample ce = new FinalClassExample(i,s,h1);
		
		//Lets see whether its copy by field or reference
		System.out.println(s==ce.getName());
		System.out.println(h1 == ce.getTestMap());
		//print the ce values
		System.out.println("ce id:"+ce.getId());
		System.out.println("ce name:"+ce.getName());
		System.out.println("ce testMap:"+ce.getTestMap());
		//change the local variable values
		i=20;
		s="modified";
		h1.put("3", "third");
		//print the values again
		System.out.println("ce id after local variable change:"+ce.getId());
		System.out.println("ce name after local variable change:"+ce.getName());
		System.out.println("ce testMap after local variable change:"+ce.getTestMap());
		
		HashMap<String, String> hmTest = ce.getTestMap();
		hmTest.put("4", "new");
		
		System.out.println("ce testMap after changing variable from accessor methods:"+ce.getTestMap());

	}

}
```

## String真的不可变吗？

然而，我们可以用反射(reflection)，反射出String对象中的value属性， 进而改变通过获得的value引用改变数组的结构。

```java
public static void testReflection() throws Exception {

		String s = "Hello World"; 

		System.out.println("s = " + s);	//Hello World

		Field valueFieldOfString = String.class.getDeclaredField("value");

		valueFieldOfString.setAccessible(true);

		char[] value = (char[]) valueFieldOfString.get(s);

		value[5] = '_';

		System.out.println("s = " + s);  //Hello_World
	}
```

## String.substring()

JDK 6最新版本中（JDK6u45）
```java
String(int offset, int count, char value[]) {
	this.value = value;
	this.offset = offset;
	this.count = count;
}

public String substring(int beginIndex, int endIndex) {
	... //check boundary
	
    return new String(offset + beginIndex, endIndex - beginIndex, value);
}
```
![JDK 6 String.substring()](https://c1.staticflickr.com/6/5663/29959523264_7bc93bef02_o.jpg)

JDK 7最新版本中（JDK7u80）
```java
public String(char value[], int offset, int count) {
    ... //check boundary
        
    this.value = Arrays.copyOfRange(value, offset, offset+count);
}

public String substring(int beginIndex, int endIndex) {
    ... //check boundary
    
    int subLen = endIndex - beginIndex;
    return new String(value, beginIndex, subLen);
}
```
![JDK 7 String.substring()](https://c4.staticflickr.com/6/5462/30591441115_360be9e5a6_o.jpg)

因为实现细节不同，对于下面的代码，JDK 6会抛出OutOfMemoryError，而JDK 7不会。

```java
public class TestGC {
    private String largeString = new String(new byte[100000]);
    private String smallString = "foo";
    
    String getString() {
        // if caller stores this substring, this object will not be gc'ed
        return this.largeString.substring(0,2);
//        return new String(this.largeString.substring(0,2)); // no error here!
//        return smallString; // no error here!
    }
    
    public static void main(String[] args) {
        java.util.ArrayList list = new java.util.ArrayList();
        for (int i = 0; i < 1000000; i++) {
            TestGC gc = new TestGC();
            list.add(gc.getString());
        }
    }
}
```

对于String.substring()的改变褒贬不一。
Java 6的实现，当进行substring时，使用共享内容字符数组，速度会更快，不用重新申请内存。虽然有可能出现上面的内存性能问题，但也是有方法可以解决的，譬如可以使用`return new String(this.largeString.substring(0,2));`
Java 7的实现不需要程序员特殊操作避免了本文中问题，但是进行每次substring的操作性能总会比Java 6 的实现要差一些。这种实现显得有点“糟糕”。我更倾向JDK 6的实现。

[参考](http://www.programcreek.com/2013/09/the-substring-method-in-jdk-6-and-jdk-7/)|[参考2](http://bugs.java.com/view_bug.do?bug_id=4513622)

## String.intern()

String.intern()方法是一个native方法，作用是尽可能的减少重复的字符串，节约内存。虽然intern()方法会耗时，但如果使用正确的话，可以节约不少内存。对于一个使用大量重复字符串的系统，内存成为瓶颈的话，这时候正确使用intern()会节约很多内存，而对效率的影响微不足道。如何正确使用，后面有提到。

> intern:如果常量池中存在当前字符串，就会直接返回当前字符串。如果常量池中没有此字符串, 会将此字符串放入常量池中后, 再返回。

### JDK6和JDK7中String.intern()的区别

```java
class Test {
    public static void main(String args[]) {
        String s1 = new String("1") + new String("2");
        String s2 = s1.intern();

        System.out.println(s1 == s2);
    }

}
```

JDK 6运行结果是`false`
JDK 7运行结果是`true`

产生的原因在于JDK7中已经将String Pool从PermGen移到heap中。
在Java 6中，如果String Pool中没有该字符串，则在String Pool中创建这个字符串，并返回对这个字符串的引用。

![Java 6中的String.intern()](https://c2.staticflickr.com/6/5755/30312229290_8585d389dd_o.png)

而Java 7中，由于String Pool在heap中，如果String Pool没有该字符串，再查看heap中有没有创建该字符串，如果已经创建就将String Pool引用指向heap中已经创建的字符串，并返回该引用。

![Java 7中的String.intern()](https://c2.staticflickr.com/6/5745/30523844201_51bee65108_o.png)

### String.intern()的性能

- String Pool底层使用StringTable数据结构保存字符串引用，实现和HashMap类似，根据字符串的hashcode定位到对应的数组，遍历链表查找字符串，当字符串比较多时(相对于StringTable大小而言)，碰撞较多，会降低intern()的效率。
- 在JDK6中，由于常量池在PermGen中，受到PermGen大小的限制，不建议使用该方法。
- JDK7、8将String Pool放到heap中，可以调高StringTable的大小，使得可以intern更多的字符串。
- Java 6和Java 7u40之前，-XX:StringTableSize默认大小是1009。在Java 7u40之后Java 8中增加到60013。
- 在JDK7、8中，可以通过-XX:StringTableSize参数设置常量池的StringTable大小。一旦程序启动，这个值就不可改变。
- 如果默认的StringTable大小不能满足我们的需求。-XX:StringTableSize通常我们设置为：（应用中要intern的不同的字符串的个数*2）最接近的质数。这样可以减少碰撞，使得String.intern()所需的时间可以近乎不变，而又不占用太多的heap空间。
- 可以使用-XX:+PrintStringTableStatistics JVM参数设置查看字符串常量池的使用情况。

[参考1](http://tech.meituan.com/in_depth_understanding_string_intern.html)|[参考2](http://www.jianshu.com/p/0d1c003d2ff5)|[参考3](http://java-performance.info/string-intern-in-java-6-7-8/)

## String v.s. StringBuffer v.s. StringBuilder

|特点|String|StringBuffer|StringBuilder|
|---|---|---|---|
|加入时间|JDK1.0|JDK1.0|JDK1.5|
|可变性|不可变|可变|可变|
|是否线程安全|线程安全|线程安全|线程不安全|
|加操作|非常慢|比StringBuilder慢|很快|




