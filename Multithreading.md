# 多线程
标签：Java
----------
## 实现线程的几种写法
### 继承Thread

```java

public class ThreadExample {

	public static void main(String args[]) throws Exception {
	
		Thread t1 = new MyThread("threadA");
		
		t1.start();
		
		Thread t2 = new MyThread("threadB");
		
		t2.start();
	}
	
	static class MyThread extends Thread {
		
		MyThread(String threadName) {
			super(threadName);
		}
		
		@Override
		public void run() {
			System.out.println("Created Thread " + getName());
		}
	}
}

```

### 实现Runnable接口

- Runnable的写法麻烦些，但是鉴于只能单继承，但可以多接口，一般推荐使用implement Runnable的方式

```java

public class RunnableExample {
	
	public static void main(String args[]) throws Exception {
	
		Thread t1 = new Thread(new MyRunnable(), "threadA");
		
		t1.start();
		
		Thread t2 = new Thread(new MyRunnable(), "threadB");
		
		t2.start();
	}
	
	static class MyRunnable implements Runnable {
		
		
		@Override
		public void run() {
			System.out.println("Created Thread " + Thread.currentThread().getName());
		}
	}
}

```


### Callable和FutureTask

- 如果线程不需要有返回值，则只需实现Runnable接口
- 如果线程需要有返回值，则需要实现Callable接口，返回FutureTask对象

```java

import java.util.Random;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class CallableAndFuture {
	
    public static void main(String[] args) {
	
        Callable<Integer> task = new MyCallable();
        
        FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
        
        Thread t = new Thread(futureTask);
		
        t.start();
        
        try {
            Thread.sleep(1000);
            System.out.println(futureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
    
    static class MyCallable implements Callable<Integer> {
    	
    	@Override
    	public Integer call() throws Exception {
    		return new Random().nextInt(100);
    	}
    }
}

```

----------
## 线程状态图

