---
title: Java多线程初始
date: 2021-07-08 21:43:25
tags: 学习
---

#### 1. 什么是进程?什么是线程?多线程有哪些应用场景?

​	进程是一个应用程序.线程是一个进程中的执行场景或者说是执行单元.一个进行可以启动多个线程

​	对于Java来说 启动Java程序,首先会激动JVM,JVM就是一个进程,JVM再启动一个主线程调用main方法,或者再启动一个垃圾回收线程进行负责回收垃圾.

​	两个进程之间资源不共享,两个线程之间堆内存和方法区内存共享,但是栈不共享.是属于一个线程一个栈

​	多线程的应用场景: 

​		1) 异步的实现发送短信,快速提高响应,用户的体验好

​		2) 异步的实现记录日志

​		3) 支付异步回调

​		4) 对我们后端接口中比较耗时间的代码都可以采用异步实现.

​		目的: 就是为了提高对HTTP协议的响应

#### 2. 并发和并行的区别

- 并发是一个处理器同时处理多个任务
- 并行是多个处理器或者是多核的处理器同时处理多个不同的任务

就好比是,并发是一个人同时吃三碗饭,并行就三个人同时吃三碗饭

#### 3. 对于单核的cpu来说,可以做到真正的多线程并发吗?什么是真正的多线程并发?多线程是开的越多越好吗?

​	A线程执行A,B线程执行B.A不会影响B,B也不会影响A,这就叫做真正的多线程并发.

​	单核的cpu不能狗做到真正的多线程并发,但是可以做到一种多线程并发的感觉.其实对于单核的cpu来说,在某一个时间点上实际只能处理一件事情.

​	如果在服务器上频繁的开启线程,会影响到服务器的性能,如果是项目比较小的话,可以采用多线程实现异步,如果是项目比较大或者高并发的情况下建议使用MQ实现异步.

#### 4. Java创建线程的方式(简单的)

- 编写一个类,继承Tread方法即可.
- 编写一个类,实现Runable接口,然后new Tread(编写的类)即可.
- 再调用start方法即可.
- 如果直接调用线程类的run方法,就不会启动方法,就等于是调用一个普通的方法

多线程创建小例子:

```java
public class MyThread {
    public static void main(String[] args) {
        System.out.println("开始线程!");
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                System.out.println("自定义线程1 -> " + i );
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i <10 ; i++) {
                    System.out.println("自定义线程2 -> "+i);
                }
            }
        }).start();
        for (int i = 0; i <10 ; i++) {
            System.out.println("main线程-> " +i);
        }
        System.out.println("主线程结束!");
    }
}
```

#### 5. Java创建线程的方式(补充)

- Callable接口,可以带返回结果的线程 底层原理就是wait和notify包装而成的 
- 线程池 四种实现方式,但是alibaba官方文档不推荐使用
- 利用框架如spring提供异步注解`@Asyn`

```java
public class CallableThread implements Callable<String> {
    @Override
    public String call() throws Exception {
        Thread.sleep(3000);
        return "Callable实现多线程的创建,并且有返回值!";
    }
    //callable接口是有返回值的
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> futureTask = new FutureTask<>(new CallableThread());
        new Thread(futureTask).start();
        //必须要等待线程结束完成,才能获取到返回值;
        //相当于如果需要获取返回值,那么就是一个类似单线程,如果不要返回值,才会是类似多线程的样子
        //并且如果有返回值,将会堵塞接下来的代码,使接下来的代码一起等待线程结束,再从上往下执行
        System.out.println(futureTask.get());
        System.out.println("主线程完毕!");
    }
}
```

```java
//线程池的使用
public class ThreadPool {
    public static void main(String[] args) {
        //线程池的核心就是帮助我们做复用机制.
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(()->{
            System.out.println(Thread.currentThread().getName()+"我是线程池!!");
        });
    }
}
```

使用`@Async`实现异步

controller:

```java
@RestController
public class SpringAsync {
    @Resource
    private AsyncManage asyncManage;
    @GetMapping("/springAsync")
    public String springAsync() throws InterruptedException {
        asyncManage.print();
        //做查询操作
        Thread.sleep(1000);
        return "登录成功!";
    }
}
```

`@Async`无返回值实现:

```java
@Component
public class AsyncManage{
    @Async
    public void print() throws InterruptedException {
        print1();
        print2();
        print3();
    }

    public void print1() throws InterruptedException {
        System.out.println("打印日志!");
        Thread.sleep(2000);
    }

    public void print2() throws InterruptedException {
        System.out.println("发送邮件!");
        Thread.sleep(2000);
    }

    public void print3() throws InterruptedException {
        System.out.println("发送短信!");
        Thread.sleep(2000);
    }
}
```

`@Async`有返回值实现(利用Callable):

```java
@Async
    public Future<String> printString() throws InterruptedException {
        print1();
        print2();
        print3();
        return  new AsyncResult<>("有返回值的异步处理已完成!");
    }
```

​	在不访问其返回结果的时候就是异步处理,访问其结果的就是单线程处理的感觉

有几个限制：

- 两个方法都在同一个类里面，只是一个方法调用另一个异步方法，不生效。

  解决方法：拆分两个方法，将异步方法单独放在一个类里面，然后再去调用就解决了。

- 有接口方法的实现类里的注解不生效.

  解决方法：在实现类中调用一个没有接口的类才可以。

- `@SpringBootApplication`启动类当中需要添加`@EnableAsync`注解。

  异步方法使用注解`@Async`的返回值只能为void或者Future。

#### 6. 用户线程和守护线程的区别

区别:

- 用户线程,当我们主线程停止之后,用户线程不会随着主线程停止.
-  守护线程,当我们主线程停止之后,守护线程会随着主线程停止.

守护线程的使用场景

- GC线程,垃圾回收

Java默认情况创建的线程都是用户线程,设置守护线程的方法:

```java
Thread.setDaemon(true);
```

#### 7. 停止线程

 不建议使用stop方法停止线程,因为它底层使用强制停止线程,如果代码没有执行完毕的情况下也是强行停止.可以使用编码知识,例如判断条件为false或者true来进行判断执行,停止线程只用改成false即可.

#### 8. 线程的生命周期

- 新建状态 ----- 调用start方法
- 就绪状态 ----- 又叫可运行状态,具有抢夺CPU时间片的权利(执行权)
- 运行状态 ----- run方法的执行标志着线程开始进入运行状态
- 阻塞状态 ----- 当遇到阻塞时间(sleep方法等),此时线程会进入阻塞状态,处于阻塞状态会放弃当前占有的CPU时间片,结束后回到就绪状态
- 死亡状态

​	



