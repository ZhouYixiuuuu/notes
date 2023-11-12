# Java多线程

## 进程和线程

一个进程里可以有多个线程

比如说一个视频中有声音、弹幕、图像

多个线程的运行由调度器安排调度

发生资源抢夺时，采用并发控制

## 三种实现多线程的方法  

### 继承Thread类

1. 自定义线程类继承`Thread`类
2. 重写`run()`方法，编写线程执行体
3. 创建线程对象，调用`start()`方法启动线程

```java
public class ThreadTest extends Thread{
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println("我在看代码");
        }
    }

    public static void main(String[] args) {
        ThreadTest t = new ThreadTest();
        t.start();

        for (int i = 0; i < 20; i++) {
            System.out.println("我在学习多线程");
        }
    }
}
```

你想给这个线程取名字

```java
public thread1(String name) {
            super(name);
        }

//或者
t.setname("")
```

实现一个构造函数

就可以实现取名字了`thread1 t1 = new thread1("继承自Thread类->");`

### 实现Runnable接口

1. 定义`MyRunnable`类实现`Runnable`接口
2. 实现`run()`方法，编写线程执行体
3. 创建线程对象，调用`start()`方法启动线程

```java
public class ThreadTest implements Runnable{
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println("我在看代码");
        }
    }

    public static void main(String[] args) {
        ThreadTest t = new ThreadTest();
        new Thread(t).start();

        for (int i = 0; i < 20; i++) {
            System.out.println("我在学习多线程");
        }
    }
}
```

**推荐使用，避免单继承局限性，灵活方便，方便同一个对象被多个线程使用**

龟兔赛跑复习一下前面所学

```java
public class race implements Runnable{
    private String Winner;
    @Override
    public void run() {
        for (int i = 1; i <= 100; i++) {
            if(Thread.currentThread().getName().equals("兔子") && i % 10 == 0)
            {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            boolean flag = gameOver(i);
            if (flag) break;
            System.out.println(Thread.currentThread().getName() + "-->跑了" + i + "步");
        }

    }

    public boolean gameOver(int x)
    {
        if (x == 100)
        {
            Winner = Thread.currentThread().getName();
            System.out.println("Winner is" + Winner);
            return true;
        }
        else if (Winner != null) return true;
        return false;
    }

    public static void main(String[] args) {
        race r = new race();

        new Thread(r, "兔子").start();
        new Thread(r, "乌龟").start();
    }
}
```

### 实现Callable接口

## 静态代理

1. 真实对象和代理对象都要实现同一个接口
2. 代理对象要代理真实对象

```java
public class StaticProxy {
    public static void main(String[] args) {
        //new Thread(t).start();
        //是不是非常地像，thread底层就是静态代理
        new WeddingCompany(new you()).Happymarry();
    }
}

interface Marry{
    void Happymarry();
}

class you implements Marry{
    public void Happymarry()
    {
        System.out.println("小红要结婚了，超开心");
    }
}

class WeddingCompany implements Marry {
    private Marry target;
    public WeddingCompany(Marry target) {
        this.target = target;
    }

    @Override
    public void Happymarry() {
        before();
        this.target.Happymarry();
        after();
    }

    private void after() {
        System.out.println("结婚之后");
    }

    private void before() {
        System.out.println("结婚之前");
    }
}

```

## Lamda表达式

> 函数式接口：一个接口包含唯一一个抽象方法

实例代码

```java
public class ThreadTest{
    public static void main(String[] args) {
        ILove l = (int a) -> {
            System.out.println("I Love You->" + a);
        };
        l.love(520);
    }
}

interface ILove{
    void love(int a);
}
```

简化之后

```java
ILove l = a -> System.out.println("I love you->" + a);
        l.love(521);
```

## 线程状态

![线程状态](C:\Users\z1382\AppData\Roaming\Typora\typora-user-images\image-20221112153635926.png)

### 线程停止



### 线程休眠

实现一个实时显示时间的钟表

