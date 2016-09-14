## ExecutorService

~~~~

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ExecutorServiceExample {

    private static final int THREAD_NUM = 30;

    public static void main(String args[]) throws Exception {

        ExecutorService executor = Executors.newFixedThreadPool( THREAD_NUM );

        for(int i = 0; i < THREAD_NUM; i++) {
            Runnable worker = new MyRunnable("Thread " + i);
            executor.execute(worker);
        }

        executor.shutdown();

        // Blocks until all tasks have completed execution after a shutdown request
        executor.awaitTermination(Long.MAX_VALUE, TimeUnit.DAYS);
        
        System.out.println("\nFinished all threads");
    }

    public static class MyRunnable implements Runnable {
    	
    	private final String threadName;
    	
    	MyRunnable(String threadName) {
    		this.threadName = threadName;
    	}

        @Override
        public void run() {
        	System.out.println("This is " + threadName);
        }
    }
}

~~~~