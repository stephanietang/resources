## ExecutorService

public class ExecutorServiceExample {

    private static final int THREAD_NUM = 30;

    public static void main(String args[]) throws Exception {
	
		ExecutorService executor = Executors.newFixedThreadPool( THREAD_NUM );

		for(int i = 0; i < THREAD_NUM; i++) {
			Runnable worker = new MyRunnable();
			executor.execute(worker);
		}

		executor.shutdown();

		// Blocks until all tasks have completed execution after a shutdown request
		executor.awaitTermination(Long.MAX_VALUE, TimeUnit.DAYS);
	}

	public static class MyRunnable implements Runnable {

		@Override
		void run() {
			 // code
		}
	}
}