```java
import java.text.SimpleDateFormat;
import java.util.Date;

public class ThreadSleep {
    public static void main(String[] args) throws InterruptedException {
        Date date = new Date(System.currentTimeMillis());
        while(true)
        {
            Thread.sleep(1000);
            System.out.println(new SimpleDateFormat("HH:mm:ss").format(date));
            date = new Date(System.currentTimeMillis());
        }
    }
}
```

实现倒计时

```java
public static void main(String[] args) throws InterruptedException {
        int count = 10;
        while(count > 0)
        {
            System.out.println(count -- );
            Thread.sleep(1000);
        }
    }
```



### 线程礼让

礼让不一定成功，看cpu心情

使用`Thread.yield()`

### 线程强制执行

`join()`

```java
public class ThreadJoin implements Runnable{

    @Override
    public void run() {
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        for (int i = 0; i < 1000; i ++ )
        {
            System.out.println("线程vip来咯");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ThreadJoin j = new ThreadJoin();
        Thread thread = new Thread(j);
        thread.start();

        for (int i = 0; i < 500; i++) {
            if (i == 200)
            {
                thread.join();
            }
            System.out.println("main->" + i);
        }
    }
}
```

### 观测线程状态

`Thread.State state = thread.getState()`

判断当线程未结束`while(state != Thread.State.TERMINATED)`

```java
public class testState {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(()->{
            for (int i = 0; i < 5; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.println("/////");
            }
        });

        Thread.State state = thread.getState();
        System.out.println(state);

        thread.start();
        state = thread.getState();
        System.out.println(state);

        while(state != Thread.State.TERMINATED)
        {
            Thread.sleep(100);
            state = thread.getState();
            System.out.println(state);
        }
    }
}
```

### 线程优先级

`Thread.currentThread().getPriority()`查看线程优先级

`t2.setPriority(1);`设置线程优先级

你把优先级设置到最高`t3.setPriority(Thread.MAX_PRIORITY);`也不一定让人家最先执行

先设置优先级，再启动！！！

### 守护线程

什么线程是守护线程啊？如，后台记录操作日志、监控内存、垃圾回收等待……

`thread.setDaemon(true)`默认false是用户线程，正常的线程都是用户线程

## 线程同步

> 并发：同一个对象被多个线程同时操作
>
> 线程同步就是一个等待机制，多个需要同时访问此对象的线程进入这个对象的等待池形成队列
>
> 锁机制(synchronized)

### 同步方法及同步块

`synchronized`直接加在方法前面（实际上就是作用在自己的类对象上！对于实现Runnable接口的多线程方式比较有用！）

`synchronized(Obj){}`Obj可以是任何对象，但是这个对象不能发生修改

在对象上，是Integer，不是int

### 死锁

> 死锁：多个线程互相抱着对方需要的资源，然后形成僵持

```java
package homework1111_1;

public class DeadLock {
    public static void main(String[] args) {
        Makeup g1 = new Makeup(1, "小红");
        Makeup g2 = new Makeup(0, "小丽");

        g1.start();
        g2.start();
    }
}

class Lipstick {

}

class Mirror {

}

class Makeup extends Thread {
    static Lipstick l = new Lipstick();
    static Mirror m = new Mirror();

    private int choice;
    private String name;

    public Makeup(int choice, String name)
    {
        this.choice = choice;
        this.name = name;
    }

    @Override
    public void run() {
        if (choice == 1) //当她先选择了口红
        {
            synchronized (l) {
                System.out.println(name + "正在使用口红");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }

                synchronized (m) {
                    System.out.println(name + "正在使用镜子");
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
            }
            
        }
        else {
            synchronized (m)
            {
                System.out.println(name + "正在使用镜子");
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }

                synchronized (l)
                {
                    System.out.println(name+"正在使用口红");
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
            }
            
        }
    }
}

```

当一个人锁定镜子的时候，还要锁定口红；

另一个锁定口红的时候，还要锁定镜子；形成僵持

```java
synchronized (l) {
                System.out.println(name + "正在使用口红");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
synchronized (m) {
                System.out.println(name + "正在使用镜子");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
```

这样子就可以解决死锁问题

### Lock锁

