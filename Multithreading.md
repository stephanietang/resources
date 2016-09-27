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

----------

## Java的内存模型(JMM, Java Memory Model)

Java的内存模型。


----------


## volatile关键字


----------


## synchronized关键字用法
synchronized用来实现互斥。

> 互斥：是指某一资源同时只允许一个访问者对其进行访问，具有唯一性和排它性。但互斥无法限制访问者对资源的访问顺序，即访问是无序的。

有两种锁：

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

## Lock