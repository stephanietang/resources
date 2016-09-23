## extend Thread

```java

public class ThreadExample {

	public static void main(String args[]) throws Exception {
	
		Thread t1 = new MyThread("aaa");
		
		t1.start();
		
		Thread t2 = new MyThread("bbb");
		
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

## implement Runnable

- Runnable的写法麻烦些，但是鉴于只能单继承，但可以多接口，一般推荐使用implement Runnable的方式

```java

public class RunnableExample {
	
	public static void main(String args[]) throws Exception {
	
		Thread t1 = new Thread(new MyRunnable("thread1"));
		
		t1.start();
		
		Thread t2 = new Thread(new MyRunnable("thread2"));
		
		t2.start();
	}
	
	static class MyRunnable implements Runnable {
		
		private String threadName;
		
		MyRunnable(String threadName) {
			this.threadName = threadName;
		}
		
		@Override
		public void run() {
			System.out.println("Created Thread " + threadName);
		}
	}
}

```


## Callable和FutureTask

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

## ExecutorService

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