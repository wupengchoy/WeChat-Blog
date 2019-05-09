[TOC]

## Semaphore

### API

​	Semaphore—信号量，控制某个时间内线程执行的个数，构造方法中传入一个int类型的参数，控制线程执行的个数。

```java
Semaphore semaphore = new Semaphore(10);
```

​	Semaphore中有两个方法—acquire()/release();代码执行到此处时会判断信号量是否使用完，如果使用完，则当前线程需要等待前面的线程release后释放信号量。在acquire()的重载方法中可以传入一个int类型参数，表示每次线程消耗的信号量。例如使用上面创建出来的Semaphore，此时有10个信号量，如果设置参数semaphore.acquire(2);那么当前Semaphore最对只能支持5个线程并发执行,同时，在返还信号量的时候，也必须需要使用release()的带参重载方法release(int permits)返回之前使用的信号量数量，否则不带参的release将会默认返回1个信号量的值，导致Semaphore中的信号量数量减少，后面的线程可能永远获取不到而产生死锁。

​	举例说明：

```java
 //permits，统一时间内允许同事执行acquire和release之间代码的线程个数-受消耗信号量限制
Semaphore semaphore = new Semaphore(2);
for (int i = 0; i < 5; i++) {
	new Thread(() -> {
		try {
			semaphore.acquire();//可以传入int参数表示每次消耗多少信号量，如果设置信号量10，此处传入2，则每次只能支持5个线程
			System.out.println(Thread.currentThread().getName() + " start time：" + System.currentTimeMillis());
			Thread.sleep(1000);
			System.out.println(Thread.currentThread().getName() + " end time：" + System.currentTimeMillis());
			semaphore.release();
		} catch (Exception e) {
			System.out.println("error");
		}
	}).start();
}
```

​	执行结果如下：

```java
Thread-0 start time：1555667822270
Thread-1 start time：1555667822271
Thread-0 end time：1555667823271
Thread-2 start time：1555667823271
Thread-1 end time：1555667823276
Thread-3 start time：1555667823276
Thread-2 end time：1555667824271
Thread-4 start time：1555667824272
Thread-3 end time：1555667824281
Thread-4 end time：1555667825276
```

​	可以看到每次只允许两个线程在执行，如果将Semaphore的参数修改为10，设置semaphore.acquire(2),并创建10个线程执行，则得到的结果如下：

```java
Thread-0 start time：1555667906143
Thread-1 start time：1555667906144
Thread-2 start time：1555667906144
Thread-3 start time：1555667906145
Thread-4 start time：1555667906146
Thread-0 end time：1555667907145
Thread-1 end time：1555667907145
Thread-2 end time：1555667907145
Thread-5 start time：1555667907146
Thread-3 end time：1555667907145
Thread-6 start time：1555667907146
Thread-4 end time：1555667907146
Thread-6 end time：1555667908146
Thread-5 end time：1555667908146
Thread-7 start time：1555667908147
Thread-7 end time：1555667909147
Thread-8 start time：1555667909147
Thread-8 end time：1555667910151
```

​	同一时间每次最多只允许5个线程开始执行，只有在有线程执行release()并归还信号量的时候，下一个线程才允许加入。

​	正常情况下，如果使用acquire()方法，在release()之间，如果线程被中断，则会报出异常java.lang.InterruptedException，如下代码：

```java
Thread t1 = new Thread(()->{
	try {
		//semaphore.acquireUninterruptibly();
		semaphore.acquire();
		System.out.println(Thread.currentThread().getName() + " start time：" + LocalDateTime.now());
		//Thread.sleep(5000);
		for (int i = 0; i < Integer.MAX_VALUE / 50; i++) {
			String str = new String();Math.random();
		}
		System.out.println(Thread.currentThread().getName() + " end time：" + LocalDateTime.now());
		semaphore.release();
	} catch (Exception e) {
		System.out.println(e.toString());
	}
});
t1.start();
try {
	t1.interrupt();
	System.out.println(t1.getName() + " interrupt");
} catch (Exception e) {
	System.out.println("error2");
}
```

​	输出如下：

```java
Thread-0 interrupt
java.lang.InterruptedException
```

​	将semaphore.acquire()替换为semaphore.acquireUninterrupt()结果如下：

