---
type: blog
title: Java 多线程
date: 2021-07-05
categories: 技术
tags: 模板
cover: false
meta:
  date: false
---



<!-- more -->

## 一、基础概念

程序：

进程：

线程：



## 二、线程创建

**继承 Thread 类、实现 Runnable 接口、实现 Callable 接口**

### 2.1 继承 Thread 类

```java
/*
继承Thread类
1）继承Thread类
2）重写 run() 方法
3）调用 start() 方法
 */
public class TestThread1 extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("线程内 ==== " + i);
            try {
                sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        TestThread1 testThread1 = new TestThread1();
        testThread1.start();
        // testThread1.run();

        for (int i = 0; i < 100; i++) {
            System.out.println("主线程 main ==== " + i);
            sleep(100);
        }
    }
}
```

### 2.2 实现 Runnable 接口

推荐使用这种方法，因为 Java 是单继承模式，使用接口更加灵活

```java
/*
实现 Runnable 接口

1）实现 Runnable 接口
2）实现 run() 方法
3）创建线程对象，调用 start() 方法
 */
public class TestThread2 implements Runnable {
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("线程内 ==== " + i);
            try {
                sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new TestThread2()).start();

        for (int i = 0; i < 100; i++) {
            System.out.println("主线程 main ==== " + i);
            sleep(10);
        }
    }
}
```

### 2.3 实现 Callable 接口

好处是：

1）可以定义返回值

2）可以抛出异常

```java
/*
实现 Callable 接口

1）实现 Callable 接口
2）实现 call() 方法
3）创建执行服务 ExecutorService
4）提交执行 submit()
5）获取结果 get()
6）关闭服务 shutdown()
 */
public class TestThread3 implements Callable<Boolean> {

    public Boolean call() throws Exception {
        for (int i = 0; i < 100; i++) {
            String tname = Thread.currentThread().getName();
            System.out.println(tname + " 线程内 ==== " + i);
            try {
                sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        return true;
    }

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        // 创建执行服务
        ExecutorService executorService = Executors.newFixedThreadPool(3);

        // 提交执行
        Future<Boolean> f1 = executorService.submit(new TestThread3());
        Future<Boolean> f2 = executorService.submit(new TestThread3());
        Future<Boolean> f3 = executorService.submit(new TestThread3());

        // 获取结果
        Boolean rf1 = f1.get();
        Boolean rf2 = f2.get();
        Boolean rf3 = f3.get();

        System.out.println(rf1);
        System.out.println(rf2);
        System.out.println(rf3);

        // 关闭服务
        executorService.shutdown();
    }
}
```



## 三、线程状态 & 转换

### 线程状态

![image-20210713110235210](https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20210713110235210.png)

### 线程方法

| 方法          | 说明                               |
| ------------- | ---------------------------------- |
| `setPriority` | 更改线程优先级                     |
| `sleep`       | 线程休眠                           |
| `join`        | 等待该线程终止？                   |
| `yield`       | 暂停当前执行的线程，并执行其他线程 |
| `interrupt`   | 中断线程（别用！）                 |
| `isAlive`     | 检测线程是否存活                   |

### 停止线程

1. JDK 提供的`stop()`和`destroy()`方法（已废弃，不要用）
2. 设置一个标志位终止线程（但并不会直接停下来，flag置为false之后，子线程可能还会把之前进行到一半的继续执行完）

```java
public class TestStopThread1 implements Runnable {
    private boolean flag = true;

    public void run() {
        int i = 0;
        while (flag) {
            System.out.println("run... Thread" + i++);
        }
    }

    public void stop() {
        flag = false;
        System.out.println("Thread stop!!!");
    }

    public static void main(String[] args) {
        TestStopThread1 testStopThread1 = new TestStopThread1();
        new Thread(testStopThread1).start();
        for (int i = 0; i < 1000; i++) {
            System.out.println("main--- " + i);
            if (i == 900) {
                testStopThread1.stop();
            }
        }
    }
}
```

3. 所以怎么做？

### 线程休眠

- `sleep` 存在异常 `InterruptedException`
- `sleep` 时间到达后，线程进入就绪状态
- 每一个对象都有一个锁，`sleep`不会释放锁？

### 线程礼让

- `yield` 让当前正在执行的线程暂停，但不阻塞
- 将线程从运行状态转为就绪状态
- 让CPU重新调度