![线程状态图](https://c2.staticflickr.com/6/5705/30294237975_742128d2a3_o.png)

几种比较重要的方法：

- `wait()`
    
    + `wait()`和`wait(timeout)`是属于`Object`类，只有拥有锁的线程才能调用`wait`方法，也就是`wait()`必须包含在同步块中，否则运行时会抛出`IllegalMonitorStateException`异常。
    + `wait`会立即释放锁，进入`Waiting`或者`Timed Waiting`状态，直到其他线程调用该锁对象的`notify`或者`notifyAll`或者到了指定时间，才会重新进入`Runnable`状态，等待再次被CPU调用。

```java
Object lock = new Object();
try{
    synchronized (lock){
        lock.wait();
    }
} catch (InterruptedException e) {
    e.printStackTrace();
}
```
 - `notify()`和`notifyAll()`
    + `notify()`和`notifyAll()`是属于`Object`类，和`wait`一样，必须包含在同步块中，否则也会抛出`IllegalMonitorStateException`异常。
    + `notify`可以随机唤醒等待状态中等待同一个对象锁的一个线程，并使得进入Runnable状态
    + `notifyAll`可以唤醒所有正在等待状态中等待同一个对象锁的全部线程，让它们进入Runnable状态 
    + 和`wait`不同，`notify`不会立即释放锁，而会运行完锁对象同步的代码之后，才会释放锁
    + 因为`notify`只随机唤醒一个线程，如果不止一个线程等待某个对象的锁，而只唤醒了一个线程，那么其他没被唤醒的线程就会无限制的等待下去。作为一个好的编程实践，任何情况下，我们都应该用`notifyAll`，而避免使用`notify`
 - `Thread.sleep()`
    + 属于`Thread`类
    + 区别于`wait`, `sleep`不会释放锁，到了指定时间，再重新运行
    + static方法，直接使用`Thread.sleep(timeout)`使当前线程暂停
 - `interrupt()`
    + 属于`Thread`类，用于中断某个线程
    + `wait`或`join`或`sleep`被中断，会抛出`InterruptedException`
```java
MyThread t = new MyThread();
t.start();
t.interrupt();
```
 - `join()`
    + `join()`和`join(timeout)`属于`Thread`类
    + 可以被中断，抛出`InterruptedException`异常

```java
Thread t = new Thread(new ThreadDemo());
t.start();
try{
    t.join(); // 等到线程t执行完毕，main线程才继续执行
} catch (InterruptedException e) {
    e.printStackTrace();
}
```
 - `Thread.yield()`
    + 当前线程停止一段时间，但是Java并没有对此方法有明确的定义，Linux和Windows以及不同Java版本对此方法的实现细节不同，换句话说这个方法并不可靠，在实际应用中永远不要用这个方法
    + [参考](http://www.javamex.com/tutorials/threads/yield.shtml)
 - `setPriority()` / `getPriority()`
    + `Thread`中的方法
    + 实际应用中，不要依赖这个方法，因为它们依赖于Java的版本和OS
 - 我们仅需要了解这些方法的原理，而实际的大型应用，**不要**用这些很底层的方法，而应该用其他成熟的工具和技术，如`java.util.concurrent`下的工具代替

----------
## 线程安全

并发有两个非常重要的特性：

 - 原子性(atomicity)：指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其它线程干扰。

    Java中的原子操作包括：

    - 除long和double之外的基本类型的赋值操作
    - 所有引用reference的赋值操作
    - java.concurrent.atomic.* 包中所有类的一切操作
    - 声明了volatile的long和double变量赋值操作

> 对long和double的赋值操作是非原子操作。long和double占用的字节数都是8，也就是64bits。在32位操作系统上对64位的数据的读写要分两步完成，每一步取32位数据。这样对double和long的赋值操作就会有问题：如果有两个线程同时写一个变量内存，一个进程写低32位，而另一个写高32位，这样将导致获取的64位数据是失效的数据。因此需要使用volatile关键字来防止此类现象。volatile本身不保证获取和设置操作的原子性，仅仅保持修改的可见性。但是java的内存模型保证声明为volatile的long和double变量的get和set操作是原子的。

- 可见性(visibility)：当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

    - 线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中。
    - 线程获取锁时，JMM会把该线程对应的本地内存置为无效，从而使得被监视器保护的临界区代码必须从主内存中读取共享变量。

----------

## volatile关键字

Java内存模型规定和指引Java程序在不同的内存架构、CPU和操作系统间有确定性的行为。
在Java内存模型中，所有线程共用主存(main memory)，并在工作内存(working memory: L1, L2, L3高速缓存)中保存变量的副本，直接从工作内存中读写变量，效率会快很多。但如果线程不及时将工作内存中的变量刷进主存中，线程不能及时看到另一个线程的对变量的更新，则会出现意想不到的结果。

![Java内存模型](https://c3.staticflickr.com/8/7558/30155423522_d9a56d134a.jpg)

声明volatile保证了可见性，但损失了效率。具体如何实现可见性，请参考：http://ifeve.com/volatile/
但volatile只能保证可见性，并不能保证原子性。所以对于i++这种非原子操作，仅仅使用volatile是不够的，若要保证非原子操作的线程安全，则还需要synchroniezed或者Lock。

使用volatile需要注意的地方：
- 运算结果并不依赖于当前值，或者能确保只有单一的线程能够修改变量的值。
- 变量不需要和其他的状态变量共同参与不变约束
volatile经典的用法是声明一个boolean变量作为状态变量，用于指示发生了一个重要的一次性事件，例如完成初始化或请求停机。

```java
volatile boolean shutdownRequested;

...

public void shutdown() { shutdownRequested = true; }

public void doWork() { 
    while (!shutdownRequested) { 
        // do stuff
    }
}
```
更多的经典的volatile用法参见：
http://www.ibm.com/developerworks/cn/java/j-jtp06197.html

----------

## synchronized关键字用法

syncrhonized既可以满足可见性，又可以满足原子性。
syncrhonized机制有两种锁：

 - 对象锁：每个对象都有一个对象锁
 - 类锁：每个类有一个类锁

### 修饰一个代码块

修饰一个代码块，是对象锁，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象。 

```java
class ClassName {
    private byte[] lock = new byte[0];
    
    // 对lock进行同步
    public void method1() {
        synchronized (lock) {
            // todo
        }
    }
    
    // 对ClassName类的实例对象进行同步
    public void method2() {
        synchronized (this) {
            // todo
        }
    }
}
```

### 修饰一个方法

修饰一个方法，是对象锁，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象。 

```java
class ClassName {
    public synchronized void instanceMethod() {
       // todo
    }
}
```

### 修饰一个静态方法

类锁

```java
class ClassName {
    public synchronized static void staticMethod() {
       // todo
    }
}
```

### 修饰一个类

类锁

```java
class ClassName {
    public void method() {
       synchronized(ClassName.Class) {
            // todo
       }
    }
}
```
关于对象锁和类锁，需要注意以下几点：

 - 只有synchronized修饰的代码块/方法/类才存在锁，也就是说，并不会阻塞非synchronized的代码块或者方法的执行
 - 同一对象的对象锁才能互斥
 - 不同对象的对象锁之间不存在互斥
 - 同一个类的类锁才能互斥
 - 不同类的类锁之间不存在互斥
 - 对于同一个类A，线程1争夺A对象实例的对象锁，线程2争夺类A的类锁，这两者不存在竞争关系。也就说对象锁和类锁互不干预内政。


----------
## ThreadLocal

当我们希望多个线程不共用某个变量的时候，`ThreadLocal`可以派上用场。譬如`SimpleFormat`是非线程安全的，所以如果在Servlet环境下，多线程共用的话，以下的写法其实是错误的。很多程序员都会犯的错误！

```java
private static final DateFormat FORMAT = new SimpleDateFormat("yyyy-MM-dd");

...

public Date getDate(String str) throws ParseException {
    Date date = dateFORMAT.parse(str);
    return date;
}

```
要验证这个错误，我们来写一个多线程环境下的程序：

```java
import java.text.DateFormat;
import java.text.SimpleDateFormat;

public class SimpleDateFormatNotThreadSafe {
	
	private static final DateFormat FORMAT = new SimpleDateFormat("yyyy-MM-dd");
	
	public static void main(String args[]) throws Exception {
		
		for(int i = 0 ; i < 10; i++) {
			Thread t = new MyThread("thread "+i);
			t.start();
		}
		
	}
	
	static class MyThread extends Thread {
		
		MyThread(String threadName) {
			super(threadName);
		}
		
		@Override
		public void run() {
			System.out.println("Created Thread " + getName());
			try{
				System.out.println(FORMAT.parse("2016-01-01"));
			} catch( Exception e){
				e.printStackTrace();
			}
		}
	}

}

```

以上程序有可能会抛出异常，因为`SimpleDateFormat`并不是线程安全的（这其实是`SimpleDateFormat`设计者的错误，`SimpleDateFormat`应该设计成不可变类）。然而`SimpleDateFormat`对象的创建是很昂贵的，所以我们可以用`ThreadLocal`来解决这个问题：

```java
import java.text.DateFormat;
import java.text.SimpleDateFormat;

public class ThreadLocalThreadSafe {

	private static final ThreadLocal<DateFormat> FORMAT = new ThreadLocal<DateFormat>(){
		
		@Override 
		protected DateFormat initialValue(){
			return new SimpleDateFormat("yyyy-MM-dd");
		}
	};
	
	public static void main(String args[]) throws Exception {
		
		for(int i = 0 ; i < 10; i++) {
			Thread t = new MyThread("thread "+i);
			t.start();
		}
		
	}
	
	static class MyThread extends Thread {
		
		MyThread(String threadName) {
			super(threadName);
		}
		
		@Override
		public void run() {
			System.out.println("Created Thread " + getName());
			try{
				System.out.println(FORMAT.get().parse("2016-01-01"));
			} catch( Exception e){
				e.printStackTrace();
			}
		}
	}

}
```
`ThreadLocal`可以为每个线程创建独立的“本地”变量，这样线程之间并不会共用，也就不会有线程安全问题。这是不想写不可变类，又不想进行同步时的一种选择。
当然针对DateFormat，我们有更好的选择：

- apache commons中的[FastDateFormat](https://commons.apache.org/proper/commons-lang/javadocs/api-2.6/org/apache/commons/lang/time/FastDateFormat.html)
- joda-time中的[DateTimeFormat](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)
- Java 8中的[DateTimeFormatter](http://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html)

而`ThreadLocal`和线程池一起用的时候，要小心内存泄露的问题，[参考1](http://cs.oswego.edu/pipermail/concurrency-interest/2007-October/004456.html)|[参考2](http://stackoverflow.com/questions/17968803/threadlocal-memory-leak)|[参考3](http://javarevisited.blogspot.hk/2012/03/simpledateformat-in-java-is-not-thread.html)|[参考4](http://java.jiderhamn.se/2012/01/29/classloader-leaks-iv-threadlocal-dangers-and-why-threadglobal-may-have-been-a-more-appropriate-name/)，大部分J2EE应用容器都会使用线程池来重用线程，所以我们需要特别小心ThreadLocal，最后需要进行清理：

```java
try {
    threadLocal.set(value);
    ...
}
finally {
    threadLocal.remove();
}
```

----------

## 同步集合（Synchronized Collection）

我们知道ArrayList, HashMap, HashSet都是非线程安全的。多线程环境下，没有同步的情况下不能直接使用ArrayList, HashMap, HashSet。
Java JDK中已经有一些同步集合，用来保证我们的线程安全。

- Vector和Hashtable
    + 多线程环境，[Vector](http://docs.oracle.com/javase/6/docs/api/java/util/Vector.html)代替ArrayList
    + 多线程环境，[Hashtable](http://docs.oracle.com/javase/6/docs/api/java/util/Hashtable.html)代替HashMap
    + 但实际Vector和Hashtable性能很差，已经很少用了，多线程并发中，我们通常用并发集合`CopyOnWriteArrayList`和`ConcurrentHashMap`代替
    + 如果数据量大的情况下，单线程下，Vector反而比ArrayList的效率高（可以写个简单的程序测试），这是因为Vector的扩容方式和ArrayList不同，相对于扩容，加锁解锁的开销反而可以忽略
- Collections中的静态方法
    + 多线程环境，[Colloctions.synchronizedList(List<T> list)](http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#synchronizedList(java.util.List))代替ArrayList
    + 多线程环境，[Colloctions.synchronizedMap(Map<K,V> m)](http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#synchronizedMap(java.util.Map))用来代替HashMap

但使用同步集合，线程安全并不是万无一失的，还是需要注意同步问题。
Vector仅能保证同一时刻只能有一个线程访问Vector，但并不能保证多步操作的原子性，譬如vector.size()和vector.remove(i)之间可能已经被另外的线程修改了vector。下面的做法才是正确的做法：
```java
Vector vector = new Vector();
...
synchronized (vector) {
    for(int i = 0; i < vector.size(); i++)
        vector.remove(i);
}
```
而使用Collections.synchronizedList时，需要进行iterator遍历时，也需要同步：
```java
List list = Collections.synchronizedList(new ArrayList());
...
synchronized(list) {
     Iterator i = list.iterator(); // Must be in synchronized block
    while (i.hasNext())
        foo(i.next());
}
```

----------
## java.util.concurrent常用工具

`java.util.concurrent`提供了许多常用工具。在实际应用中，我们应该更多的使用它们，而不是再自己重复造轮子。

### Lock
http://www.ibm.com/developerworks/cn/java/j-jtp10264/index.html
Java 1.5在`java.util.concurrent.locks`中增加了Lock接口，定义了以下的方法：

 - lock() 
 - lockInterruptibly() 
 - tryLock() 
 - tryLock(long timeout, TimeUnit timeUnit)
 - unlock()

### ReentrantLock
ReentrantLock是Lock的实现类。
典型用法：
```java
class X {
    private final ReentrantLock lock = new ReentrantLock();
    // ...

    public void m() {
        lock.lock();  // block until condition holds
        try {
            // ... method body
        } finally {
            lock.unlock()
        }
    }
}
```
它具有和synchronized相同的内存语义、相同的锁定，但具有更好的性能，以及其他的用途和优点：

- 定时的锁等待
- 可中断的锁等待
- 公平性
- 实现非块结构的加锁
- Condition中的`await()`，`signal()`和`signalAll()`对应Object中的`wait()`，`notify()`和`notifyAll()`

### ReadWriteLock

### 并发集合（Concurrency Collection）
由于同步集合对所有的访问无论读写都进行了同步，大大降低了并发效率。Java 5在java.util.concurrent中加入了并发集合，改善了效率：

- ConcurrentHashMap
    + ConcurrentHashMap采用了分段锁(segment)的设计，读时并没有同步，写时也不是对整个map同步，只是对segment同步，因此大大提高了同步性
- CopyOnWriteArrayList
    + Copy-On-Write(COW)的策略是：读的时候不加锁，写的时候(add, remove)的时候先将集合复制，然后修改，修改完毕，再将引用指向新的集合
    + 但要注意占用空间的问题，因为每次更改都会复制，如果list太大，则会造成频繁的GC，这时可能不用Copy-On-Write，而使用其他集合，如ConcurrentHashMap
```java
public E get(int index) {
    return (E)(getArray()[index]);
}

public boolean add(E e) {
	final ReentrantLock lock = this.lock;
	lock.lock();
	try {
	    Object[] elements = getArray();
	    int len = elements.length;
	    Object[] newElements = Arrays.copyOf(elements, len + 1);
	    newElements[len] = e;
	    setArray(newElements);
	    return true;
	} finally {
	    lock.unlock();
	}
}
```
- CopyOnWriteArraySet
    + CopyOnWriteArraySet里面包含一个CopyOnWriteArrayList，其实是对CopyOnWriteArrayList包装类，因此CopyOnWriteArrayList也适用于CopyOnWriteArraySet

### Synchronizer
### AbstractQueuedSynchronizer(AQS)

### Semaphore

### CountDownLatch

### CyclicBarrier

### Exchanger

### ExecutorService

```java

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class ExecutorServiceExample {

    private static final int THREAD_NUM = 10;

    public static void main(String args[]) throws Exception {

        ExecutorService executor = Executors.newFixedThreadPool(THREAD_NUM);
		
		List<Future<Integer>> futures = new ArrayList<Future<Integer>>();

        for(int i = 0; i < THREAD_NUM; i++) {
		
            MyCallable task = new MyCallable();
			
			Future<Integer> future = executor.submit(task);
			
			futures.add(future);
        }
		
        executor.shutdown();
		
		try {
            Thread.sleep(1000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
		
		try {
			for(Future<Integer> future : futures) {
				System.out.println("Result "+ future.get());
			}
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    static class MyCallable implements Callable<Integer> {
    	
    	@Override
    	public Integer call() throws Exception {
    		return new Random().nextInt(100);
    	}
    }
    
}
```
### ForkJoinPool

Java 7又增加了ForkJoinPool。它类似ExecutorService

## 死锁（Deadlock）
死锁简单来说就是线程A拥有锁a，而需要线程B的锁b，而同时线程B拥有锁b，同时又需要锁a，则会发生死锁。
死锁是多线程编程应该注意的问题，高并发环境下，一旦死锁，则会造成程序无法执行甚至崩溃，只能重启应用了。
下面是一个简单的模拟死锁的程序：
```java
public class DeadLock {
	
	private final Object lockA = new Object();
    private final Object lockB = new Object();
    
    public void methodA() throws Exception {
        synchronized (lockA) {
            Thread.sleep(2000);
            synchronized (lockB) {
                System.out.println("methodA end!");
            }
        }
    }
    
    public void methodB() throws Exception{
        synchronized (lockB){
            Thread.sleep(2000);
            synchronized (lockA) {
                System.out.println("methodB end!");
            }
        }
    }

	public static void main(String[] args) {
	    DeadLock dl = new DeadLock();
	    ThreadA ta = new ThreadA(dl);
	    ThreadB tb = new ThreadB(dl);
	    ta.start();
	    tb.start();
	}
	
	static class ThreadA extends Thread {
	    private DeadLock dl;
	    
	    public ThreadA(DeadLock dl){
	        this.dl = dl;
	    }
	    
	    @Override
	    public void run(){
	        try {
	            dl.methodA();
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	}
	
	static class ThreadB extends Thread {
	    private DeadLock dl;
	    
	    public ThreadB(DeadLock dl) {
	        this.dl = dl;
	    }
	    
	    @Override
	    public void run() {
	        try {
	            dl.methodB();
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	}
}
```
### 死锁的排查工具
jps + jstack

- 获得JVM正在运行的pid: `jps`或者`ps -ef | grep java`
- 打印线程堆栈(thread dump): `jstack pid` 
- 查看CPU占用：`top -H -p pid`

![jstack](https://c6.staticflickr.com/9/8130/30185424581_ff7272843b_b.jpg)

### 如何避免死锁

- 不用多线程
- 同步的代码块越小越好，运行时间越短越好，将费时不需要同步的操作（如IO操作）移出同步代码
- 不要一次用多个锁，如果必须要使用多个锁的时候，保证每次调用的次序一样
- 不要在同步块中调用其他的函数
- 用可以中断的锁，譬如用设置了等待时间的ReentrantLock代替synchronized


参考：
http://www.slideshare.net/caroljmcdonald/java-concurrency-memory-model-and-trends-4961797

关于锁的最佳实践
http://www.ibm.com/developerworks/cn/java/j-lo-lock/index.html

锁分类：
http://luojinping.com/2015/07/09/java%E9%94%81%E4%BC%98%E5%8C%96/

### 关于Spring MVC和Struts线程安全问题
- Spring Controller默认是Singleton，不同request虽然在不同的线程中，但是会使用同一个Controller，所以Controller必须要考虑线程安全问题，不能有可以更改的成员变量。
如下代码就会有线程安全问题：
```java
@Controller
public class MyController {
    private int count = 0;
    @RequestMapping(value = "/index")
    public String respond() {
        count++;
        return "index";
    }
}
```
要解决线程安全问题，开发时需要特别注意，要么不使用可变成员变量，要么就对变量进行同步，要么成员变量是stateless的（譬如service, repository）。
- Struts 1 Action和Spring Controller一样，是Singleton，必须线程安全，不能在Action中使用成员变量，需要将它们封装在Form中
- Struts 2 Action，每个request都会new一个Action对象，不需要考虑线程安全问题，可以在Action中使用成员变量

### 多线程面试题

- [Java线程面试题 Top 50](http://www.importnew.com/12773.html)
- [JAVA多线程和并发基础面试问答](http://ifeve.com/java-multi-threading-concurrency-interview-questions-with-answers/)
- [40个Java多线程问题总结](http://www.cnblogs.com/xrq730/p/5060921.html)

### 参考：

- http://tutorials.jenkov.com/java-util-concurrent/index.html
- https://www.ibm.com/developerworks/cn/views/java/libraryview.jsp?search_by=Java+%E5%B9%B6%E5%8F%91%E6%80%A7
- http://www.cnblogs.com/dolphin0520/category/602384.html
- http://www.cnblogs.com/xrq730/category/733883.html
- https://github.com/pzxwhc/MineKnowContainer
- http://www.javamex.com/tutorials/threads/



