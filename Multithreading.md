# Multi Thread
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
    + 属于`Thread`类，用于中断某个阻塞的线程，换句话说不能中断正在运行的线程
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

当我们希望多个线程不共用某个变量的时候，`ThreadLocal`可以派上用场。譬如为每个request记录一个transaction id或者sequence id。另外，`SimpleFormat`是非线程安全的，所以如果在Servlet环境下，多线程共用的话，以下的写法其实是错误的。很多程序员都会犯的错误！

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

而`ThreadLocal`和线程池一起用的时候，要小心内存泄露的问题，导致`OutOfMemoryError`。[参考1](http://stackoverflow.com/questions/17968803/threadlocal-memory-leak)|[参考2](https://blog.codecentric.de/en/2008/09/a-threadlocal-memory-leak/)|[参考3](https://wiki.apache.org/tomcat/MemoryLeakProtection)，大部分J2EE应用容器都会使用线程池来重用线程，所以我们需要特别小心ThreadLocal，最后需要进行清理：

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

### Lock和ReentrantLock
Java 5在`java.util.concurrent.locks`中增加了Lock接口，Lock的加锁和解锁的语义是等同于synchronized的，只不过Lock可以更细粒度的控制锁。Lock定义了以下的方法：

 - `void lock()`: 加锁，如果当前线程获得锁则立即返回，如果未获得锁则等待，则等待直至获得锁
 - `void lockInterruptibly() throws InterruptedException`：如果当前线程获得锁，则立即返回，如果当前线程未能获得锁则等待，等待过程中可以被另外的线程中断
 - `boolean tryLock()`:当前线程尝试获得锁，如果成功则立刻返回true，如果失败则立刻返回false
 - `boolean tryLock(long time, TimeUnit unit) throws InterruptedException`：在一定的时间内尝试获得锁，并且在这段时间直接可以被打断。如果成功获得，那么将返回 true，否则，返回 false
 - `unlock()`：解锁，Lock不会像 synchronized 那样自动释放锁，所以：一定要放在 try-finally块中，保证锁的释放
```java
try {
    lock.lock();
    ......
} finally {
    lock.unlock();  
}
```
- `Condition newCondition()`
    + Condition 的方法与wait、notify和notifyAll方法类似，分别命名为await、signal和signalAll
    + Lock对象可以有不止一个条件变量与它关联。这样就简化了许多并发算法的开发
    + 相对于wait，notify/notifyAll，因为未知唤醒的是消费者或是生产者，可能想要唤醒生产者，却唤醒的是消费者，则需要将消费者重新阻塞，直至唤醒的是生产者为止，这样白白浪费了资源。而用Condition可以指定多个Condition，明确定义是唤醒的是消费者还是生产者。
下面是用Condition和Lock来实现生产者-消费者模型：
```java
class FIFOBuffer {      
    final Lock lock = new ReentrantLock();     
    final Condition notFull  = lock.newCondition();       
    final Condition notEmpty = lock.newCondition();
    
    final Object[] items = new Object[100];      
    int putptr/*put pointer*/, takeptr/*take pointer*/, count;      
    public void put(Object x) throws InterruptedException {      
        lock.lock();      
        try {      
            while (count == items.length)
                notFull.await();
            items[putptr] = x;
            if (++putptr == items.length) 
                putptr = 0;
            ++count;    
            notEmpty.signal();
        } finally {      
            lock.unlock();      
        }      
    }      

    public Object take() throws InterruptedException {      
        lock.lock();      
        try {      
            while (count == 0)      
                notEmpty.await();
            Object x = items[takeptr];
            if (++takeptr == items.length) 
                takeptr = 0;
            --count;
            notFull.signal();
            return x;      
        } finally {      
            lock.unlock();      
        }      
    }       
} 
```
ReentrantLock是Lock的实现类，它是一个可重入锁。另外synchronized也是可重入锁。

> 什么叫可重入锁：指同一个线程，外层函数获得锁之后，内层递归函数仍有获得该锁的代码，但是不受影响。
可重入锁的最大作用就是可以避免死锁。例如：A线程有两个方法a和b，其中a方法会调用b方法，假如a，b两个方法都需要获得锁，那么首先 a 方法先执行，会获得锁，此时b方法将永远获得不了锁，b方法将一直阻塞住，a方法由于b方法没有执行完，它本身也不释放锁，此时就会造成一个死锁。

ReentrantLock同synchronized的区别总结如下是：

- lock在获取锁的过程可以被中断
- lock可以尝试获取锁，如果锁被其他线程持有，则返回 false，不会使当前线程休眠
- lock在尝试获取锁的时候，传入一个时间参数，如果在这个时间范围内，没有获得锁，那么就是终止请求
- synchronized会自动释放锁，lock则不会自动释放锁
- 可以在不同方法中lock()和unlock()，实现非块结构的加锁解锁
- Condition中的`await()`，`signal()`和`signalAll()`对应Object中的`wait()`，`notify()`和`notifyAll()`
- 可以实现公平的ReentrantLock锁（默认是非公平锁），而synchronized是非公平锁
- 在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时，此时Lock的性能要远远优于synchronized。[参考](http://www.ibm.com/developerworks/cn/java/j-jtp10264/index.html)

### ReadWriteLock和ReentrantReadWriteLock
ReadWriteLock接口和Lock既不是继承关系，也不是实现关系。它实现了如下接口：

```java
public interface ReadWriteLock {  
    Lock readLock();  
    Lock writeLock();  
}
```
它可以实现并行的读，但是串行的读/写，写/读，写/写。主要用于读多于写的情况，可以大大提高并行效率。
- 如果有一个线程已经占用了读锁，则此时其他线程如果也要申请读锁，则其他线程不会被阻塞。
- 如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁。
- 如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会一直等待释放写锁。
ReentrantReadWriteLock是ReadWriteLock的实现类，是读写可重入锁。

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

### BlockingQueue

- 这是个接口，可以线程安全的加入和从队列中取得元素
- 不能加入null
- 实现了以下的方法

|操作|抛出异常|返回值|阻塞|设置超时|
| ---- | ----  | ----  | ---- | ---- |
|增加|add(e)|offer(e)|put(e)|offer(e, time, unit)|
|删除|remove()|poll()|take()|poll(time, unit)|
|检测|element()|peek()|||

![Blocking Queue](https://c1.staticflickr.com/6/5560/30167718120_be3b5f45b2.jpg)

具体的实现类有：
    
- [ArrayBlockingQueue](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ArrayBlockingQueue.html)：是一个基于数组结构的有界阻塞队列，此队列按FIFO(先进先出)原则对元素进行排序
- [LinkedBlockingQueue](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/LinkedBlockingDeque.html)：一个基于链表结构的阻塞队列，此队列按FIFO(先进先出)排序元素，可以定义有界的或无界
- [PriorityBlockingQueue](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/PriorityBlockingQueue.html)：放入其中的元素需要实现Comparable接口
- [SynchronousQueue](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/SynchronousQueue.html)：同步队列，来一个元素就及时处理一个
- [LinkedBlockingDeque](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/LinkedBlockingDeque.html)：队列两头都可以加入也可以取出元素的队列
- [DelayQueue](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/DelayQueue.html)：其中的元素过期最多的首先被取出（头部），元素需要实现Delayed接口，如果没有元素过期，则poll返回null

### Synchronizer
### AbstractQueuedSynchronizer(AQS)
AbstractQueuedSynchronizer是`java.util.concurrent`的核心，其他工具类（如ReentrantLock，ReentrantReadWriteLock, CountDownLatch，Semaphore）都要依赖它。
深入解读AbstractQueuedSynchronizer，参考：

- [深度解析Java 8：AbstractQueuedSynchronizer的实现分析(上)](http://www.infoq.com/cn/articles/jdk1.8-abstractqueuedsynchronizer)
- [深度解析Java 8：AbstractQueuedSynchronizer的实现分析(下)](http://www.infoq.com/cn/articles/java8-abstractqueuedsynchronizer)
- [深入浅出java同步器](http://www.jianshu.com/p/d8eeb31bee5c)
- [AbstractQueuedSynchronizer的介绍和原理分析](http://ifeve.com/introduce-abstractqueuedsynchronizer/)

### CountDownLatch
CountDownLatch可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。
下面例子waiter等待decrementer线程执行一次countdown。CountDownLatch初始化时需要声明要countdown几次，每countdown一次，count减1，直到减到0，waiter才会在await()从阻塞状态重新恢复运行。

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
	
	public static void main(String args[]) throws Exception {
		
		CountDownLatch countDownLatch = new CountDownLatch(1);
		
		Thread waiter = new Waiter(countDownLatch);
		
		Thread decrementer = new Decrementer(countDownLatch);
		
		waiter.start();
		decrementer.start();
	}
	
	static class Waiter extends Thread {
		
		private CountDownLatch countDownLatch;
		
		Waiter(CountDownLatch countDownLatch) {
			this.countDownLatch = countDownLatch;
		}
		
		@Override
		public void run() {
			System.out.println("Create wait Thread");
			try{
				this.countDownLatch.await();
			} catch(InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("End wait Thread");
		}
	}
	
	static class Decrementer extends Thread {
		
		private CountDownLatch countDownLatch;
		
		Decrementer(CountDownLatch countDownLatch) {
			this.countDownLatch = countDownLatch;
		}
		
		@Override
		public void run() {
			System.out.println("Created Decrementer Thread");
			try{
				Thread.sleep(1000);
				this.countDownLatch.countDown();
				System.out.println("count down once");
			} catch(InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("End Decrementer Thread");
		}
	}
}
```

### CyclicBarrier

通过CyclicBarrier可以实现让一组线程等待至某个状态之后再全部同时执行。当调用await()方法之后，线程就处于barrier了，而当所有线程都达到barrier之后，就会全部继续执行。

![CyclicBarrier](https://c1.staticflickr.com/9/8416/30306524302_215a5e92a1_o.png)

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierExample {
	
	public static void main(String args[]) throws Exception {
		
		Runnable barrier1Action = new Runnable() {
		    public void run() {
		        System.out.println("BarrierAction 1 executed ");
		    }
		};
		Runnable barrier2Action = new Runnable() {
		    public void run() {
		        System.out.println("BarrierAction 2 executed ");
		    }
		};

		CyclicBarrier barrier1 = new CyclicBarrier(2, barrier1Action);
		CyclicBarrier barrier2 = new CyclicBarrier(2, barrier2Action);

		CyclicBarrierRunnable barrierRunnable1 = new CyclicBarrierRunnable(barrier1, barrier2);

		CyclicBarrierRunnable barrierRunnable2 = new CyclicBarrierRunnable(barrier1, barrier2);

		new Thread(barrierRunnable1).start();
		new Thread(barrierRunnable2).start();
	}
	
	static class CyclicBarrierRunnable implements Runnable{

	    CyclicBarrier barrier1 = null;
	    CyclicBarrier barrier2 = null;

	    public CyclicBarrierRunnable(CyclicBarrier barrier1, CyclicBarrier barrier2) {

	        this.barrier1 = barrier1;
	        this.barrier2 = barrier2;
	    }

	    public void run() {
	        try {
	            Thread.sleep(1000);
	            System.out.println(Thread.currentThread().getName() + " waiting at barrier 1");
	            this.barrier1.await();

	            Thread.sleep(1000);
	            System.out.println(Thread.currentThread().getName() + " waiting at barrier 2");
	            this.barrier2.await();

	            System.out.println(Thread.currentThread().getName() + " done!");

	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        } catch (BrokenBarrierException e) {
	            e.printStackTrace();
	        }
	    }
	}
}
```
和CountDownLatch不同，CyclicBarrier可以被重复使用，从字面Cyclic（循环）就可以看出来。

### Semaphore

Semaphore可以控制同时访问的线程个数，Semaphore初始化需要传入"permits"个数，指明仅能有"permits"个的线程能访问某一个资源。线程通过 acquire() 获取一个许可，如果没有许可就等待，而 release() 释放一个许可。

```java
import java.util.concurrent.Semaphore;

public class SemaphoreExample {
	
    public static void main(String[] args) {
        
    	final int WORKER_NUM = 8;
    	final int MACHINE_NUM = 5;
        Semaphore semaphore = new Semaphore(MACHINE_NUM);
        for(int i = 0; i < WORKER_NUM; i++)
            new Worker(i,semaphore).start();
    }
     
    static class Worker extends Thread{
        private int num;
        private Semaphore semaphore;
        public Worker(int num,Semaphore semaphore){
            this.num = num;
            this.semaphore = semaphore;
        }
         
        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("worker " + this.num + " aquires the machine");
                Thread.sleep(2000);
                System.out.println("worker " + this.num + " releases the machine");
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
另外有一个fair参数可以控制是否可以公平的获得许可。
```java
public Semaphore(int permits, boolean fair)
```
倘若fair为true，则先阻塞等待许可的线程将首先获得许可。默认是不公平的，如果设置为公平的Semaphore，则会牺牲性能，所以非必要情况不要设置fair为true。

### 线程池

所谓线程池，那么就是相当于有一个池子，线程就放在这个池子中进行重复利用，能够减去了线程的创建和销毁所带来的代价。
线程池比较重要的几个关键类和接口：

- Executors
- ExecutorService
- ThreadPoolExecutor
- ScheduledExecutorService
- ScheduledThreadPoolExecutor

#### Executors

这是个工厂类，提供很多静态工厂方法用来创建ExecutorService实例，常用的方法有：

```java
ExecutorService executorService1 = Executors.newSingleThreadExecutor();

ExecutorService executorService2 = Executors.newFixedThreadPool(10);

ExecutorService executorService3 = Executors.newCachedThreadPool();

ExecutorService executorService5 = Executors.newSingleThreadScheduledExecutor();

ExecutorService executorService4 = Executors.newScheduledThreadPool(10);

```
创建的线程池有什么特性要结合ThreadPoolExecutor和ScheduledThreadPoolExecutor来理解。

#### ExecutorService

```java
public interface ExecutorService extends Executor {

    Future<?> submit(Runnable task);
    
    <T> Future<T> submit(Callable<T> task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
    
    void shutdown();
    
    List<Runnable> shutdownNow();
    ...
}
```

- `void execute(Runnable command)`：继承自Executor接口，无法返回执行结果，也不知道是否执行成功
- `Future<?> submit(Runnable task)`：无法返回执行结果，但可以知道是否执行成功，future.get()返回null时则表示任务执行成功
- `<T> Future<T> submit(Callable<T> task)`:这种用法可以获得执行结果，future.get()可以得到执行结果。
- `<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException`: 把任务放进集合中，进行批量的执行，并且可以有返回值
- `void shutdown()`: 当调用这个方法，线程池不会立即停止，它不再会接受新的任务，当已经提交的任务（不管是否正在运行还是等待）运行完毕，这个线程池会关闭
- `List<Runnable> shutdownNow()`: 调用它时，会不再接受新的提交任务，也不会再执行已提交并正在等待的任务，而正在执行中的任务也会尝试中断，但并不能保证一定能顺利中断，并返回尚未执行的任务

#### ThreadPoolExecutor

ThreadPoolExecutor实现了ExecutorService。创建线程池最常用的是使用ThreadPoolExecutor，实际上Executors里的工厂方法中很多都是通过创建ThreadPoolExecutor实例来实现线程池。

```java
new ThreadPoolExecutor(
            int corePoolSize, 
            int maximumPoolSize, 
            long keepAliveTime, 
            TimeUnit unit, 
            BlockingQueue<Runnable> workQueue, 
            ThreadFactory threadFactory,
            RejectedExecutionHandler handler);
```

- corePoolSize：线程池的基本大小，当提交一个任务到线程池时，线程池会创建一个线程来执行任务（即使有空闲线程也会创建），等到需要执行的任务数大于线程池基本大小时就不再创建。
- workQueue：任务队列，用于保存等待执行的任务的阻塞队列。 可以选择以下几个阻塞队列。
    + ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。
    + LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
    + SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool()使用了这个队列。

- maximumPoolSize：线程池最大大小，线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。
- threadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。
- handler：饱和策略，当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。以下是JDK1.5提供的四种策略。
    + AbortPolicy：丢弃任务并抛出RejectedExecutionException异常
    + DiscardPolicy：也是丢弃任务，但是不抛出异常
    + CallerRunsPolicy：由调用线程处理该任务
    + DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
    
当然也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。
- keepAliveTime：线程活动保持时间，线程池的工作线程空闲后，保持存活的时间。
- TimeUnit：线程活动保持时间的单位。

![ThreadPoolExecutor创建流程](https://c7.staticflickr.com/6/5680/30433240462_45b08d547b_o.jpg)

- 首先线程池判断基本线程池（corePoolSize）是否已满？没满，创建一个工作线程来执行任务。满了，则进入下个流程。
- 其次线程池判断工作队列(workQueue)是否已满？没满，则将新提交的任务存储在工作队列里。满了，则进入下个流程。
- 最后线程池判断整个线程池(maximumPoolSize)是否已满？没满，则创建一个新的工作线程来执行任务，满了，则交给饱和策略来处理这个任务。
- 如果线程空闲时间超过keepAliveTime，线程会被终止。

#### ScheduledExecutorService

ScheduledExecutorService继承了ExecutorService，它可以实现延迟执行任务和执行周期性任务。

```java
public interface ScheduledExecutorService extends ExecutorService {
    public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
    
    public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);
	
	 public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit);
	 
	 public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit);
}
```

- `ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit)`:延迟执行任务，当任务完成时，ScheduledFuture.get()返回null
- `ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit)`: 延迟执行任务，并且可以返回ScheduledFuture，ScheduledFuture可以用来在任务开始前结束任务，或者返回任务的结果
- `ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)`:延迟initialDelay开始执行任务，并且每隔period就周期性执行一次任务，如果任务执行的时间大于period，则等到任务执行完毕再开始执行任务
- `ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)`:延迟initialDelay开始执行任务，period周期不同于scheduleAtFixedRate中的period，它是上一次任务结束到下一次任务开始为一个period

### ScheduledThreadPoolExecutor
    
ScheduledThreadPoolExecutor继承了ThreadPoolExecutor。用来实现延迟任务线程池，或周期任务线程池。
其他参数都和ThreadPoolExecutor一样，唯一的区别就是ScheduledThreadPoolExecutor使用的workQueue是DelayedWorkQueue，是一个基于Delay的队列，最先取出的是过期最多的队列，也就可以保证实现延时任务和周期任务。

## Atomic Variable(原子变量)

在`java.util.concurrent.atomic`包下有许多原子变量类，它们在多线程的环境下是安全的。
例如`AtomicInteger`是一个支持原子操作的Integer类，就是保证对AtomicInteger类型变量的增加和减少操作是原子性的，不会出现多个线程下的数据不一致问题。
而原子类的基础就是CAS(Compare And Swap)技术。CAS是一种乐观锁技术。现在的CPU都直接支持CAS指令。当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。

> **悲观锁与乐观锁**
前面的synchronized和Lock都是**悲观锁**，即在某个资源不可用的时候，就将CPU让出，把当前等待线程切换为阻塞状态。等到资源(比如一个共享数据）可用了，那么就将线程唤醒，让他进入Runnable状态等待CPU调度。
而当任务执行时间很短的时候，悲观锁就显得很耗费资源，因为挂起和恢复存在着很大的开销。
所以就有了**乐观锁**的概念，他的核心思路就是，每次不加锁(lock-free)而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。

截取一小段AtomicInteger的源代码：

```java
public class AtomicInteger extends Number implements java.io.Serializable {

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();

    public final int incrementAndGet() {  
        for (;;) {  
            int current = get();  
            int next = current + 1;  
            if (compareAndSet(current, next))  
                return next;  
        }
    }
    
    public final boolean compareAndSet(int expect, int update) {
    	return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
        ...
}
```

`compareAndSet`所做的为调用Sun的UnSafe的`compareAndSwapInt`方法来完成，此方法为JNI(Java Native Interface)方法，借助C来调用CPU底层指令实现的。所以基于CAS的操作是无锁的(lock-free)，不会阻塞或挂起线程，由于CAS操作是CPU原语，所以性能比较好。

使用CAS需要注意ABA问题，更多[参考1](https://www.ibm.com/developerworks/cn/java/j-jtp11234/)|[参考2](http://ifeve.com/atomic-operation/)

### concurrent包总结

如果我们仔细分析concurrent包的源代码实现，会发现一个通用化的实现模式，volatile和CAS是基础：

 1. 首先，声明共享变量为volatile； 
 2. 然后，使用CAS的原子条件更新来实现线程之间的同步；
 3. 同时，配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的通信。

AQS(AbstractQueuedSynchronizer)和原子变量类（java.util.concurrent.atomic包中的类），这些concurrent包中的基础类都是使用这种模式来实现的，而concurrent包中的高层类又是依赖于这些基础类来实现的。

整个`java.util.concurrent`包可以用一幅图来总结:

![java.util.concurrent层次图](https://c2.staticflickr.com/6/5511/30523836201_2b2ca13da9_o.png)

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
- http://ifeve.com/