```java
public class TestYield1 implements Runnable {

    public void run() {
        System.out.println(Thread.currentThread().getName() + " 线程开始执行");
        Thread.yield();
        System.out.println(Thread.currentThread().getName() + " 线程结束执行");
    }

    public static void main(String[] args) {
        TestYield1 testYield = new TestYield1();

        new Thread(testYield, "Thread_A").start();
        new Thread(testYield, "Thread_B").start();
        new Thread(testYield, "Thread_C").start();
        new Thread(testYield, "Thread_D").start();
        new Thread(testYield, "Thread_E").start();
    }
}
```

### 线程插队

- `join` 合并线程，待此线程执行完成后，再执行其他线程，其他线程阻塞

```java
public class TestJoin1 implements Runnable {
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("join..." + i);
            try {
                sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        TestJoin1 testJoin1 = new TestJoin1();
        Thread thread = new Thread(testJoin1);
        thread.start();

        for (int i = 0; i < 200; i++) {
            if (i == 50) {
                thread.join();
            }
            System.out.println("main..." + i);
        }
    }
}
```



## 四、线程同步

### 线程同步机制

**要解决的问题：**

同一个资源多个人（线程）都想使用

**解决方案：**

队列 & 锁

**线程同步带来的问题：**

- 造成性能下降
- 优先级倒置：可能有高优先级的线程等待低优先级线程的情况



线程不安全例子：

```java
public class UnsafeBuyTicket {
    public static void main(String[] args) {
        BuyTicket buyTicket = new BuyTicket();

        new Thread(buyTicket, "t1").start();
        new Thread(buyTicket, "t2").start();
        new Thread(buyTicket, "t3").start();
        new Thread(buyTicket, "t4").start();
    }
}

class BuyTicket implements Runnable {
    private int ticketNum = 10;
    boolean flag = true;

    @Override
    public void run() {
        while (flag) {
            try {
                buy();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void buy() throws InterruptedException {
        if (ticketNum <= 0) {
            flag = false;
            return;
        }

        Thread.sleep(100);
        System.out.println(Thread.currentThread().getName() + "拿到 " + ticketNum--);
    }
}
```



**线程同步 synchronized** 

`synchronized`

1. 锁方法

锁的是 `this`

```java
private synchronized void buy() {
  ...
}
```

2. 锁代码块

```java
synchronized (对象名) {
  ...
}
```



**死锁**

发生原因：某一个同步块同时拥有“两个以上对象的锁”



**Lock（锁）**



## 五、线程协作

**生产者消费者模式**

问题：两个线程之间的通信问题

`wait()`

`notify()`

解决方案：

1. 管程法

生产者将生产好的数据放入缓冲区，消费者从缓冲区拿数据

```java
public class TestMonitor {
    public static void main(String[] args) {
        Container container = new Container();

        new Producer(container).start();
        new Consumer(container).start();
    }
}

class Producer extends Thread {
    private Container container;

    public Producer(Container container) {
        this.container = container;
    }

    @Override
    public void run() {
        for (int i = 1; i < 100; i++) {
            Chicken chicken = new Chicken(i);
            container.push(chicken);
        }
    }
}

class Consumer extends Thread {
    private Container container;

    public Consumer(Container container) {
        this.container = container;
    }

    @Override
    public void run() {
        for (int i = 1; i < 100; i++) {
            container.pop();
        }
    }
}

class Chicken {
    int id;

    public Chicken(int id) {
        this.id = id;
    }
}

class Container {
    Chicken[] chickens = new Chicken[10];
    int count = 0;

    // 生产者放入产品
    public synchronized void push(Chicken chicken) {
        // 如果满了，等待消费者
        if (count == chickens.length) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        // 没满就放入
        chickens[count] = chicken;
        count++;
        System.out.println("生产了... " + chicken.id + " 目前有" + count);

        // 通知消费者
        this.notifyAll();
    }

    // 消费者消费产品
    public synchronized Chicken pop() {
        // 如果没有，就等待生产
        if (count == 0) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        // 如果有就弹出
        Chicken chicken = chickens[count - 1];
        count--;
        System.out.println("消费了... " + chicken.id + " 目前有" + count);

        this.notifyAll();

        return chicken;
    }
}
```



2. 信号灯法

设置标志位

类似于上例中的 count（只不过count只有两种状态）



## 六、线程池

问题：频繁创建、销毁线程对性能影响很大

思路：提前创建好线程，用的时候取，用完了还回来不销毁。





线程 并行？并发？