```java
class ticket implements Runnable {
    int ticketCount = 10;
    private final ReentrantLock lock = new ReentrantLock();
    @Override
    public void run() {
        while(true)
        {
            try {
                lock.lock();
                if (ticketCount > 0)
                {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                    System.out.println(ticketCount --);
                }
            }finally{
                lock.unlock();
            }
        }
    }
}
```

```
class A {
    private final ReentrantLock lock = new ReentrantLock();
    public void run() {
        try {
            lock.lock();
            //保证线程安全的代码
        }
        finally {
            lock.unlock();
        }
    }
}
```

## 线程协作

### 线程通信

* wait() : 表示线程一直等待，直到其他线程通知
* notifyAll()：唤醒同一个对象上所有调用wait()方法的线程

这两个方法只能用在`synchronized`里面

### 管程法

生产者将生产好的数据放入缓冲区，消费者从缓冲区拿出数据。

```java
package 多线程;

//生产者消费者问题，管程法
public class TestPC {
    public static void main(String[] args) {
        SynContainer container = new SynContainer();

        Producer producer = new Producer(container);
        Consumer consumer = new Consumer(container);

        new Thread(producer).start();
        new Thread(consumer).start();
    }
}

class Chicken {
    int id;

    public Chicken(int id)
    {
        this.id = id;
    }
}

class Producer implements Runnable {
    SynContainer container;

    public Producer(SynContainer container)
    {
        this.container = container;
    }
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            container.push(new Chicken(i));
            System.out.println("生产了" + i + "只鸡");
        }
    }
}

class Consumer implements Runnable {
    SynContainer container;

    public Consumer(SynContainer container)
    {
        this.container = container;
    }
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("消费了-->" + container.pop().id + "只鸡");
        }
    }
}

//缓冲区
class SynContainer {
    Chicken[] chickens = new Chicken[10];
    int count = 0;

    public synchronized void push(Chicken chicken){
        if (count == chickens.length)
        {
            //缓冲区已经满了，通知消费者消费，并且让生成者停止
            try {
                this.wait();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }

        chickens[count ++ ] = chicken;
        this.notifyAll();
    }

    public synchronized Chicken pop(){
        if (count == 0) //缓冲区是空的，通知生产者生产，并且让消费者停止消费
        {
            try {
                this.wait();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        count -- ;
        Chicken c = chickens[count];
        this.notifyAll();
        return c;
    }
}

```



### 信号灯法

设置一个标志位

```java
package 多线程;

//生产者消费者问题，管程法
public class TestPC {
    public static void main(String[] args) {
        TV tv = new TV();

        new Thread(new player(tv)).start();
        new Thread(new watcher(tv)).start();
    }
}

class player implements Runnable {
    TV tv;

    public player(TV tv)
    {
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 50; i ++ )
        {
            if (i % 10 == 0)
            {
                this.tv.play("抖音：记录美好生活");
            }
            else
            {
                this.tv.play("快乐大本营");
            }
        }
    }
}

class watcher implements Runnable{
    TV tv;

    public watcher(TV tv) {
        this.tv = tv;
    }
    @Override
    public void run() {
        for (int i = 0; i < 50; i++) {
            tv.watch();
        }
    }
}

class TV {
    String voice;
    boolean flag = true;

    public synchronized void play(String voice)
    {
        if (!flag)
        {
            try {
                this.wait();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }

        this.voice = voice;
        System.out.println("演员表演了" + voice);
        this.notifyAll();
        this.flag = !this.flag;
    }

    public synchronized void watch()
    {
        if (flag)
        {
            try {
                this.wait();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }

        System.out.println("观众观看了" + this.voice);
        this.notifyAll();
        this.flag = !this.flag;
    }
}


```

### 线程池

提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。可以避免频繁创建销毁，实现重复利用。

ExecutorService ：真正的线程池接口

Executors：用于创建并返回不同类型的线程池

```java
package 多线程;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class TestPool {
    public static void main(String[] args) {
        //1.创建服务，创建线程池
        //newFixedThreadPool 参数为线程池的大小
        ExecutorService service = Executors.newFixedThreadPool(10);

        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());

        //2.关闭链接
        service.shutdown();
    }
}

class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}

```





