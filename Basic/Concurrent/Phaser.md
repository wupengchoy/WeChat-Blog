[TOC]

# Phaser

​	前面CountDownLatch和CyclicBarrier都有各自的优缺点，但是他们在创建之后，parties阻塞条件是确定的不能修改。所以在使用CountDownLatch和CyclicBarrier之前就应该确定线程个数，并且不能中途退出或者加入。

​	Phaser功能与CountDownLatch和CyclicBarrier类似，但是可以动态的在使用过程中进行注册或者退出。Phaser在构造方法中也需要传递一个int型参数，设置屏障个数。

## arriveAndAwaitAdvance()

​	与之前await()方法类似，Phaser阻塞的方法叫arriveAndAwaitAdvance()，方法的作用是阻塞到达此处的线程。没执行一次arriveAndAwaitAdvance方法，Phaser之前设置的屏障个数减一，知道达到0为止，会唤醒所有被阻塞的线程。Phaser可以设置多个屏障，在下次继续使用arriveAndAwaitAdvance的时候会重新开始计数。

```java
Phaser phaser = new Phaser(3);
for (int i = 0; i < 3; i++) {
    new Thread(()-> {
        System.out.println(Thread.currentThread().getName()+" started...");
        //多段使用-与CountDownLatch的await方法类似，但是await()方法不能多段使用
        phaser.arriveAndAwaitAdvance();
        System.out.println(Thread.currentThread().getName()+" runnung...");
        phaser.arriveAndAwaitAdvance();
        System.out.println(Thread.currentThread().getName()+" ended...");
    }).start();
}
```

​	如上代码执行之后结果如下：

```tex
Thread-0 started...
Thread-2 started...
Thread-1 started...
Thread-1 runnung...
Thread-0 runnung...
Thread-2 runnung...
Thread-2 ended...
Thread-0 ended...
Thread-1 ended...
```

​	从结果可以看出每个阶段屏障之前都会等待每个线程执行到此处位置，随后一起唤醒，直到下一个屏障，并且每次使用的都是同一个Phaser对象，说明Phaser可以复用。

## arrive()

​	与arriveAndAwaitAdvance功能类似，表示到达屏障处，使得parties加1，但是线程不会阻塞，而是继续向下执行。

## arriveAndDeregister()

​	修改如上代码，是线程thread-2在执行过程中退出，看看会出现什么情况？

```java
Phaser phaser = new Phaser(3);
for (int i = 0; i < 3; i++) {
    new Thread(()-> {
        System.out.println(Thread.currentThread().getName()+" started...");
        //多段使用-与CountDownLatch的await方法类似，但是await()方法不能多段使用
        phaser.arriveAndAwaitAdvance();
        System.out.println(Thread.currentThread().getName()+" runnung...");
        if("Thread-2".equals(Thread.currentThread().getName()))
        {
            return;
        }
        phaser.arriveAndAwaitAdvance();
        System.out.println(Thread.currentThread().getName()+" ended...");
    }).start();
}
```

​	执行结果如下：

```tex
Thread-0 started...
Thread-1 started...
Thread-2 started...
Thread-2 runnung...
Thread-1 runnung...
Thread-0 runnung...
```

​	线程最后会阻塞在end之前，因为thread-2中途退出后不再调用arriveAndAwaitAdvance方法，导致剩余的两个线程阻塞并且不会解除。可以在退出时调用arriveAndDeregister方法，表示当前线程注销，此时原来传入的屏障数量会减一，后面的arriveAndAwaitAdvance在执行的时候判断的屏障数量会减少一个，从而不会阻塞剩余线程。所以在return之前加上**phaser.arriveAndDeregister();**后执行结果如下，可以看到剩余线程都能够执行完成。

```tex
Thread-0 started...
Thread-1 started...
Thread-2 started...
Thread-2 runnung...
Thread-0 runnung...
Thread-1 runnung...
Thread-1 ended...
Thread-0 ended...
```

> getRegisteredParties():动态获取当前注册数量。
>
> register():与arriveAndDeregister()相反，动态的注册线程阻塞数量。
>
> bulkRegister(int parties):批量增加parties数量的注册数。

## getPhase()

​	getPhase()方法可以获取当前到达第一个屏障，可以简单的理解为调用多少次arrveAndAwaitAdvance()方法。

```java
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        phaser.arriveAndAwaitAdvance();
        System.out.println("phase count1:" + phaser.getPhase());
        phaser.arriveAndAwaitAdvance();
        System.out.println("phase count2:" + phaser.getPhase());
        phaser.arriveAndAwaitAdvance();
        System.out.println("phase count3:" + phaser.getPhase());
    }).start();
}
```

​	如上测试用例输出：

```tex
phase count1:1
phase count1:1
phase count1:1
phase count2:2
phase count2:2
phase count2:2
phase count3:3
phase count3:3
phase count3:3
```

## onAdvance()

​	onAdvance方法在每个屏障第一次被调用的时候运行，需要将Phaser初始化改为如下：

```java
Phaser phaser = new Phaser(3){
    @Override
    protected boolean onAdvance(int phase, int registeredParties) {
        System.out.println("onAdvance()");
        return super.onAdvance(phase, registeredParties);
    }
};
```

​	onAdvance方法在返回true的时候表示当前屏障失效，不阻塞接下来的线程。返回false则当前Phaser对象继续生效。

>getArrivedParties():获取已被使用的parties数量。
>
>getUnarrivedParties():获取未被使用的parties熟练。
>
>awaitAdvance(int phase):如果传入的phase参数与调用getPhase()返回值相同