```java
Thread-0 interrupt
Thread-0 start time：2019-04-19T20:02:03.099
Thread-0 end time：2019-04-19T20:02:04.579
```

> ​	为什么不能使用Thread.sleep()方法延迟？
>
> ​	在使用sleep()方法时，如果线程被中断，sleep方法中还是会抛出Interrupt异常，在sleep的catch中，如果只包含sleep方法，那么这个sleep方法将因为异常失效，如果这个catch范围包含了线程的业务代码，那么包含的业务代码将会被跳过。

​	PS. semaphore.acquireUninterrupt()也存在一个包含int参数的重载方法，作用也是定义每个线程消耗的信号量，与acquire()方法类似。

> 其他方法：
>
> availablePermits():返回当前可用信号量。
>
> drainPermits():返回当前可用信号量个数并清零。
>
> getQueueLength():返回当前等待信号量的线程个数。
>
> hasQueuedThreads():返回拥有当前信号量的线程个数。
>
> tryAcquire(int permits):尝试获取，如果成功返回true,如果失败则返回false，此时不会对线程进行阻塞，通常与if联用。
>
> tryAcquire(long timeOut,TimeUnit unit):在指定的事件内尝试获取信号量，获取不到返回false。
>
> tryAcquire(int permits,long timeOut,TimeUnit unit):在指定事件内尝试获取一定数量的信号量，获取不到返回false。

### 公平与非公平信号量

​	Semaphore有一个重载构造方法Semaphore(int permits,boolean isFair);如果isFair是true，保证了线程获取锁的顺序与线程启动的顺序一致，但是不能保证线程一定会取得信号量，但是会按照启动顺序调用acquire()方法。即线程启动的顺序与调用acquire()方法的顺序一致，如果当前信号量消耗完，剩余的线程将会依照启动顺序排队，知道获取到指定数量的信号量。

### 应用

​	可以使用Semaphore实现一个自己的线程池。线程池中可用线程的数量就是信号量，在acquire()和release()方法之间获取可用线程。在生产者和消费者模式中也可以使用，消费者仓库的容量时一定的，在容量满了的时候，生产者阻塞不进行生产。

## Exchanger

​	Exchanger可以用来在两个不同的线程之间进行通信，主要方法是exchange(T data),用来在两个线程之间交换数据，当执行到exchange(T data)方法的时候，线程会进行阻塞，直到另外一个线程执行相同的方法并塞入另一个data数据，并且两个线程之间返回另一个线程传入的数据。如下线程测试：

```java
Exchanger<String> exchanger = new Exchanger<>();
for (int i = 0; i < 4; i++) {
    new Thread(() -> {
        try {
            String threadName = Thread.currentThread().getName();
            System.out.println(threadName + "中获取线程exchanger交换的数据:" + exchanger.exchange(threadName));
            System.out.println("end " + threadName);
        } catch (Exception e) {
            System.out.println(e.toString());
        }
    }).start();
}
```

​	当for循环设置为1时，只启动了一个线程，此时没有第二个线程来与当前线程交换数据，导致程序阻塞，控制台没有输出。

​	当for循环设置为2时，创建了两个线程交换数据，此时输出如下：

```tex
Thread-1中获取线程exchanger交换的数据:Thread-0
Thread-0中获取线程exchanger交换的数据:Thread-1
end Thread-0
end Thread-1
```

​	当for循环设置为3时，前两个线程交换数据，控制台输出与之前设置为2时一致，当执行到第三个线程的时候，没有数据与之交换，被阻塞，控制台没有继续输出。

​	当for循环设置为4时，创建的线程中的数据会两两交换，并且不会被阻塞，控制台输出如下：

```teX
Thread-0中获取线程exchanger交换的数据:Thread-1
end Thread-0
Thread-1中获取线程exchanger交换的数据:Thread-0
end Thread-1
Thread-3中获取线程exchanger交换的数据:Thread-2
end Thread-3
Thread-2中获取线程exchanger交换的数据:Thread-3
end Thread-2
```

​	Exchanger中还有一个重载方法exchange(T data,long timeOut,TimeUnit unit)表示在规定时间内有没有其他线程交换数据，如果没有，则抛出超时异常java.util.concurrent.TimeOutException。