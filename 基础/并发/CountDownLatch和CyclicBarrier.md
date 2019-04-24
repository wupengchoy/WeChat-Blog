[TOC]

## CountDownLatch

​	CounDownLatch是同步功能的一个辅助类，用来阻塞线程，知道符合一定的条件的时候才开始执行。在构造方法中传入一个int类型的参数，用来表示当前有多少个线程到达。

​	CountDownLatch在使用时通常需要将await()方法和countDown()方法联合使用。在当前线程执行到await方法的时候，会判断当前CountDownLatch在创建时传入的int参数是否是0，如果不是则阻塞当前线程，知道int属性变为0。countDown()方法会将这个int参数减1。

​	CountDownLatch使用场景通常是需要某些线程到达一定的条件后一起执行。相当于在代码某处使用栏杆将这些线程拦住，当到达条件后将栏杆取回，这些线程开始抢占时间片段执行。

​	需要注意的是CountDownLatch在初始化之后只能使用1次，下次使用需要重新初始化，并且在await方法在不阻塞之后，后面的线程再次调用也不会进行阻塞，如下测试代码：

```java
CountDownLatch down = new CountDownLatch(2);
//只能使用一次，当全部到达之后，后面再来的线程不会阻塞
for (int i = 0; i <6 ; i++) {
    new Thread(()->{
        try{
            System.out.println(Thread.currentThread().getName()+"in...");
            down.await();
            System.out.println(Thread.currentThread().getName()+"out...");
        }catch (Exception e){}
    }).start();
    try {
        Thread.sleep(1000);
    }catch (Exception e){}
    down.countDown();
}
```

​	执行结果如下，可以看到循环阻塞两个线程之后，后面的6个线程不再阻塞。

```tex
Thread-0in...
Thread-1in...
Thread-0out...
Thread-1out...
Thread-2in...
Thread-2out...
Thread-3in...
Thread-3out...
Thread-4in...
Thread-4out...
Thread-5in...
Thread-5out...
```

> **API**
>
> await(long timeOut,TimeUnit unit),阻塞特定时长，如果超过这个时长则自动唤醒继续往下执行。
>
> getCount(),获取当前CountDownLatch剩余的计数值。

## CyclicBarrier

​	与CountDownLatch类似，也是线程同步的辅助类，同样是将线程阻塞直到某个条件达成的时候一起唤醒。CountDownLatch的计数不能重置，是一次性的，并且在计数的时候做减法。CyclicBarrier在创建的时候也会传入一个int类型的参数用来计数，但是CyclicBarrier在调用await的时候会做加法操作，并判断计数是否达到创建时传入的参数，如果没有达到，阻塞当前线程，达到了，唤醒当前所有阻塞的线程，并将计数重置。如果需要重新计数使用，可以调用reset将计数重置。

​	如下例子可以很好的说明：

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(3);
for (int i = 0; i <7 ; i++) {
    new Thread(()->{
        try{
            cyclicBarrier.await();
        }catch (Exception e){}
        System.out.println(Thread.currentThread().getName()+" run...");
    }).start();
    System.out.println("thread started...");
    try {
        Thread.sleep(1000);
    }catch (Exception e){}
}
```

​	输出结果如下,限制条件为3，只有当前线程阻塞条件达到3，即await方法被调用3次，线程才会被唤醒，否则将会一直阻塞下去。

```tex
thread started...
thread started...
thread started...
Thread-1 run...
Thread-2 run...
Thread-0 run...
thread started...
thread started...
thread started...
Thread-5 run...
Thread-4 run...
Thread-3 run...
thread started...
```

> **API**
>
> getNumberWaiting(),获取当前被阻塞的线程数量。
>
> getParties(),获取当前计数总量，即创建时传入的int参数。
>
> isBroken(),判断当前CyclicBarrier是否处于损坏状态。当阻塞的一个线程出现异常的时候，剩余的线程在执行的时候会抛出BrokenBarrierException，此时isBroken返回true，执行之前调用此判断可以判断当前线程是否能继续执行，避免抛出异常。
>
> await(long timeOut,TimeUnit unit),等待指定时长的阻塞时间，如果超过时长，则抛出TimeOutException异常。
>
> reset()，重置CyclicBarrier。